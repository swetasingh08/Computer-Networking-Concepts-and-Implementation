

# TCP Echo Server-Client

A Python implementation of the classic echo protocol over TCP sockets, where the server reflects back any message sent by the client. The session continues until the client sends "bye" to terminate.

---

## Overview

This project demonstrates a stateful TCP client-server interaction where multiple messages are exchanged within a single persistent connection. Unlike single-request protocols, the echo server maintains the connection and processes a sequence of messages until the client signals termination.

The echo service (well-known port 7 in the original RFC) is one of the simplest and oldest network services, traditionally used for testing network connectivity and round-trip latency. This implementation captures that fundamental testing utility while demonstrating persistent TCP connections, message loops, and graceful session termination.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/echo-server.git
cd echo-server

# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

Type messages in the client terminal. Each message will be echoed back by the server. Type "bye" to terminate the session.

---

## How It Works

### The Echo Protocol

The echo protocol follows a simple request-response pattern within a persistent connection:

1. Client connects to server
2. Client sends a message
3. Server receives the message
4. Server sends the same message back (echoes it)
5. Client displays the echoed message
6. Repeat from step 2 until client sends "bye"
7. Both sides close the connection

This is a stateful interaction: both client and server maintain the connection state across multiple message exchanges.

---

### Protocol Flow Diagram

```
Client                                          Server
  |                                               |
  |-- TCP Connect (127.0.0.1:12345) ------------> | accept()
  |                                               |
  |-- Send: "hello" -----------------------------> | recv()
  |                                               |-- Print: "hello"
  |<-- Echo: "hello" ---------------------------- | send()
  |-- Print: "Echo from server: hello"            |
  |                                               |
  |-- Send: "how are you" -----------------------> | recv()
  |                                               |-- Print: "how are you"
  |<-- Echo: "how are you" ---------------------- | send()
  |-- Print: "Echo from server: how are you"      |
  |                                               |
  |-- Send: "bye" -------------------------------> | recv()
  |                                               |-- Print: "bye"
  |                                               |-- Break loop
  |-- Break loop                                  |
  |                                               |
  |-- TCP Disconnect ----------------------------> | close()
```

---

## Architecture

### Client (client.py)

The client operates in a send-receive loop:

```
1. Connect to server at 127.0.0.1:12345
2. Display connection status and usage instructions
3. Loop indefinitely:
   a. Read user input from console
   b. Send message to server
   c. If message is "bye" (case-insensitive):
         Break the loop and terminate
   d. Otherwise:
         Receive the echoed response from server
         Display the echoed message
4. Close the socket connection
```

Key design decisions:

- The "bye" check happens immediately after sending, so the client does not wait for an echo of "bye"
- The recv(1024) call blocks until the server responds, creating natural synchronization
- The loop runs until explicitly broken by the "bye" condition
- Case-insensitive comparison (msg.lower() == "bye") allows "BYE", "Bye", etc.

### Server (server.py)

The server operates in a receive-send loop for each client:

```
1. Create socket and bind to 127.0.0.1:12345
2. Listen for incoming connections (backlog of 1)
3. Accept a single client connection
4. Loop indefinitely:
   a. Receive data from client (up to 1024 bytes)
   b. If no data received (client disconnected):
         Break the loop
   c. Print the received message
   d. If message is "bye" (case-insensitive):
         Break the loop
   e. Otherwise:
         Send the same data back to client (echo)
5. Close the client connection
6. Close the server socket
```

Key design decisions:

- The server processes only one client connection per run (single accept() call)
- Empty data (not data) indicates the client closed its end of the connection
- The server echoes back the exact bytes it received, not a modified version
- Both the client and server check for "bye" independently to break their respective loops

---

## Connection Lifecycle States

### Client States

```
CLOSED --> socket()
CREATED --> connect()
CONNECTED --> [send/recv loop] --> "bye" sent --> CLOSED
           \-> Connection error --> CLOSED
```

### Server States

```
CLOSED --> socket()
CREATED --> bind()
BOUND --> listen()
LISTENING --> accept() [blocks here]
CONNECTED --> [recv/send loop] --> "bye" received --> CLOSED
           \-> Client disconnects --> CLOSED
```

---

## Sample Output

Server console:

```
Echo Server started on port 12345
Client connected: 127.0.0.1
Received: hello
Received: how are you
Received: bye
```

Client console:

```
Connected to Echo Server
Type a message to send to the server (type 'bye' to quit):
hello
Echo from server: hello
how are you
Echo from server: how are you
bye
```

---

## Use Cases for Echo Servers

### 1. Network Connectivity Testing

An echo server confirms that:
- The network path between client and server is functional
- TCP connections can be established
- Data can be transmitted in both directions
- The server application is running and responsive

### 2. Round-Trip Time Measurement

By timestamping messages before sending and after receiving the echo, the client can measure end-to-end latency:

```
import time

start = time.time()
client.send(msg.encode())
echo = client.recv(1024)
rtt = time.time() - start
print(f"Round-trip time: {rtt*1000:.2f} ms")
```

### 3. Throughput Testing

Sending large messages or many messages in rapid succession can help measure throughput and identify bottlenecks.

### 4. Protocol Debugging

When developing new network protocols, an echo server provides a known-good endpoint for testing client implementations.

---

## Comparison: Echo vs Other Simple Protocols

| Protocol          | Server Behavior                        | Use Case                    |
|-------------------|----------------------------------------|-----------------------------|
| Echo              | Returns exact copy of received data    | Connectivity testing, RTT   |
| Discard           | Receives data, sends nothing           | Throughput testing          |
| Chargen           | Sends continuous stream of characters  | Load testing                |
| Daytime           | Sends current date and time            | Time synchronization        |
| Quote of the Day  | Sends random quotation                 | Entertainment/testing       |

---

