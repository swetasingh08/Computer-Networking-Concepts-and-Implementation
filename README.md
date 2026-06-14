

# Computer-Networking-Concepts-and-Implementation

> Hands-on Python implementations of core computer networking concepts, protocols, and socket programming — bridging the gap between theoretical models and executable code.

![GitHub stars](https://img.shields.io/github/stars/swetasingh08/Computer-Networking-Concepts-and-Implementation?style=for-the-badge&logo=github) ![GitHub forks](https://img.shields.io/github/forks/swetasingh08/Computer-Networking-Concepts-and-Implementation?style=for-the-badge&logo=github) ![GitHub issues](https://img.shields.io/github/issues/swetasingh08/Computer-Networking-Concepts-and-Implementation?style=for-the-badge&logo=github) ![Last commit](https://img.shields.io/github/last-commit/swetasingh08/Computer-Networking-Concepts-and-Implementation?style=for-the-badge&logo=github) ![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge) ![Python](https://img.shields.io/badge/Python-3.7+-blue?style=for-the-badge&logo=python) ![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter)

---

## Table of Contents

- [Description](#-description)
- [Key Features](#-key-features)
- [Implemented Protocols and Concepts](#-implemented-protocols-and-concepts)
- [Use Cases](#-use-cases)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Detailed Module Documentation](#-detailed-module-documentation)
- [How It All Fits Together](#-how-it-all-fits-together)
- [Contributing](#-contributing)
- [License](#-license)
- [References](#-references)

---

## Description

This repository provides practical, executable implementations of fundamental computer networking concepts using Python and Jupyter Notebooks. Each module tackles a specific protocol or concept — from address resolution and flow control to subnetting and network diagnostics — with fully documented, runnable code.

The project is designed to bridge the gap between textbook networking theory and real socket-level programming. Every implementation uses TCP loopback sockets or standard system utilities, making them immediately accessible on any machine with Python installed — no special hardware, no external dependencies beyond the standard library.

What sets this collection apart is its pedagogical structure: each protocol is implemented as a pair of notebooks (client and server, or sender and receiver), allowing students and developers to observe both sides of the communication channel simultaneously. Console output with timestamps makes the protocol state transitions visible and debuggable.

---

## Key Features

- **Address Resolution Protocol Simulations** — Complete bidirectional ARP and RARP implementations demonstrating IP-to-MAC and MAC-to-IP resolution over TCP loopback sockets with pre-configured lookup tables.

- **Flow Control Protocol Implementations** — Executable models of Stop-and-Wait ARQ and Go-Back-N Sliding Window protocols with configurable packet loss simulation, timeout recovery, and cumulative acknowledgment handling.

- **TCP Socket Programming Fundamentals** — Progressive examples from basic "Hello World" client-server exchange to a persistent echo server that maintains state across multiple message exchanges.

- **Network Diagnostics Toolkit** — Python wrappers around system ping and traceroute utilities using the subprocess module, demonstrating programmatic invocation of network diagnostic commands.

- **Subnetting Calculator** — Pure Python implementation of IPv4 subnet calculations using bitwise operations — no external libraries, just the mathematics of network addressing.

- **Remote Procedure Call Demonstration** — A client-server system that marshals function calls (add, subtract) over TCP, illustrating the core RPC abstraction of making remote function invocations feel local.

- **Image Transfer Over TCP** — Binary data transmission with a length-prefix protocol, demonstrating how to handle TCP's stream nature for reliable file transfer.

- **HTTP Socket Example** — Raw HTTP request construction and response parsing at the socket level, revealing what happens beneath libraries like requests and urllib.

---

## Implemented Protocols and Concepts

| Module                      | Protocol/Concept           | Key Learning Outcomes                                        |
|-----------------------------|----------------------------|--------------------------------------------------------------|
| ARP Protocol                | Address Resolution (RFC 826)    | IP-to-MAC mapping, lookup tables, request-response pattern   |
| RARP Protocol               | Reverse ARP (RFC 903)           | MAC-to-IP mapping, diskless workstation bootstrapping        |
| Stop and Wait Protocol      | Stop-and-Wait ARQ               | Timeout handling, ACK loss recovery, alternating bit concept |
| Sliding Window Protocol     | Go-Back-N ARQ                   | Pipelining, cumulative ACKs, window-based flow control       |
| TCP Sockets via Echo        | Echo Protocol (RFC 862)         | Persistent connections, multi-message sessions, stateful communication |
| Socket Programming          | TCP Fundamentals                | Socket lifecycle, connection establishment, basic send/recv  |
| Ping and Traceroute         | ICMP Diagnostics                | Network reachability, path tracing, subprocess integration   |
| Subnetting                  | IPv4 Addressing (RFC 950, 1519) | CIDR notation, bitwise operations, subnet mask calculation   |
| Remote Procedure Call       | RPC Abstraction                 | Function marshaling, remote execution, client-server dispatch|
| Image Transfer              | Binary Data over TCP            | Length-prefix protocol, stream reassembly, BytesIO usage     |
| HTTP Socket                 | HTTP/1.0 (RFC 1945)             | Raw HTTP request formatting, response parsing at socket level|

---

## Use Cases

### For Students

- Run executable protocol simulations alongside textbook study to see abstract concepts (timeouts, cumulative ACKs, window sliding) in action with timestamped console output.
- Modify protocol parameters (window size, loss probability, timeout duration) and observe the effects on throughput and behavior.
- Use the Jupyter Notebook format to experiment interactively — change code cells and immediately see results without restarting servers.

### For Educators

- Demonstrate live protocol behavior in classroom settings by running sender and receiver notebooks side-by-side.
- Assign modifications to the implementations as programming exercises (e.g., "Add Selective Repeat support to the Sliding Window implementation").
- Use the subnetting calculator to verify manual subnet calculations during networking lectures.

### For Developers

- Reference implementations of common socket programming patterns: single-request, persistent connection, binary transfer, and RPC dispatch.
- Quick-start templates for building TCP client-server applications in Python.
- Examples of proper error handling patterns for network programming.

### For Self-Learners

- Progressive learning path from simple TCP hello to complex sliding window protocols.
- Each module builds on concepts introduced in earlier modules.
- Self-contained — every module works independently with clear console output showing protocol state.

---

## Quick Start

### Prerequisites

- Python 3.7 or higher
- Jupyter Notebook or JupyterLab (for .ipynb files)
- Pillow library (for Image Transfer module only): `pip install Pillow`
- All other modules use Python standard library exclusively

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/swetasingh08/Computer-Networking-Concepts-and-Implementation.git
cd Computer-Networking-Concepts-and-Implementation

# 2. Install Jupyter (if not already installed)
pip install jupyter

# 3. Launch Jupyter Notebook
jupyter notebook
```

### Running the Modules

Each module consists of two notebooks (client and server, or sender and receiver). For proper operation:

1. Open the server/receiver notebook and run all cells first — this starts the listening service
2. Open the client/sender notebook in a separate browser tab and run all cells
3. Observe the protocol interaction through console output in both notebooks

For the subnetting calculator and diagnostic tools, only a single notebook needs to be run.

---

## Project Structure

```
│
├── ARP Protocol
│   ├── client.ipynb              # ARP client: IP input → MAC lookup request
│   └── server.ipynb              # ARP server: maintains IP→MAC table
│
├── HTTP Socket
│   ├── client.ipynb              # HTTP client: raw socket HTTP request
│   └── server.ipynb              # HTTP server: basic HTTP response
│
├── Ping And Traceroute
│   ├── ping.ipynb                # Ping wrapper using subprocess
│   └── traceroute.ipynb          # Traceroute wrapper using subprocess
│
├── RARP Protocol
│   ├── client.ipynb              # RARP client: MAC input → IP lookup request
│   └── server.ipynb              # RARP server: maintains MAC→IP table
│
├── Remote Procedure Call
│   ├── client.ipynb              # RPC client: sends "add 5 3" style commands
│   └── server.ipynb              # RPC server: dispatches add/sub functions
│
├── Sliding Window Protocol
│   ├── receiver.ipynb            # GBN receiver: window size 1, no buffering
│   └── sender.ipynb              # GBN sender: window-based pipelining
│
├── Socket Programming
│   ├── client.ipynb              # Basic TCP client: "Hello Server" message
│   └── server.ipynb              # Basic TCP server: single connection handler
│
├── Stop and Wait Protocol
│   ├── receiver.ipynb            # Stop-and-Wait receiver with loss simulation
│   └── sender.ipynb              # Stop-and-Wait sender with timeout retransmission
│
├── Subnetting
│   └── subnetting.ipynb          # IPv4 subnet calculator (CIDR notation)
│
├── TCP Sockets via Echo
│   ├── client.ipynb              # Echo client: persistent connection, "bye" to quit
│   └── server.ipynb              # Echo server: reflects messages back to client
│
├── .gitignore
├── LICENSE                        # MIT License
└── README.md                      # Documentation
```
---

## Detailed Module Documentation

### 1. ARP Protocol (Address Resolution Protocol)

**Concept:** Maps logical IP addresses to physical MAC addresses, essential for local network communication where hosts must resolve Layer 3 addresses to Layer 2 hardware addresses before frames can be delivered.

**Implementation:**
- Server maintains a static ARP table: `{"165.165.80.80": "6A:08:AA:C2", "165.165.79.1": "8A:BC:E3:FA"}`
- Client prompts user for an IP address and queries the server
- Server performs lookup and returns the corresponding MAC address or "Not Found"
- Communication occurs over TCP on port 5000

**Key Learning Points:**
- Understanding the request-response pattern that underlies real ARP operation
- Distinguishing between Layer 2 (MAC) and Layer 3 (IP) addressing
- The role of lookup tables in address resolution

---

### 2. RARP Protocol (Reverse Address Resolution Protocol)

**Concept:** Performs the inverse mapping of ARP — resolves MAC addresses to IP addresses. Historically used by diskless workstations that knew only their hardware address at boot time.

**Implementation:**
- Server maintains a static RARP table: `{"6A:08:AA:C2": "165.165.80.80", "8A:BC:E3:FA": "165.165.79.1"}`
- Client prompts user for a MAC address and queries the server
- Server returns the corresponding IP address or "Not Found"
- Communication occurs over TCP on port 6000

**Key Learning Points:**
- Understanding why RARP was necessary before DHCP and BOOTP
- The relationship between ARP and RARP as bidirectional mapping protocols
- Limitations that led to RARP's obsolescence (static tables, no subnet mask/gateway info)

---

### 3. Stop and Wait Protocol

**Concept:** The simplest reliable data transfer protocol. Sender transmits one frame at a time and waits for an acknowledgment before sending the next. If the ACK doesn't arrive within the timeout period, the frame is retransmitted.

**Implementation:**
- Sender transmits 5 frames sequentially
- Socket timeout set to 2 seconds for retransmission triggers
- Receiver simulates frame loss (20% probability) and ACK loss (30% probability)
- Both sides log all state transitions with timestamps

**Key Learning Points:**
- Understanding the trade-off between simplicity and efficiency
- How timeout mechanisms provide reliability without negative acknowledgments
- Why Stop-and-Wait is inefficient for high-bandwidth, high-latency links
- The foundation for understanding more complex sliding window protocols

---

### 4. Sliding Window Protocol (Go-Back-N)

**Concept:** Implements pipelined data transfer where multiple frames can be in flight simultaneously, bounded by a window size. Uses cumulative acknowledgments and Go-Back-N retransmission strategy on timeout.

**Implementation:**
- Window size (W) = 4, total frames to transmit = 6
- Socket timeout = 3.0 seconds
- Simulated packet loss probability = 20%
- Sender tracks base (oldest unacknowledged frame) and next_frame pointers
- Receiver maintains window size of 1 with no out-of-order buffering
- On timeout, sender resets next_frame = base (full window rollback)

**Key Learning Points:**
- How pipelining improves link utilization over Stop-and-Wait
- The Go-Back-N retransmission strategy and its inefficiency under high loss
- Cumulative acknowledgment semantics (ACK N means all frames up to N received)
- Sequence number space requirements (M >= W + 1)
- The trade-off between Go-Back-N and Selective Repeat

---

### 5. Socket Programming (Basic TCP)

**Concept:** The simplest possible TCP client-server interaction — a "Hello World" of network programming demonstrating socket creation, connection establishment, data transmission, and teardown.

**Implementation:**
- Client connects to server on port 12345
- Client sends "Hello Server" message
- Server responds with "Hello from server"
- Both sides print received messages and close the connection

**Key Learning Points:**
- The TCP socket lifecycle: socket() → bind() → listen() → accept() → send()/recv() → close()
- String encoding/decoding for network transmission
- The accept() return value: a new socket for the client connection plus the client address
- The foundational pattern that every other protocol in this repository builds upon

---

### 6. TCP Sockets via Echo

**Concept:** A persistent, stateful TCP connection where the server echoes back every message the client sends. Demonstrates multi-message sessions over a single connection.

**Implementation:**
- Client and server maintain an open connection across multiple message exchanges
- Server echoes back the exact bytes received
- Session terminates when client sends "bye" (case-insensitive)
- Both sides independently check for the termination condition

**Key Learning Points:**
- Persistent connections vs. single-request connections
- Implementing application-layer session management over TCP
- Graceful connection termination signaling
- The echo protocol as a testing and diagnostic tool (RFC 862)

---

### 7. Ping and Traceroute

**Concept:** Python wrappers around system network diagnostic commands using the subprocess module. Demonstrates how to programmatically invoke and capture output from native OS utilities.

**Implementation:**
- Ping sends 4 ICMP Echo Requests using `ping -n 4 host` (Windows) or `ping -c 4 host` (Linux/Mac)
- Traceroute maps the network path using `tracert host` (Windows) or `traceroute host` (Linux/Mac)
- Both use subprocess.run() with capture_output=True and text=True
- Platform detection can be added using platform.system()

**Key Learning Points:**
- Using subprocess to interface with system utilities from Python
- Understanding ICMP diagnostics without implementing raw sockets
- Platform-aware command construction
- Capture and display of external command output

---

### 8. Subnetting Calculator

**Concept:** Pure Python implementation of IPv4 subnet calculations using bitwise operations. Computes subnet mask, network address, broadcast address, and usable host range from an IP address and CIDR prefix length.

**Implementation:**
- Subnet mask generation: 0xFFFFFFFF shifted left by (32 - prefix_length)
- Network address: IP AND subnet mask (bitwise)
- Broadcast address: network OR (NOT mask) (bitwise)
- Host range: network + 1 to broadcast - 1

**Key Learning Points:**
- IPv4 addresses as 32-bit integers and the mathematics of subnetting
- Bitwise operations: AND for network extraction, OR for broadcast, NOT for wildcard
- CIDR notation and prefix length semantics
- Reserved addresses: network address (all host bits 0) and broadcast (all host bits 1)

---

### 9. Remote Procedure Call (RPC)

**Concept:** Demonstrates the core RPC abstraction — making a function call on a remote server as if it were local. The client marshals a function name and arguments, transmits them, and the server executes and returns the result.

**Implementation:**
- Server exposes two arithmetic functions: add(a, b) and sub(a, b)
- Client sends space-delimited commands: "add 5 3" or "sub 10 7"
- Server parses the command, dispatches to the appropriate function, and returns the result
- Function dispatch uses if-elif conditional logic (dictionary dispatch recommended for extension)

**Key Learning Points:**
- Marshaling and unmarshaling function calls for network transmission
- Function dispatch patterns (conditional vs. dictionary-based)
- The RPC abstraction layer between caller and callee
- How production RPC frameworks (gRPC, Thrift) extend this basic concept

---

### 10. Image Transfer Over TCP

**Concept:** Demonstrates reliable binary file transfer over TCP using a length-prefix protocol. Addresses the fundamental challenge of TCP's stream nature: the receiver must know exactly how many bytes constitute the complete file.

**Implementation:**
- Client reads an image file in binary mode
- Client sends a 4-byte big-endian integer header containing the image size using struct.pack('!I', size)
- Client sends the raw image bytes using sendall()
- Server receives the 4-byte header, extracts the size with struct.unpack()
- Server loops recv() calls, accumulating data until the total matches the expected size
- Server wraps bytes in BytesIO and opens with PIL/Pillow for display

**Key Learning Points:**
- Why TCP's stream nature requires application-level message framing
- Length-prefix protocol design for binary data
- struct module for cross-platform binary encoding
- BytesIO for treating raw bytes as file-like objects

---

### 11. HTTP Socket

**Concept:** Raw HTTP request and response handling at the socket level, revealing the text-based protocol that underlies all web communication.

**Implementation:**
- Client constructs a properly formatted HTTP GET request string
- Client sends the request over a raw TCP socket to a web server
- Client receives and displays the HTTP response headers and body
- Server implements a minimal HTTP response for local testing

**Key Learning Points:**
- HTTP message format: request line, headers, blank line, body
- How libraries like requests and urllib abstract away socket-level details
- The text-based nature of HTTP/1.x

---

## How It All Fits Together

The modules in this repository form a progressive learning path through computer networking concepts:

### Layer-by-Layer Understanding

```
Application Layer:    HTTP Socket, RPC, Echo Server
                          ↓
Transport Layer:      TCP Socket Programming, Stop-and-Wait, Sliding Window
                          ↓
Network Layer:        Subnetting, Ping, Traceroute, ARP
                          ↓
Data Link Layer:      ARP, RARP (MAC addressing concepts)
```

### Conceptual Progression

1. **Start here:** Basic TCP Socket Programming — learn the socket lifecycle
2. **Add persistence:** Echo Server — maintain state across multiple messages
3. **Add reliability:** Stop-and-Wait — handle packet loss with timeouts
4. **Add efficiency:** Sliding Window — pipeline frames for better throughput
5. **Address resolution:** ARP and RARP — map between addressing schemes
6. **Network diagnostics:** Ping and Traceroute — test reachability and paths
7. **Address planning:** Subnetting — design and verify network segments
8. **Higher abstractions:** RPC — remote function execution
9. **Real protocols:** HTTP Socket — understand what browsers do
10. **Binary data:** Image Transfer — handle non-text payloads

### Running Modules Together

Several modules are designed as complementary pairs that can run simultaneously:

- **ARP + RARP:** Verify bidirectional address mapping consistency
- **Stop-and-Wait + Sliding Window:** Compare efficiency of different flow control strategies
- **Ping + Traceroute:** Diagnose network path issues end-to-end

---

## Contributing

Contributions are welcome and appreciated! Here's how to contribute:

### Standard Workflow

1. **Fork** the repository
2. **Clone** your fork:
   ```bash
   git clone https://github.com/your-username/Computer-Networking-Concepts-and-Implementation.git
   ```
3. **Create a branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes** following the existing code style
5. **Commit** with a descriptive message:
   ```bash
   git commit -m 'feat: add Selective Repeat protocol implementation'
   ```
6. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a pull request** with a clear description of your changes

### Contribution Ideas

- Implement the Selective Repeat protocol for comparison with Go-Back-N
- Add threading support to the echo server for concurrent client handling
- Create a DHCP simulation based on the ARP/RARP pattern
- Add IPv6 support to the subnetting calculator
- Implement a DNS resolver simulation
- Add performance metrics (throughput, latency) to the flow control protocols
- Create a unified GUI for all tools using Tkinter or PyQt
- Add comprehensive error handling and input validation to existing modules
- Convert Python scripts to Jupyter notebooks with explanatory markdown cells
- Add protocol state machine diagrams to notebook documentation

### Code Style Guidelines

- Use Python 3.7+ features where appropriate
- Follow PEP 8 conventions
- Include docstrings for functions
- Add comments explaining non-obvious logic
- Use meaningful variable names that match protocol terminology (e.g., `base`, `next_frame`, `expected` rather than `x`, `y`, `z`)

---

## License

This project is licensed under the **MIT** License. See the [LICENSE](LICENSE) file for full details.
