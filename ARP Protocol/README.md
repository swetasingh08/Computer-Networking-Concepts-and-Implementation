
# ARP (Address Resolution Protocol) Simulation

A Python socket-based client-server simulation demonstrating the core functionality of the Address Resolution Protocol (ARP), which maps logical IP addresses to physical MAC addresses.

---

## Overview

This project simulates a simplified ARP lookup service using TCP loopback sockets. It consists of two components:

- A server that maintains an ARP table (IP-to-MAC mappings) and responds to lookup requests
- A client that accepts user input (an IP address) and queries the server for the corresponding MAC address

This implementation demonstrates the fundamental request-response pattern that underlies real ARP operation at the network layer, where hosts must resolve IP addresses to hardware addresses before frames can be delivered on a local network segment.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/arp-simulation.git
cd arp-simulation

# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

Enter an IP address when prompted. The client will query the server and display the resolved MAC address (or "Not Found" if the IP is not in the table).

---

## How It Works

### ARP Table (Pre-configured Mappings)

The server maintains a static ARP table with the following entries:

| IP Address      | MAC Address   |
|-----------------|---------------|
| 165.165.80.80   | 6A:08:AA:C2   |
| 165.165.79.1    | 8A:BC:E3:FA   |

Any IP address not in this table returns "Not Found."

### Protocol Flow

```
Client                              Server
  |                                    |
  |-- TCP Connect (127.0.0.1:5000) --> |
  |                                    |
  |-- Send IP: "165.165.80.80" ------> |
  |                                    |-- Lookup in ARP table
  |                                    |-- Found: "6A:08:AA:C2"
  |                                    |
  |<-- Receive MAC: "6A:08:AA:C2" ---- |
  |                                    |
  |-- TCP Disconnect ----------------> |
```

### Client (client.py)

The client performs the following steps:

1. Creates a TCP socket and connects to the server at 127.0.0.1:5000
2. Prompts the user to enter a logical address (IP)
3. Sends the IP address to the server (newline-terminated string)
4. Receives the resolved physical address (MAC) from the server
5. Displays the result and closes the connection

Error handling:
- If the server is not running, the client catches the ConnectionRefusedError and displays a user-friendly error message
- The socket is always closed in the finally block to prevent resource leaks

### Server (server.py)

The server performs the following steps:

1. Creates a TCP socket and binds it to 127.0.0.1:5000
2. Sets SO_REUSEADDR to allow immediate port reuse after stopping
3. Listens for incoming client connections
4. Upon accepting a client:
   - Receives the requested IP address
   - Looks up the IP in the pre-configured ARP table
   - Returns the corresponding MAC address or "Not Found"
   - Closes the client connection
5. Continues listening for additional requests until interrupted (Ctrl+C)

Design notes:
- The server handles one client at a time (no threading)
- Each client connection is closed after a single request-response exchange
- Graceful shutdown is handled via KeyboardInterrupt

---

## Why This Matters

In real network stacks, ARP is essential for local network communication:

- When a host wants to send an IP packet to another host on the same subnet, it must first determine the destination's MAC address
- The host broadcasts an ARP request: "Who has IP 192.168.1.5? Tell 192.168.1.10"
- The target host responds with a unicast ARP reply containing its MAC address
- The resolved mapping is cached in the host's ARP table to avoid repeated lookups

This simulation abstracts away the broadcast mechanism and focuses on the core lookup logic, using TCP for reliable client-server communication instead of raw Ethernet frames.

---

## Extending the Simulation

Possible enhancements:

- Add support for dynamic ARP table updates (client can register new IP-MAC mappings)
- Implement a proper ARP cache with TTL (Time To Live) expiration
- Support multiple simultaneous clients using threading
- Replace TCP with UDP and implement a broadcast-style discovery mechanism
- Add logging of all ARP requests and responses to a file

---
