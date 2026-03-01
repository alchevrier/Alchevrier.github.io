---
layout: post
title:  "Building Mini-Kafka Phase 3: Partitioning"
date:   2026-02-28 14:00:00 +0800
categories: projects distributed-systems kafka storage
---

## Introduction

On the 22nd of February, we shipped our own binary protocol written from scratch. This gives us two pillar of Kafka implemented from scratch: log storage engine and binary protocol. Phase 3 is about leveraging this code and add a new dimension which later on will enable fault-tolerance and replication: partitioning.

Partitioning is about splitting a topic within a broker into multiple pieces which can be accessed independently. This enables:
- Grouping of messages of the same class within the same group
- Consumer that cares about a certain class of messages not having to go through the whole log
- Producers may care about producing to a certain partition or let the broker decide for them

This is the building block for replication as with replication we can leverage partitioning to elect leaders within multiple brokers for each partition which achieves fault-tolerance. Replication is Phase 4 which will leverage the work we have done in Phase 3 here. We could have started with replication directly — most production environments use a single partition — but since this project is called distributed-messaging, we wanted to support multiple partitions first.

Like always we have worked by designing first the ADR (Architecture Decision Record) which is available here: https://github.com/alchevrier/distributed-messaging/blob/main/docs/adr/0011-phase-3-project-structure.md 

For us designing-first is a superpower:
- Most choices are made upfront
- This speeds up massively development
- Very few key development decision made at development time just implementation concerns

The key idea for partitioning is:
- When producing, we receive a key or not
- Receiving a key means we need to find its assigned partition by using the following formula: hash(key) % noOfPartition
- Not receiving a key means we will assign the message to the least-used partition
- LogManager instead of keeping a record of just the topic assign to a log, it will now track topic-partitionNumber as well as a PartitionManager
- The PartitionManager keeps track of the current number of messages per partition for a given topic as well as resolving the partition number given a key. 

## Implementing MurmurHash3 by Hand

A key element of partitioning is to have the following properties when resolving a key to a partitionNumber:
- Given the same key it should give the same partitionNumber
- There should be a good distribution of keys
- Speed of execution

To achieve this Kafka has chosen to use MurmurHash2 for their DefaultPartitioner. We have chosen the most up-to-date hash which is MurmurHash3 - also used widely such as Cassandra. 

MurmurHash3 is a non-cryptographic hash function designed for speed and good distribution. The name comes from the two core operations: Multiply and Rotate. It consists in 3 phases:
- Phase 1: Body — Process 4 bytes at a time
- Phase 2: Tail — Handle remaining bytes
- Phase 3: Finalization (fmix)

```java
public class MurmurHash3HashProvider implements HashProvider {

    private static final int C1 = 0xcc9e2d51;
    private static final int C2 = 0x1b873593;
    private static final int C3 = 0xe6546b64;
    private static final int C4 = 0x85ebca6b;
    private static final int C5 = 0xc2b2ae35;

    @Override
    public int hash(String key) {
        var toBeProcessed = key.getBytes();

        var bodyResult = 0;
        for (int i = 0; i < (toBeProcessed.length / 4) * 4; i = i + 4) {
            var chunk = toInt(Arrays.copyOfRange(toBeProcessed, i, i + 4));
            chunk = Integer.rotateLeft(chunk * C1, 15) * C2;
            bodyResult ^= chunk;
            bodyResult = Integer.rotateLeft(bodyResult, 13);
            bodyResult *= 5;
            bodyResult += C3;
        }

        var missingLastByte = toBeProcessed.length % 4;
        var tailResult = 0;
        switch (missingLastByte) {
            case 3: tailResult ^= (toBeProcessed[toBeProcessed.length - 3] & 0xff) << 16;
            case 2: tailResult ^= (toBeProcessed[toBeProcessed.length - 2] & 0xff) << 8;
            case 1: tailResult ^= toBeProcessed[toBeProcessed.length - 1] & 0xff ;
        }
        bodyResult ^= Integer.rotateLeft((tailResult * C1), 15) * C2;

        bodyResult ^= toBeProcessed.length;
        bodyResult ^= bodyResult >>> 16;
        bodyResult *= C4;
        bodyResult ^= bodyResult >>> 13;
        bodyResult *= C5;
        bodyResult ^= bodyResult >>> 16;

        return bodyResult;
    }

    private int toInt(byte[] chunk) {
        return ByteBuffer.wrap(chunk).order(ByteOrder.LITTLE_ENDIAN).getInt();
    }
}
```

As you can see in the above a few gotcha came up while implementing the hash:
- When using ByteBuffer java uses by default ByteOrder.BIG_ENDIAN this is to align with the Network side of Java NIO that needs to interact with Internet which historically has its Integer implemented in BIG_ENDIAN. The algorithm expects little-endian byte order because MurmurHash3 was designed for in-memory hashing on x86 CPUs, which are little-endian.
- & 0xff -> this is because in Java bytes are signed by design, using this AND with constant allows to unsign it
- use of fallthrough syntax switch: allow us to cater for handling the last 3/2/1 bytes remaining. The most commonly used switch syntax -> does not fall through which would have required to use a nested ifs or duplicated code. 
- we have used >>> instead of >> as the algorithm requires to manipulate unsigned integers. >>> fills the top bits with zeros instead of sign-extending, with >> a negative hash value would fill with 1s during the right shift, producing wrong finalization results. 

## Designing the PartitionManager Abstraction

In order to implement partitioning we needed an entity which:
- Can decide based on current statistics which partition to choose when key is not provided
- Provides the same partition given a key 
- Tracks the message count per partition

```java 
public class DefaultPartitionManager implements PartitionManager {

    private final AtomicLongArray currentCounts; // ensures thread-safety
    private final HashProvider hashProvider; 

    public DefaultPartitionManager(int numberOfPartition) {
        currentCounts = new AtomicLongArray(numberOfPartition);
        hashProvider = new MurmurHash3HashProvider();
    }

    @Override
    public int resolve(String key) {
        if (key == null) { // case where least-used partition strategy is used, key == null
            var leastUsedPartition = 0;
            for (var i = 1; i < currentCounts.length(); i++) {
                if (currentCounts.get(leastUsedPartition) > currentCounts.get(i)) {
                    leastUsedPartition = i;
                }
            }
            return leastUsedPartition;
        }
        // case where we have a non null key and therefore use the following formula for good distribution and consistency
        // using 0x7fffffff to avoid negative partition number
        return (hashProvider.hash(key) & 0x7fffffff) % currentCounts.length();
    }

    @Override
    public void incrementCount(int partition, long by) {
        currentCounts.addAndGet(partition, by);
    }

    @Override
    public long getCount(int partition) {
        return currentCounts.get(partition);
    }
}
```

This PartitionManager is used for producing as a consumer will always tell you which partition to read from. We therefore now have in our LogManager the following:

```java
    private Map<String, Log> logsPerPartition; // key: <topicName>-<partitionNumber>
    private Map<Topic, PartitionManager> partitionManagerPerTopics; // PartitionManger per topics

    @Override
    public AppendResponse append(Topic topic, String key, byte[] data) {
        // Get the correct PartitionManager for the given topic
        var partitionManager = partitionManagerPerTopics.computeIfAbsent(
                topic,
                _ -> new DefaultPartitionManager(partitionNumber)
        );

        // Applied the formula to the given key to find the correct partitionNo
        var partitionNumber = partitionManager.resolve(key);
        var topicRetrieved = logsPerPartition.computeIfAbsent(
                topic.name() + "-" + partitionNumber,
                it -> new LogImpl(logDirectory + "/" + it, maxSegmentSize, flushInterval)
        );

        // Append first as if an issue happened the count is not diverging
        var offset = topicRetrieved.append(data);
        partitionManager.incrementCount(partitionNumber, 1);

        // Returning the partitionNo and offset as expected by the ProducerResponse
        return new AppendResponse(partitionNumber, offset);
    }
```

Kafka uses a sticky partition strategy for batching efficiency. Since our producer doesn't batch, we chose least-loaded for simpler, even distribution

The main issue we can see here is relative to rebalancing and what would happen if we were to change the number of partition. This is out of scope for now but a solution would be to replay the log with the new partition count, though this is non-trivial and out of scope for Phase 3. 

## What's Next — Phase 4: replication

With partitioning in place, our broker can now route messages to specific partitions by key or distribute them evenly across partitions using a least-loaded strategy. This foundation is what makes replication possible.

Replication is the heart of why most firms have decided to use Kafka, paired with partitioning it allows for fault-tolerance as well as high-performance. Firms care about meeting SLAs and having a distributed system that allows them to automatically fall back without human intervention is key for continuity of service. Replication encompasses a lot of concepts on top of partitioning such as: handling multi-broker setup (who's the leader of which partition, who needs the produce request to be forwarded to), introducing concepts of metadata stored to be consumed by producers, adding high-concurrency. It is adding a lot of complexity to unlock key use-cases sought for by firms in production.