

# TCP Socket Communication (Client-Server Hello)

A minimal Python client-server application demonstrating fundamental TCP socket communication by exchanging greeting messages over a loopback connection.

---

## Overview

This project implements the simplest possible TCP client-server interaction: a client connects to a server, sends a greeting message, receives a response, and disconnects. Despite its simplicity, it demonstrates every essential element of TCP socket programming in Python.

This serves as the foundation for understanding more complex networked applications. Every protocol simulation, file transfer, and remote procedure call builds upon the core concepts demonstrated here: socket creation, connection establishment, data transmission, and connection teardown.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/tcp-hello.git
cd tcp-hello

# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

---

## How It Works

### The TCP Connection Lifecycle

```
Client                                           Server
  |                                                |
  | socket() ---- Create socket                    | socket() ---- Create socket
  |                                                | bind() ------ Bind to port 12345
  |                                                | listen() ---- Wait for connections
  |                                                |
  | connect() --- Request connection ------------> | accept() ---- Accept connection
  |                                                |
  | send() ------ "Hello Server" ----------------> | recv() ------ Receive message
  |                                                |
  | recv() <----- "Hello from server" ------------ | send() ------ Send response
  |                                                |
  | close() ----- Terminate connection ----------> | close() ----- Close connection
```

### Client (client.py)

The client performs these sequential steps:

1. Creates a TCP socket using socket.socket()
   - AF_INET specifies IPv4 addressing
   - SOCK_STREAM specifies TCP (reliable, connection-oriented)
2. Connects to the server at localhost on port 12345
3. Sends the message "Hello Server" encoded as bytes
4. Receives the server's response (up to 1024 bytes) and decodes it to a string
5. Prints the server's message
6. Closes the socket connection

Code breakdown:

```
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 12345))

client.send("Hello Server".encode())

data = client.recv(1024).decode()
print("Server says:", data)

client.close()
```

### Server (server.py)

The server performs these sequential steps:

1. Creates a TCP socket
2. Binds the socket to localhost on port 12345
   - Binding assigns a specific network interface and port to the socket
   - localhost (127.0.0.1) restricts connections to the local machine only
3. Listens for incoming connections with a backlog of 1
   - The backlog parameter limits the queue of pending connections
4. Enters the accept state, blocking until a client connects
5. Accepts the connection, which returns:
   - conn: a new socket object for communicating with this specific client
   - addr: the client's address tuple (IP, port)
6. Receives the client's message (up to 1024 bytes)
7. Prints the received message
8. Sends the response "Hello from server" back to the client
9. Closes the connection socket

Code breakdown:

```
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 12345))
server.listen(1)

print("Server is waiting for connection...")

conn, addr = server.accept()
print("Connected by", addr)

data = conn.recv(1024).decode()
print("Client says:", data)

conn.send("Hello from server".encode())
conn.close()
```

---

## Sample Output

Server console:

```
Server is waiting for connection...
Connected by ('127.0.0.1', 54321)
Client says: Hello Server
```

Client console:

```
Server says: Hello from server
```

---

## Key Concepts Demonstrated

### 1. Socket Creation

```
socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

- AF_INET: Address Family for IPv4. Use AF_INET6 for IPv6.
- SOCK_STREAM: Socket type for TCP. Use SOCK_DGRAM for UDP.

### 2. Encoding and Decoding

```
"Hello Server".encode()    # String to bytes (default UTF-8)
data.decode()              # Bytes to string (default UTF-8)
```

Network sockets transmit raw bytes, not strings. Encoding converts human-readable text to bytes for transmission; decoding reconstructs the original text on the receiving end.

### 3. Port Selection

- Port 12345 is an unprivileged port (above 1024), chosen arbitrarily for this demonstration
- Only one application can bind to a given port at a time
- Ports below 1024 require root/administrator privileges on most systems

### 4. The accept() Return Value

```
conn, addr = server.accept()
```

- conn: A new socket dedicated to this specific client connection. The server can continue listening on the original socket for additional clients.
- addr: A tuple containing (IP_address, port_number) of the connected client.

### 5. recv() Buffer Size

```
data = conn.recv(1024)
```

The argument (1024) specifies the maximum number of bytes to receive in a single call. TCP is a stream protocol, so a single send() may require multiple recv() calls to receive all data — though for this short message, one call suffices.

---

## Server Lifecycle States

A TCP server transitions through several distinct states during its lifecycle:

```
CLOSED -> socket()
BOUND -> bind()
LISTENING -> listen()
ACCEPTING -> accept() [blocks here]
CONNECTED -> [send/recv with client]
CLOSED -> close()
```

Understanding these states is essential for debugging connection issues and implementing robust servers.

---

## Common Issues and Solutions

### "Address already in use" Error

```
OSError: [Errno 98] Address already in use
```

This occurs when port 12345 is still held by a previous instance of the server. Solutions:

- Wait for the operating system to release the port (typically 30-120 seconds)
- Use SO_REUSEADDR to allow immediate reuse:

```
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

### "Connection refused" Error

```
ConnectionRefusedError: [Errno 111] Connection refused
```

This means the server is not running or not listening on the specified port. Ensure server.py is executing before running client.py.

### Message Truncation

For longer messages, a single recv() call may not return all the data. Production code should loop until the complete message is received or implement a length-prefix protocol.

---

## Extending the Project

This minimal example can be extended in many directions:

- Handle multiple clients sequentially by placing accept() in a while loop
- Handle multiple clients concurrently using threading or asyncio
- Implement a simple chat system by looping send/recv on both sides
- Add message length prefixing for reliable transmission of larger messages
- Replace localhost with an actual IP address for network communication
- Add SSL/TLS encryption using the ssl module
- Implement a request-response protocol with structured messages (JSON)

Multi-client server example:

```
while True:
    conn, addr = server.accept()
    print("Connected by", addr)
    data = conn.recv(1024).decode()
    print(f"{addr} says:", data)
    conn.send("Hello from server".encode())
    conn.close()
```

---

## File Structure

```
.
├── server.py          # TCP server (single connection)
├── client.py          # TCP client (single request)
└── README             # This file
```

---

## Comparison Table: TCP vs UDP

| Feature          | TCP (Used Here)                       | UDP                                |
|------------------|---------------------------------------|------------------------------------|
| Connection       | Connection-oriented (connect/accept)  | Connectionless                     |
| Reliability      | Guaranteed delivery, ordering         | No guarantees                      |
| Data Boundary    | Stream (no message boundaries)        | Datagram (preserves boundaries)    |
| Speed            | Slower (overhead for reliability)     | Faster (no overhead)               |
| Use Cases        | Web, email, file transfer, SSH        | Video streaming, DNS, VoIP, gaming |
| Socket Type      | SOCK_STREAM                           | SOCK_DGRAM                         |

