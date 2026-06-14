

# RPC (Remote Procedure Call) Simulation

A Python socket-based client-server simulation demonstrating the Remote Procedure Call (RPC) paradigm, where a client invokes functions on a remote server as if they were local calls.

---

## Overview

This project implements a simplified RPC system using TCP loopback sockets. The server exposes arithmetic operations (addition and subtraction) as remotely callable procedures. The client sends a text-based command specifying the function name and arguments, and the server executes the corresponding function and returns the result.

This simulation captures the core abstraction of RPC: making a function call across a network boundary transparent to the caller. While production RPC frameworks use serialization formats like JSON, Protocol Buffers, or XML, this implementation uses a simple space-delimited text protocol for clarity and educational purposes.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/rpc-simulation.git
cd rpc-simulation

# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

When prompted, enter an RPC call in the format: operation value1 value2

Examples:
- add 5 3
- sub 10 7

---

## How It Works

### The RPC Abstraction

In traditional local programming, calling a function is straightforward:

```
result = add(5, 3)    # Direct function call
```

RPC extends this model across a network by marshaling the function name and arguments, transmitting them to a remote server, executing the function there, and returning the result:

```
Client:  "add 5 3"  --[network]-->  Server: add(5, 3) -> 8  --[network]-->  Client: 8
```

The client experience remains simple — provide input, receive output — while the network communication is handled transparently.

---

### Protocol Design

The RPC protocol uses a simple space-delimited text format:

```
<function_name> <argument1> <argument2>
```

- function_name: "add" or "sub" (case-sensitive)
- argument1: integer value
- argument2: integer value

The server responds with a plain text result (the computed integer, or "Invalid function" for unrecognized operations).

---

### Client (client.py)

The client performs the following steps:

1. Creates a TCP socket and connects to the server at localhost:818
2. Prompts the user to enter an RPC call in the format "add 5 3" or "sub 10 7"
3. Sends the entire command string to the server using sendall() for reliable delivery
4. Receives the result string from the server (up to 1024 bytes)
5. Displays the result and closes the connection

Code walkthrough:

```
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("localhost", 818))

msg = input("Enter RPC call (e.g. add 5 3): ")
client.sendall(msg.encode())          # Encode string to bytes

result = client.recv(1024).decode()   # Receive and decode response
print("Result:", result)

client.close()
```

Key design decisions:
- sendall() ensures the complete message is transmitted (unlike send() which may send partial data)
- recv(1024) reads up to 1KB — more than sufficient for integer results
- The connection is short-lived (one request-response cycle per client run)

---

### Server (server.py)

The server performs the following steps:

1. Defines local functions add(a, b) and sub(a, b) that perform arithmetic
2. Creates a TCP socket, binds to localhost:818, and listens for connections
3. Enters an infinite loop to continuously accept and serve clients:
   - Receives the command string from the client
   - Splits the string into function name and arguments
   - Converts arguments from strings to integers
   - Dispatches to the appropriate function using if-elif logic
   - Returns the result or "Invalid function" error
   - Closes the client connection
4. Continues listening for the next client

Code walkthrough:

```
while True:
    conn, addr = server.accept()
    
    data = conn.recv(1024).decode().strip()
    parts = data.split()              # ["add", "5", "3"]
    
    func = parts[0]                   # "add"
    a = int(parts[1])                 # 5
    b = int(parts[2])                 # 3
    
    if func == "add":
        result = add(a, b)
    elif func == "sub":
        result = sub(a, b)
    else:
        result = "Invalid function"
    
    conn.sendall(str(result).encode())
    conn.close()
```

Key design decisions:
- The server handles one client at a time within the main loop
- Function dispatch uses conditional logic (a dictionary-based dispatch would scale better)
- Integer conversion assumes valid numeric input (no error handling for non-numeric arguments)
- The server runs indefinitely until interrupted with Ctrl+C

---

## RPC Call Flow Diagram

```
Client (Caller)                           Server (Callee)
  |                                           |
  |-- TCP Connect (localhost:818) ----------> |
  |                                           |
  |-- Send: "add 5 3" ---------------------> |
  |                                           |-- Parse: func="add", a=5, b=3
  |                                           |-- Execute: add(5, 3) -> 8
  |                                           |
  |<-- Receive: "8" ------------------------- |
  |                                           |
  |-- TCP Disconnect ---------------------->  |
  |                                           |-- Close client connection
  |                                           |-- Wait for next client
```

---

## The RPC Execution Model

### Marshaling (Serialization)

The client converts the function call into a transmittable format:

```
Local call:    add(5, 3)
Marshaled:     "add 5 3"  (space-delimited string)
Bytes on wire: b"add 5 3" (UTF-8 encoded)
```

### Unmarshaling (Deserialization)

The server reconstructs the function call from the received data:

```
Bytes received: b"add 5 3"
Decoded string:  "add 5 3"
Parsed:          func="add", args=[5, 3]
Executed:        add(5, 3) -> 8
```

### Result Return

The computed result follows the reverse path:

```
Server result:   8
Marshaled:       "8"
Bytes on wire:   b"8"
Client receives: "8"
```

---

## Comparison with Production RPC Systems

| Aspect              | This Simulation        | Production RPC (gRPC, Thrift)    |
|---------------------|------------------------|----------------------------------|
| Transport           | TCP (raw sockets)      | HTTP/2, TCP                     |
| Serialization       | Space-delimited text   | Protocol Buffers, Thrift, JSON  |
| Interface Definition| None (ad-hoc)          | .proto, .thrift IDL files       |
| Error Handling      | "Invalid function"     | Typed exceptions, status codes  |
| Type Safety         | Manual conversion      | Generated stubs, static types   |
| Service Discovery   | Hardcoded host:port    | DNS, etcd, Consul, Zookeeper    |
| Authentication      | None                   | TLS, OAuth, JWT                 |
| Streaming           | Not supported          | Bi-directional streaming        |

This simulation demonstrates the fundamental concept; production systems add layers of reliability, type safety, and scalability.

---

## Supported Operations

| Function | Syntax    | Example     | Result |
|----------|-----------|-------------|--------|
| add      | add a b   | add 5 3     | 8      |
| sub      | sub a b   | sub 10 7    | 3      |

Any other function name returns: "Invalid function"

---

## Error Handling

### Current Behavior

- Unrecognized functions: Returns "Invalid function" string
- Non-numeric arguments: Server crashes with ValueError (int conversion fails)
- Missing arguments: Server crashes with IndexError (parts list too short)
- Extra arguments: Extra values are silently ignored

### Recommended Improvements

```
try:
    func = parts[0]
    a = int(parts[1])
    b = int(parts[2])
    
    if func == "add":
        result = add(a, b)
    elif func == "sub":
        result = sub(a, b)
    else:
        result = "Error: Unknown function"
        
except (IndexError, ValueError):
    result = "Error: Usage: <add|sub> <int> <int>"
```

---

## Extending the Project

Possible enhancements:

- Add more operations (multiply, divide, power, modulo)
- Use a dictionary for function dispatch instead of if-elif chains
- Implement JSON serialization for structured request/response messages
- Add support for variable number of arguments
- Implement server-side logging of all RPC calls with timestamps
- Add authentication (require a secret token in each request)
- Build a client that can make multiple calls within a single connection
- Use Protocol Buffers or MessagePack for binary serialization
- Add timeout handling for unresponsive servers
- Implement a proper service definition file (JSON schema)



