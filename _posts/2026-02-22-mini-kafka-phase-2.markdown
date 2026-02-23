---
layout: post
title:  "Building Mini-Kafka Phase 2: Binary Protocol"
date:   2026-02-22 14:00:00 +0800
categories: projects distributed-systems kafka storage
---

## Introduction

In early February 2026, I shipped Phase 1 of the mini-Kafka project which is the stepping-stone upon which I want to work on and enrich until obtaining a project with very similar qualities/attributes as the real Kafka. We have developed a append-only log structured messaging system that could be access via REST API which was implemented in Java 25/Spring Boot 4. We did the REST API for speed-purposes to proof that our storage engine was working as expected. 

For Phase 2, we ramp up by bringing a new component which is meant to be a key element of what Kafka provides in order to achieve high-performances: a binary protocol. We decided to keep the REST API and build a TCP-based binary protocol using Java NIO alongside the reasoning being that both should be able to be interoperable which would be our proof of concept. Let's walk through how it was built, step by step with the help of my pair-programmer GitHub Copilot! 

## The Protocol Design 

As for Phase 1 our approach is to first design then code but to design one must first understand:
- How to achieve a TCP-based server/client? Main ideas came from this great blog post: https://eli.thegreenplace.net/2011/08/02/length-prefix-framing-for-protocol-buffers
- What is Kafka doing in terms of messaging? Directly from the source: https://kafka.apache.org/41/design/protocol/#request-and-response-headers

We will focus here on the messaging and later on the tcp components. 

Ahead of time it very important to understand:
- What types of messages we want to be sending over the wire? 
- How to keep a prefix consistent so that the receiver can know how many bytes needs to be consumed to read the data on the wire and also what is the message type that he received. This will drive the serialization and deserialization. 

To illustrate better:

```
[client sends requests as byte[]] -- TCP (host, port) --> [server receives]

When receiving: 
- Read the first 4 bytes to get the messageLength
- Read messageLength bytes to get the full message payload
- Read the first byte to understand what is the message type and act accordingly
```

From there we need to define the different messages we want to be passing on the wire from client to server and server to client. Thankfully we had already developed a REST API from Phase 1 and have clarity on those and we defined the message types as:

```
public class MessageType {
    /**
     * 01: CONSUME_REQUEST
     * 02: CONSUME_RESPONSE
     * 03: PRODUCE_REQUEST
     * 04: PRODUCE_RESPONSE
     * 05: FLUSH_REQUEST
     * 06: FLUSH_RESPONSE
     */
    public static final byte CONSUME_REQUEST = 0x01;
    public static final byte CONSUME_RESPONSE = 0x02;
    public static final byte PRODUCE_REQUEST = 0x03;
    public static final byte PRODUCE_RESPONSE = 0x04;
    public static final byte FLUSH_REQUEST = 0x05;
    public static final byte FLUSH_RESPONSE = 0x06;

}
```

From there we also need to know the exact format of each messages ahead of time in order to speed up the process of writing serialization and deserialization code and here's the extract from our ADR-09

### PRODUCE Request
```
[length: 4][type: 03][4 bytes: topic length][N bytes: topic UTF-8][4 bytes: data length][N bytes: data]
```

### PRODUCE Response
```
Success path: [length][type: 04][success: 1][offset: 8]
Error path:   [length][type: 04][success: 0][4 bytes: error length][N bytes: error UTF-8]
```

### CONSUME Request
```
[length: 4 bytes][message type: 01][4 bytes: topic length][N bytes: topic UTF-8][8 bytes: offset (long)][4 bytes: batch size (int)]
```

### CONSUME Response
```
Success: [length][type: 02][success: 1][8 bytes: next offset][4 bytes: message count][for each: [4 bytes: msg length][8 bytes: offset][N bytes: data]]
Error:   [length][type: 02][success: 0][4 bytes: error length][N bytes: error UTF-8]
```

### FLUSH Request
```
[length: 4 bytes][message type: 05]
```

### FLUSH Response
```
Success: [length][type: 06][success: 1]
Error:   [length][type: 06][success: 0][4 bytes: error length][N bytes: error UTF-8]
```

Clear and simple, I have defined for each responses the case of success or error which will be key for us when reading from a byte[] to understand how to move forward in the serialization and deserialization. Defining the exact byte layout upfront meant that when I sat down to write ByteBufferSerializer, there were zero decisions left to make — just translate the spec into code. I would advise anyone reading this post to learn how to ADRs because this is a key for avoiding issues and speeding the development process without any AI magic (such as massive code generation). Having as much clarity as possible is key for the success of a project. 

## ByteBuffer Serialization

One of the main decisions here was to copy Kafka by making our own binary protocol without the help of existing libraries such as Protobuf. We we therefore implemented a serialize and deserialize function for each message the idea being one to be called by the initiator and the other by the receiver. 

Java NIO and more precisely ByteBuffer which is the key class used in this section.

Basically what we do is take the original request and then use its definition from the ADR to define which data comes at which bytes. For serialization we would be using putInt(), put(), putLong() mostly and for deserialization we would be using get(), getInt(), getLong(). Reading/Writing to a ByteBuffer is stateful, we have a pointer which knows where is the next position to read/write. When getting the output of the serialized message we can just ask for the underlying array using array() and we get a byte[] which corresponds to ALL the data we have written from the start.

What's important to note is that when the server or the client reads the message length it reads the first 4 bytes on the wire then needs to flip the ByteBuffer to reset the position to the start in order for us to be able to read it. Flipping resets position to 0 so getInt() reads from the beginning. Without flip(), position is still at 4 (end of what was written) and getInt() reads garbage or throws. Here's a sample code below from the tcp client: 

```java
private int readResponseLength() throws IOException {
        var responseLengthBuffer = ByteBuffer.allocate(4);
        while (responseLengthBuffer.hasRemaining()) {
            var read = channel.read(responseLengthBuffer);
            if (read == -1) {
                throw new IOException("Connection closed by server");
            }
        }
        responseLengthBuffer.flip(); // Here must flip or get else will get BufferOverflow error or garbage

        return responseLengthBuffer.getInt();
    }
```

The ADR defined every byte position upfront — serialization was just a mechanical translation of that spec into putInt(), put() and putLong() calls.

One of the main issue I encountered was off-by-4 where I ran my test to consume data that was produce by the REST API and the client code was just hanging. The issue was that when we read we do the following: 
- Read the message length
- Read the rest 
And what happened is that the message length represents the length of the whole message, including the length field which is exactly 4 bytes. Here's the sample code from the tcp-client:

```java
private byte[] readResponse() throws IOException {
    // BEFORE: var responseMessageBuffer = ByteBuffer.allocate(readResponseLength());
    var responseMessageBuffer = ByteBuffer.allocate(readResponseLength() - 4); // AFTER
    while (responseMessageBuffer.hasRemaining()) {
        var read = channel.read(responseMessageBuffer);
        if (read == -1) {
            throw new IOException("Connection closed by server");
        }
    }

    return responseMessageBuffer.array();
}
```

Imagine I was allocation for 64 bytes for examples and reading for 60 but then I was reading again because I was missing the 4 bytes which then meants I would wait for 4 bytes that would never come. When designing your own protocol there is no safety net, you need to understand how many bytes you allocate, where you allocate it and how you are meant to be consuming it. The fix here was to just remove the 4 bytes from the allocated ByteBuffer to finally read just the 60 bytes we needed to read.

## TCP Server

Since the main point is to develop a binary protocol we need a TCP Server that can accept a connection, serve requests on it indefinitely, and recover cleanly when the client disconnects. Our use case is simple for this proof of concept: one server for one client. When the client disconnects we wait for another connection and gracefully shutdown when the broker shutdown.

```
[ServerSocketChannel] --> (bind InetSocketAddress then accept) --> [SocketChannel] -- while (serverNotGracefullyShutdown) --> (handleRequest) 

No client connection

while (serverNotGracefullyShutdown)
    [ServerSocketChannel] --> (bind InetSocketAddress then accept) --> null --> Thread.sleep(10) // avoiding spin locking 

When client disconnected
[ServerSocketChannel] --> (bind InetSocketAddress then accept) --> [SocketChannel] -- while (serverNotGracefullyShutdown) --> (handleRequest) --> (read == -1) --> throw IOException (then we end up in the no client connection loop)
```

Handling a request is a matter of 3 operations as shown in the snippet below:

```java
var messageLength = getMessageLength(socketChannel);
var request = getRequest(messageLength, socketChannel);
writeResponseToClient(request, socketChannel); 
```

As mentioned before we need to first read the first 4 bytes of the request to understand how long the request will be then read the rest. In the end we will use the byte[] and delegate to ServerHandler to handle it, it is meant to be implemented by the broker. We do this to respect SRP:
- TCP Server project handles how to work around the Java API to handle TCP requests and send the response
- Broker defines the implementation of the ServerHandler without knowledge of where the data comes from 

```java
// TCP Server side
private void writeResponseToClient(byte[] request, SocketChannel socketChannel) throws IOException {
    var response = serverHandler.handle(request); // broker implements this interface
    var responseBuffer = ByteBuffer.wrap(response);
    while (responseBuffer.hasRemaining()) {
        socketChannel.write(responseBuffer);
    }
}

// Broker side
public class BrokerEndpointHandler implements ServerHandler {
    // omitted code just to focus on handle method
    // We simply care about what is the message type and how to handle it accordingly
    @Override
    public byte[] handle(byte[] message) {
        var buffer = ByteBuffer.wrap(message);
        var messageType = buffer.get();
        return switch (messageType) {
            case CONSUME_REQUEST -> handleConsumeRequest(message);
            case PRODUCE_REQUEST -> handleProduceRequest(message);
            case FLUSH_REQUEST -> handleFlushRequest(message);
            default -> throw new IllegalArgumentException("Did not recognize messageType for type: " + messageType);
        };
    }
}
```

Worth noting here that we use Java NIO in a blocking manner as we have one client if we were to support multiple clients then we would need to use Selector or to have multiple Virtual Threads running the TCP Server each time with a different port for each clients we wish to use. The Selector route is something we will explore when working on Phase 4 replication. 

## TCP Client

In the same manner as TCP Server to respect SRP:
- TCP Client handles how to write to the server via TCP and how to read the server using Java NIO
- Client provides the request and the way to serialize/deserialize it

```java
public <Req, Res> Res forwardToServer(Req message, Function<Req, byte[]> serializer, Function<byte[], Res> deserializer) throws IOException {
    if (this.channel == null) {
        connectToServer(); // connecting/reconnecting to the server when needed
    }
    forwardToServer(serializer.apply(message)); // simply writing the bytes to the socket channel
    return deserializer.apply(readResponse()); // getting the response length and reading the remaining 
}
```
As you can see we do connect to the server on demand, the main idea is when wiring to an application if the broker is down then the application would not crash at startup. Letting the application using the consumer or producer library to gracefully handle this either by backing-off for example.  

This leaves us with the consumer and producer libraries which simply use the client to send their request:

```java
// consumer
public class TcpMessageConsumer implements MessageConsumer {
    // Omitting setup code
    @Override
    public ConsumeResponse consume(Topic topic, long startOffset, int batchSize) {
        try {
            // client is an instance of TcpClient
            return client.forwardToServer(
                    new ConsumeRequest(topic, startOffset, batchSize), // the request
                    serializer::serialize, // how to serialize it 
                    deserializer::deserializeConsumeResponse // how to deserialize it
            );
        } catch (IOException e) {
            return new ConsumeResponse(null, null, e.getMessage());
        }
    }
}

//producer
public class TcpMessageProducer implements MessageProducer {
    // Omitting setup code
    @Override
    public ProduceResponse produce(ProduceRequest produceRequest) {
        try {
            // client is an instance of TcpClient
            return client.forwardToServer(
                produceRequest,  // the request
                serializer::serialize,  // how to serialize it 
                deserializer::deserializeProduceResponse // how to deserialize it
            );
        } catch (IOException e) {
            return new ProduceResponse(null, e.getMessage());
        }
    }
}
```

## Spring Wiring

Given client.tcp.host, client.tcp.port, and client.tcp.enabled=true, Spring will automatically wire a TcpClient bean along with the corresponding MessageProducer or MessageConsumer

One of the gotcha here is making sure that only ONE tcp client bean exists and this can be done by using conditional such as
```java
@Bean(destroyMethod = "close")
@ConditionalOnMissingBean(TcpClient.class)
public TcpClient tcpClient(@Value("${client.tcp.host}") String host, @Value("${client.tcp.port}") int port) {
    return new TcpClient(host, port);
}
```

And we also provide annotations to seamlessly import the configuration as well as not forcing auto-configuration when putting the libraries in the classpath in the case both libraries are included in your project leaving you control over how you want to include the consumer and producer. 

```java
// consumer
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TcpConfiguration.class)
public @interface EnableTcpConsumer {
}

// producer
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TcpProducerConfiguration.class)
public @interface EnableTcpProducer {
}
```

For the broker, we simply added a tcp server bean which could gracefully shutdown an implementation of the ServerHandler interface exposed as a bean to handle incoming byte[] and a TcpServerRunner which is meant to start the TCP Server on a virtual thread when the ApplicationReadyEvent is fired by Spring Boot.

```java
@Bean(destroyMethod = "close")
public TcpServer tcpServer(
        @Value("${binary.server.port}") int tcpPort,
        BrokerEndpointHandler brokerEndpointHandler // implementation of the ServerHandler interface exposed as bean
) {
    return new TcpServer(tcpPort, brokerEndpointHandler);
}

@Component
public class TcpServerRunner implements ApplicationListener<ApplicationReadyEvent> {

    private static final Logger LOGGER = LoggerFactory.getLogger(TcpServerRunner.class);

    private final int tcpPort;
    private final TcpServer tcpServer;

    public TcpServerRunner(
            @Value("${binary.server.port}") int tcpPort,
            TcpServer tcpServer
    ) {
        this.tcpPort = tcpPort;
        this.tcpServer = tcpServer;
    }

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        Thread.ofVirtual()
                .name("tcp-server")
                .start(() -> {
                    LOGGER.info("Starting TCP server at port: {}", tcpPort);
                    tcpServer.start();
                });
    }
}
```

## Interoperability Test

We already had extensive test cases in our integration test suite which were meant to be ran in order via Stepwise. Our main goal in the integration was to be interoperable with the REST API, therefore we simply decided to add in-between new test cases without interfering with the existing test cases. The only modification we have made is our concurrent production of messages in which we decided to use TCP instead of REST. We ended with the following suite:
- Produce ONE message via REST API
- Consume ONE message via REST API
- (NEW) Consume ONE message via TCP
- (MODIFIED) Concurrently produce 99 messages via TCP
- Consume concurrently all the messages via REST API

Having everything all green when running proves the interoperabilty, that consumer and producer client are able to work with our TCP broker. Our objectives are all met for Phase 2. 

## What's Next — Phase 3: partitioning

Phase 3 is all about the building block for what comes next: replication. Partitioning is what allows the fault tolerance aspect of kafka, without it replication cannot be done. Partitioning is all about spreading high-volume of messages across multiple partition and then later on have different leaders in charge of being caught up to the latest messages sent and have their followers catching up. Partitioning in Kafka is done in an hybrid way, either you provide a key for a producer record and then it defines according to a hash its partition or you don't provide a key and then the partition with the least load is picked. 