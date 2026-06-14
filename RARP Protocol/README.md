

---

# RARP (Reverse Address Resolution Protocol) Simulation

A Python socket-based client-server simulation demonstrating the Reverse Address Resolution Protocol (RARP), which maps physical MAC addresses to logical IP addresses — the inverse operation of ARP.

---

## Overview

This project simulates a simplified RARP lookup service using TCP loopback sockets. It consists of two components:

- A server that maintains a RARP table (MAC-to-IP mappings) and responds to reverse lookup requests
- A client that accepts user input (a MAC address) and queries the server for the corresponding IP address

In real network scenarios, RARP was historically used by diskless workstations that knew only their hardware (MAC) address and needed to discover their IP address from a RARP server on the local network. While RARP has largely been superseded by protocols like DHCP and BOOTP, understanding its operation provides valuable insight into the evolution of network address configuration.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/rarp-simulation.git
cd rarp-simulation

# Start the server in one terminal
python server.py

# In another terminal, run the client
python client.py
```

Enter a MAC address when prompted (e.g., 6A:08:AA:C2). The client will query the server and display the resolved IP address (or "Not Found" if the MAC is not in the table).

---

## How It Works

### RARP Table (Pre-configured Mappings)

The server maintains a static RARP table with the following entries:

| MAC Address   | IP Address    |
|---------------|---------------|
| 6A:08:AA:C2   | 165.165.80.80 |
| 8A:BC:E3:FA   | 165.165.79.1  |

Any MAC address not in this table returns "Not Found."

This is the inverse mapping of the ARP table used in the companion ARP simulation. Together, they demonstrate the bidirectional relationship between the two protocols.

---

### Protocol Flow

```
Client (Diskless Host)                Server (RARP Server)
  |                                         |
  |-- TCP Connect (127.0.0.1:6000) ------> |
  |                                         |
  |-- Send MAC: "6A:08:AA:C2" -----------> |
  |                                         |-- Lookup in RARP table
  |                                         |-- Found: "165.165.80.80"
  |                                         |
  |<-- Receive IP: "165.165.80.80" -------- |
  |                                         |
  |-- TCP Disconnect -------------------->  |
```

### Client (client.py)

The client performs the following steps:

1. Creates a TCP socket and connects to the server at 127.0.0.1:6000
2. Prompts the user to enter a physical address (MAC) in the format XX:XX:XX:XX
3. Sends the MAC address to the server (newline-terminated string)
4. Receives the resolved logical address (IP) from the server
5. Displays the result and closes the connection

Key design note: The client sends a newline-terminated string ("\n") to mark the end of the message, allowing the server to distinguish message boundaries in the TCP stream.

### Server (server.py)

The server performs the following steps:

1. Creates a TCP socket and binds it to 127.0.0.1:6000
2. Listens for an incoming client connection
3. Accepts a single client connection
4. Enters a loop to continuously receive MAC address queries:
   - Receives data until a newline character
   - Strips whitespace from the received MAC address
   - Looks up the MAC in the pre-configured RARP table
   - Returns the corresponding IP address or "Not Found"
   - Breaks the loop if the client closes the connection (empty data)
5. Closes both the client connection and the server socket

Key design notes:
- The server currently accepts only one client connection and processes multiple queries from that client
- The loop breaks when recv() returns an empty byte string, indicating the client has closed its end of the connection
- The server uses port 6000 to avoid conflicts with the ARP simulation (which uses port 5000)

---

## RARP vs ARP: Understanding the Relationship

### ARP (Address Resolution Protocol)

- Direction: IP address to MAC address
- Use case: A host knows the destination IP and needs the corresponding MAC to construct an Ethernet frame
- Query: "Who has IP 165.165.80.80? Tell me your MAC"
- Response: "I have IP 165.165.80.80. My MAC is 6A:08:AA:C2"

### RARP (Reverse Address Resolution Protocol)

- Direction: MAC address to IP address
- Use case: A diskless host knows only its burned-in hardware address and needs an IP address to participate in the network
- Query: "My MAC is 6A:08:AA:C2. What is my IP?"
- Response: "Your IP address is 165.165.80.80"

### Why RARP is Now Obsolete

RARP had several significant limitations that led to its replacement:

1. RARP operates at the data link layer (Layer 2) and cannot cross routers — a RARP server must be on every subnet
2. RARP only provides an IP address; it cannot provide subnet mask, default gateway, DNS servers, or other configuration
3. RARP uses a static table on the server, requiring manual maintenance
4. Modern protocols like DHCP provide complete network configuration with dynamic address allocation

BOOTP improved upon RARP by using UDP (which can be forwarded across routers) and providing additional configuration. DHCP extended BOOTP with dynamic address pools and lease management.

---

## Comparison with the ARP Simulation

| Aspect          | ARP Simulation                     | RARP Simulation                    |
|-----------------|------------------------------------|------------------------------------|
| Port            | 5000                               | 6000                               |
| Input           | IP address                         | MAC address                        |
| Output          | MAC address                        | IP address                         |
| Table Mapping   | IP to MAC                          | MAC to IP                          |
| Query Format    | "Who has this IP?"                 | "What IP belongs to this MAC?"     |
| Real Protocol   | ARP (RFC 826)                      | RARP (RFC 903)                     |
| Current Status  | Still widely used                  | Obsolete, replaced by DHCP/BOOTP   |

---

## Error Handling and Limitations

The current implementation handles:

- Unknown MAC addresses (returns "Not Found")
- Client disconnection (empty data breaks the receive loop)

Limitations to be aware of:

- The server handles only one client at a time and shuts down after that client disconnects
- No input validation is performed on the MAC address format
- The RARP table is hardcoded and cannot be modified at runtime
- No timeout handling for stalled connections

For production use, consider adding:

```
# Input validation example
import re
mac_pattern = re.compile(r'^([0-9A-Fa-f]{2}[:]){3}([0-9A-Fa-f]{2})$')
if not mac_pattern.match(mac):
    conn.send(("Invalid MAC format\n").encode())
    continue
```

---

## Extending the Simulation

Possible enhancements:

- Support for multiple concurrent clients using threading or asyncio
- Dynamic RARP table with the ability to register new MAC-IP mappings
- Integration with the ARP simulation for a complete address resolution suite
- Add BOOTP/DHCP-style configuration delivery (subnet mask, gateway, DNS)
- Implement proper RARP packet format using raw sockets (advanced)
- Add logging of all reverse lookups with timestamps
- Build a simple GUI for querying and managing the RARP table
- Make the server persistent (handle multiple client connections in sequence)

---

## Running Both ARP and RARP Together

To demonstrate the full address resolution cycle:

```bash
# Terminal 1: ARP Server
python arp_server.py      # Listens on port 5000

# Terminal 2: RARP Server
python rarp_server.py     # Listens on port 6000

# Terminal 3: ARP Client
python arp_client.py      # IP to MAC lookup
# Enter: 165.165.80.80
# Output: 6A:08:AA:C2

# Terminal 4: RARP Client
python rarp_client.py     # MAC to IP lookup
# Enter: 6A:08:AA:C2
# Output: 165.165.80.80
```

This demonstrates that both protocols correctly map the same pair of addresses in opposite directions, verifying table consistency.


