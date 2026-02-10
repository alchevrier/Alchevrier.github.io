---
layout: post
title:  "Building Mini-Kafka Phase 1: Log-Structured Storage Engine"
date:   2026-02-09 14:00:00 +0800
categories: projects distributed-systems kafka storage
---

## Introduction

In November 2025, I started a new chapter of my work-life by beginning a job that involves managing a Kafka service which connects two network-isolated data centers. The problem? I'd never had the chance to design, code, or maintain Kafka solutions. As my new job is pushing us toward AI, and after some conversations with it, I decided to implement a project in my own time: Mini-Kafka. 

The goals were:
- Put into practice the books and papers I've read about Kafka and test that knowledge
- Gain a deep understanding of Kafka's internals
- Leverage AI as a senior Software Engineer colleague: ping-pong design ideas, get documentation outlines, conduct code reviews, and use it as a supercharged search engine

I've been using AI as an educational tool, and my goal isn't to output code as fast as possible to rush a product to market. I want a lifelong learning experience that I can carry into my day-to-day work, making me better equipped to solve the problems I encounter. 

The thought process was as follows: go through each detail of the Phase 1 design, create ADRs (Architecture Decision Records), and once everything was decided, implement it. I'll start by explaining the architecture and the ADRs that were created. 

## Architecture Overview

First and foremost, the design process led me to define Phase 1 as the creation of a log storage engine, which was then proven to work by implementing a simple broker providing consumption and production capabilities. The end-to-end test was to use consumer and producer libraries, transform them into REST APIs, and call the broker endpoints to prove:
- We are able to write to the log storage engine at the provided topic
- We are able to read from the log storage engine at the provided topic and offset
- We are able to shutdown the broker and be able to recover the data on restart
- Our consumer/producer library are able to call the broker without any issues

For Phase 1 we wanted to follow the architecture below to achieve a proof-of-concept of our log storage engine: 

```
                    ┌───────────────┐
                    │ Demo Endpoint │
                    │  (REST API)   │
                    │  Spring Boot  │
                    └───────┬───────┘
                            ↓ HTTP (REST API)
     ┬──────────────────────┘────────────────────────┬
     │                                               │
     │ HTTP                                          │ HTTP
     │ (REST API)                                    │ (REST API)
     │                                               │
     ↓                                               ↓
┌──────────┐                                    ┌──────────┐
│ Consumer │                                    │ Producer │
│  Client  │                                    │  Client  │
└────┬─────┘                                    └────┬─────┘
     │                                               │
     │ HTTP                                          │ HTTP
     │ (REST API)                                    │ (REST API)
     │                                               │
     └───────────────────┬───────────────────────────┘
                         ↓
                 ┌───────────────┐
                 │   Endpoint    │
                 │  (REST API)   │
                 │  Spring Boot  │
                 └───────┬───────┘
                         │
                         │ Direct method calls
                         ↓
                 ┌───────────────────┐
                 │ Log Storage Engine│
                 │  (Core Storage)   │
                 └───────┬───────────┘
                         │
                         │ File I/O (NIO)
                         ↓
                 ┌───────────────────┐
                 │   Filesystem      │
                 │ - *.log (segments)│
                 │ - *.index         │
                 └───────────────────┘
```

The main point is leveraging the filesystem capabilities to efficiently write/read data via the Java NIO API. The goal is to achieve an easily replicable and efficient structure that can be built upon later when introducing features like replication.

## LogSegment: The Foundation

Log segments are responsible for ensuring messages are written in the correct format and keeping track of their positions via an index. Being immutable and append-only means log segments can be easily replicated and provide efficient write capabilities by design.

## Index and Log Segment

The index file maintains a mapping between an offset and the position at which it was written in the log file.

The log file contains metadata as well as the original message content.

Details about the format can be found in [ADR-003](https://github.com/alchevrier/distributed-messaging/blob/main/docs/adr/0003-use-append-only-log-storage.md).

When writing to the current log file and index file we also put the offset and filePosition in a cache for fast lookups.

### Recovery

A recovery mechanism is in place to resume writes at the correct index and provide indexes directly at restart for fast and efficient reads. When the broker restarts, the index file content is read and loaded into cache, allowing writes to resume at the correct position while providing all available offsets and file positions for consumers to read from memory. 

## Log: Segment Management

Log is responsible for keeping track of the log segment and rotating them when they reach a certain size. 

### Segment Lookup with TreeMap

Log segments are kept in a TreeMap which allows for fast lookup. When reading, `floorEntry()` is called in O(log n) time, which, given an offset, finds the segment that holds it. For writes, the last segment is always used since we only append.

Log rotation happens under a write lock to prevent writes to a segment that should have been rotated. When a certain configurable size is reached, the log rotates. This keeps log size manageable and allows for future log compaction using minimal memory. 

### Concurrency Model

Since the main goal is to build a distributed system, we need to consider concepts from foundational papers like *Time, Clocks and the Ordering of Events in a Distributed System* by Leslie Lamport.

- Only one thread can write to the log segment to ensure that event A, which happened before event B, is written first
- Multiple threads can safely read log segments concurrently when no writes are happening

To ensure this, a `ReentrantReadWriteLock` (provided by Java) is used. 

### Recovery

When starting the broker, the log directory is scanned to find all log segments written to the filesystem. The starting offsets and corresponding LogSegments are then loaded into the TreeMap, making them immediately available to consumers and producers. Writes resume at the same position and log segment as before the broker stopped. 

## LogManager: Multi-Log Orchestration

The LogManager is responsible for managing multiple topics (Logs) within the same application and ensures that writes and reads are forwarded to the correct topic.

### Directory Structure

On the filesystem, the LogManager assumes all logs are in a parent directory and loads all available topics at startup.

### Log Management

When appending to a topic, it either uses the existing log in memory or creates a new one, making it easy for producers.

When reading at a given offset for a given topic, it trusts the corresponding log to provide the correct data at that offset.

## Key Design Decisions & Trade-offs

The decisions made for Phase 1 can be found in the [ADR directory](https://github.com/alchevrier/distributed-messaging/tree/main/docs/adr). 

## Challenges & Learnings

Before starting this project, I had very little experience with:
- ADR-first, then development
- Java NIO
- Personal project development
- Developing a core system using knowledge from Kafka and academic papers, rather than building on top of existing frameworks

This was challenging because I didn't believe I could achieve it. But with AI's help to unblock me when testing ideas or when simply tired, I became much more productive. AI isn't a perfect tool—sometimes it goes on tangents—but using it to address my weaknesses provided far more learning opportunities than staying stuck would have. 

## Testing

### Unit testing

I'm proud to have more test code than production code. This helped uncover massive bugs right off the bat. Leveraging Spock and Groovy helped me craft tests in a more natural manner compared to the style I was used to with JUnit. Below is an example unit test for the log that tests the recovery mechanism:

```groovy
    def "appending more than the segment size then when appending again should append at the correct offset"() {
        when: "appending further than segment size"
            log.append("Hello World".getBytes())
            log.append("HelloW".getBytes())

            log.close()
            log = new LogImpl(logPath, TEST_SEGMENT_SIZE, FLUSH_INTERVAL)
            log.append("Should be appended at the right place".getBytes())
        then: "should be able to read all offsets anyway"
            new String(log.read(1)) == "HelloW"
            new String(log.read(2)) == "Should be appended at the right place"
            new String(log.read(0)) == "Hello World"
    }
```
Spock enables the natural given/when/then pattern and even provides the ability to label each step to further express the test's intent. 

### Integration testing

Here's the protocol used for integration testing:
- Start the demo application on port 8081, configured to call the consumer and producer APIs on port 8080
- Start the broker application on port 8080

Using Postman, the consumer and producer APIs of the demo application were called to verify that logs and their respective segments were written correctly.

Integration testing led to some eye-opening bug discoveries that even AI didn't find. When the broker restarted, the log position would be completely off and crash the application. Instead of loading offsets 1, 2, 3, 4, 5, 6 for a given log segment, it would load 1, then 344224442, then throw a BufferUnderflowException. 

The issue was with the piece of code below:

```java
    // Format of log entry: [length: 4 bytes][offset: 8 bytes][data: variable bytes]
    var buffer = ByteBuffer.allocate(4 + 8 + data.length);
    buffer.putInt(data.length);
    buffer.putLong(offset);
    buffer.put(data);
    buffer.flip();

    logChannel.write(buffer, filePosition);

    // Format of index entry: [offset: 8 bytes][filePosition: 8 bytes]
    var indexBuffer = ByteBuffer.allocate(8 + 8);
    indexBuffer.putLong(offset);
    indexBuffer.putLong(filePosition);
    indexBuffer.flip();

    indexChannel.write(indexBuffer, filePosition); // Bug: tries to write at the same filePosition as the log file
```

This subtle bug wasn't caught in unit tests because of a missing test case at the intersection of two scenarios I'd written: appending multiple messages without restart + appending one message with restart. Integration testing filled this gap and uncovered the blind spot. AI can be an amazing tool, but it's not a substitute for proper software engineering practices. 

## Conclusion

Working on this project has been very fulfilling so far—many lessons learned, failures, but also big wins like the design-first approach and exploring weaknesses in my skillset. I'm very happy with my findings despite failing multiple times in the process. Having AI by my side to ping-pong ideas or explore new possibilities I admittedly hadn't thought of made a huge difference.

See you for Phase 2, which isn't designed yet. I'll be taking a short break to recover from this first sprint of 8 days of working before and after work to make it happen. 