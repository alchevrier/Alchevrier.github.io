---
layout: post
title:  "Building Mini-Kafka Phase 4: How I fixed flaky Raft integration tests"
date:   2026-04-11 14:00:00 +0800
categories: projects distributed-systems kafka storage
---

## Introduction

One of the most challenging aspects that I had encountered during implementing Raft is integration testing. While the test cases themselves are straightforward there are many aspects that are being put to the tests: 
- How do my Raft implementation handles over three nodes the test cases? 
- Are those pass really pass or if I run them long enough I will see that actually under certain circumstances they yield a different outcome? 
- In the eventuality of flaky tests what is really the root cause? 


## The symptom: happy-path election and appending under one node failure

Raft's main premise is strong leader and an election process meant to be correct and fair. Therefore the first test I wrote was the following:

```groovy
    def "startElection - a leader should be elected and have two followers"() {
        when: "starting the different node participants"
            testHarness.start()
            firstFallbackTestHarness.start()
            secondFallbackTestHarness.start()
        then: "a leader should have been elected with two followers"
            await().atMost(2000, TimeUnit.MILLISECONDS).until {
                def nodes = [testHarness, firstFallbackTestHarness, secondFallbackTestHarness]
                nodes.count { it.raftNode.state == RaftState.LEADER } == 1 &&
                        nodes.count { it.raftNode.state == RaftState.FOLLOWER } == 2
            }
    }
```

Our test harness when calling start does the following:
- Start the election timer for the underlying RaftNode
- Start the Raft server (intercommunication server - sending/receiving heartbeats | triggering leader election | receiving votes)
- Start the Broker server (append entries)

So what we are doing is waiting for the election to be triggered by one of the nodes (leader election occurs when followers do not receive a transmission from the leader within a given random number between a lower and upper bound - per paper it is 150ms and 300ms). Thus here counting the LEADER and FOLLOWERS count as the order is not given. 

Also one of the promises of Raft is to provide a way to replicate data accross multiple nodes. This is done no matter what the AckMode is but the differences per AckMode is a matter of what the producer cares about: 
- AckMode.NONE: Fire-and-forget (at most once), whether or not the entry has been appended and replicated does not matter to me I will continue to append other records. This is maximum throughput, I get an answer immediately and can move forward.
- AckMode.LEADER: At least-once, I care about the LEADER having my record whether or not it is commited is another matter
- AckMode.ALL: I care about my entry being committed, it is obviously affecting throughput but correctness is what matters to me. 

The test presented here is about using AckMode.ALL and having one node down in a cluster of three. In Raft papers, an entry is commited only if received by a majority of nodes. Therefore a node down should not affect whether or not an entry is commited which is the purpose of the test. AckMode.ALL just ensure that when the leader append entries we have the correct picture of the non-down peer having successfully replicated the entry and the node-down being non respondant. Here's the test here:

```groovy
    def "append - appending to a leader with ACK=ALL while one node is done should be be successful to majority of peers"() {
        when: "append request on the leader with ACK=ALL"
            def leader = findLeader()
            def downNode = ([testHarness, firstFallbackTestHarness, secondFallbackTestHarness] - leader)[0]
            downNode.shutdown()
            leader = captureCurrentLeader()
            def response = leader.append(new AppendRequest("first", new byte[][] { "hello".getBytes(), "world".getBytes() }, AckMode.ALL))
        then: "everyone should have commited the entries"
            response.success()
            def remainingNode = [testHarness, firstFallbackTestHarness, secondFallbackTestHarness] - leader - downNode
            remainingNode.forEach {  response.peersAck().get(findNodeId(it)) }
            !response.peersAck().get(findNodeId(downNode))
    }
```

The reason why I presented those two tests is that they share a commonality while being correct they were both flaky. If you were to run those two tests multiple times, they would either pass, fail or hang. It was discouraging at first to find this out but I had to figure out what was the cause of this and this is the purpose of this post. 

## Three fixes, one root cause

Working with Copilot we came to a first obvious choice which was to adjust the lower/upper bound of the leader election to make it work within the bound of the test execution. As we wait for 2000ms in the await clause, setting a 500/1000 ms time window for a leader election will trigger most likely a single election during the test. Widening the window reduced the probability of a second election completing before the await() timeout expired — it did not prevent spurious elections from starting. The tests were less flaky, not fixed.

My focus moved into the node-down test, I was focused on the change of leaders and same for Copilot. The only explanation for the test to be flaky is that somehow the leader must have changed and this is very important to figure out why because in production we definitely do not want a leader changing despite having a healthy leader communicating to its peer. 

One of the first idea was because we were running the tests as a suite to figure out whether there was a cascading failure causing this. The way a test work in Spock is the following: setup -> test (given,when,then) -> cleanup. I've not shown before the setup not cleanup part of my integration test: 

```groovy 
    def setup() {
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node1/index/currentIndex.raftindex"))
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node1/logger/currentLog.raftlog"))
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node2/index/currentIndex.raftindex"))
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node2/logger/currentLog.raftlog"))
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node3/index/currentIndex.raftindex"))
        Files.deleteIfExists(Path.of("/tmp/mini-kafka/raft/node3/logger/currentLog.raftlog"))

        def node1PropertiesPath = Thread.currentThread().getContextClassLoader().getResource("node1.properties").file
        testHarness = new ClusterRaftNodeTestHarness(9182, new ClusterRaftNodeFactory().buildFromFile(node1PropertiesPath))
        def node2PropertiesPath = Thread.currentThread().getContextClassLoader().getResource("node2.properties").file
        firstFallbackTestHarness = new ClusterRaftNodeTestHarness(9185, new ClusterRaftNodeFactory().buildFromFile(node2PropertiesPath))
        def node3PropertiesPath = Thread.currentThread().getContextClassLoader().getResource("node3.properties").file
        secondFallbackTestHarness = new ClusterRaftNodeTestHarness(9187, new ClusterRaftNodeFactory().buildFromFile(node3PropertiesPath))
    }

    def cleanup() {
        testHarness.shutdown()
        firstFallbackTestHarness.shutdown()
        secondFallbackTestHarness.shutdown()
    }
```

And the test harness:

```java
    public void shutdown() {
        electionTimerService.stop();
        heartbeatTimerService.stop();
        raftServer.close();
        brokerServer.close();
    }
```

Here we see that we stop the timers then the servers this poses racing conditions:
- Since the RaftServer is still alive it is still capable of serving inter-node requests -> this would throw exception (RejectedExecutionException ) internally due to the timer being backed by a scheduledExecutorService which already been shutdown
- Being able to serve inter-node communications while shutting down pauses another set of issues such as additional racing conditions and timer reset which would eventually lead to the same error as above
- Another important one is about correctness: a leader still has pendingAppends in its state and shutting down the leader should fail-fast those or else followers will have multiple TCP connections pending waiting to be timing out. 

Despite the fixes done here to shutdown, the flaky test was still not passing consistently, leader was still changing hands and we have eliminated cross-test contamination. Only one issue remained, was my healthy follower receiving my heartbeats? According to the original code this would unfortunately not be the case: 

```java
    public void sendHeartbeats() {
        this.lock.lock();
        try {
            peers.forEach(this::sendAppendEntriesUnlocked);
        } finally {
            this.lock.unlock();
        }
    }
```

When a node was down if the node down is down the hierarchy, there was no way the leader could notify the peer on time before it triggers a new election and becomes the new leader. A wrong fix could have been to tinker further with timeouts to make it work somehow but then it is a technical debt and possible loss of revenue for the user of the application: not acceptable. The correct solution was to do the following:
- Build the requests under lock and update the last sent index per peers
- Per peers, start a virtual thread and send the heartbeat hence by passing the wait for a timeout from a previous node before sending the next heartbeat

Raft assumes H << T << MTBF — heartbeat interval must be much less than election timeout. Serial dispatch violated the left side of that inequality whenever a peer was down. Therefore the following changes were made to be Raft-compliant for this scenario:

```java
    public void sendHeartbeats() {
        List<Map.Entry<RaftPeer, AppendEntriesRequest>> requests;
        this.lock.lock();
        try {
            requests = peers.stream().map(peer -> {
                var request = buildAppendEntriesRequest(peer);
                leaderState.setLastSentIndex(peer.nodeId(), log.getLastIndex());
                return Map.entry(peer, request);
            }).toList();
        } finally {
            this.lock.unlock();
        }
        requests.forEach(req ->
            Thread.ofVirtual().start(() -> raftClient.appendEntries(req.getKey(), req.getValue()))
        );
    }
```

This fix should have been the fix that changed everything, unfortunately this wasn't the case. Both of the test presented were failing and I was left with no further explanation based on what I implemented. The issue was nowhere to be found in the codebase and Copilot suggested to circle back to the Raft paper itself. The question was: What makes followers reset their election timer? Paragraph 5.2 says it all: 

```
Followers (§5.2):
• Respond to RPCs from candidates and leaders
• If election timeout elapses without receiving AppendEntries RPC from current leader or granting vote to candidate convert to candidate
...
Once a candidate wins an election, itbecomes leader. It then sends heartbeat messages to all ofthe other servers to establish its authority and prevent new elections.
```

Here it was the missing key implementation detail that I missed, a follower must reset its timer when granting a vote or when receiving a heartbeat from the leader. I was not Raft-compliant up until this point which explained why my tests were flaky. It explains also why my happy-path test scenario was flaky:

- Followers not resetting their timer despite receiving heartbeats → they time out
- A follower starts an election → increments its term, sends RequestVote
- §5.1: the current leader receives a message with a higher term → correctly converts to follower
- Cluster is leaderless mid-test → your assertion races against re-election

This highlights that I was not Raft-compliant under a healthy leader scenario hence making my Raft implementation not valid. This fix finally fixed the flaky tests and the full build was green afterwards. 

## Conclusion

If your tests are still flaky after fixing shutdown ordering and heartbeat dispatch, stop looking at your code. Go back to §5.2 of the paper. The answer is almost certainly there. Look for places in your implementation where you might not be Raft-compliant and go back to the paper as a reference. 