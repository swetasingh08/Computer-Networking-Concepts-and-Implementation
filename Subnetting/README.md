

# Subnet Calculator (IPv4)

A Python command-line tool that calculates IPv4 subnet details — subnet mask, network address, broadcast address, and usable host range — from a given IP address and prefix length (CIDR notation).

---

## Overview

This project implements the core mathematical operations behind IPv4 subnetting. Given an IP address and a prefix length (e.g., 192.168.1.1/24), the calculator computes:

- Subnet mask in dotted-decimal notation
- Network address (the base address of the subnet)
- Broadcast address (the last address in the subnet)
- Usable host range (all addresses between network and broadcast, exclusive)

The calculator uses only bitwise operations and integer arithmetic — the same fundamental logic that routers and hosts use to determine whether a destination IP is on the local network or requires forwarding through a gateway.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Calculator

```bash
# Clone the repository
git clone https://github.com/your-username/subnet-calculator.git
cd subnet-calculator

# Run the script
python subnet_calculator.py
```

### Sample Usage

```
Enter IP address (for example, 192.168.1.1): 192.168.1.100
Enter prefix length (for example, 24): 24

Subnet Mask: 255.255.255.0
Network Address: 192.168.1.0
Broadcast Address: 192.168.1.255
Host Range: 192.168.1.1 - 192.168.1.254
```

---

## How It Works

### The Mathematics of Subnetting

An IPv4 address is a 32-bit number, conventionally written as four 8-bit octets separated by dots (dotted-decimal notation). Subnetting divides this 32-bit space into two parts:

- Network portion: The leftmost bits, identified by the prefix length
- Host portion: The remaining rightmost bits

For example, with prefix length /24:
- Network portion: first 24 bits (3 octets)
- Host portion: last 8 bits (1 octet)

```
192.168.1.100 /24

IP:    11000000.10101000.00000001.01100100
Mask:  11111111.11111111.11111111.00000000
       |________ Network (24 bits) ________| |__ Host (8 bits) __|
```

### Functions Explained

#### 1. calculate_subnet_mask(prefix_length)

Converts a prefix length to a dotted-decimal subnet mask.

Algorithm:

```
1. Start with 32 ones:  0xFFFFFFFF
2. Shift left by (32 - prefix_length) positions to zero out host bits
3. Mask with 0xFFFFFFFF to keep within 32 bits
4. Split into 4 octets and convert to dotted-decimal
```

Example for /24:

```
0xFFFFFFFF << (32 - 24) = 0xFFFFFFFF << 8 = 0xFFFFFF00
Split: 0xFF, 0xFF, 0xFF, 0x00 = 255.255.255.0
```

Example for /26:

```
0xFFFFFFFF << (32 - 26) = 0xFFFFFFFF << 6 = 0xFFFFFFC0
Split: 0xFF, 0xFF, 0xFF, 0xC0 = 255.255.255.192
```

#### 2. calculate_network_address(ip_address, subnet_mask)

Finds the network address by performing a bitwise AND between the IP address and the subnet mask.

Algorithm:

```
For each octet (0-3):
    network_octet = ip_octet AND mask_octet
```

Example:

```
IP:   192.168.1.100  = 11000000.10101000.00000001.01100100
Mask: 255.255.255.0   = 11111111.11111111.11111111.00000000
AND:  192.168.1.0     = 11000000.10101000.00000001.00000000
```

The network address always has all host bits set to 0.

#### 3. calculate_broadcast_address(network_address, subnet_mask)

Finds the broadcast address by taking the network address and setting all host bits to 1. This is done by OR-ing the network address with the bitwise NOT of the subnet mask.

Algorithm:

```
For each octet (0-3):
    broadcast_octet = network_octet OR (NOT mask_octet AND 0xFF)
```

The AND with 0xFF ensures the inverted mask octet stays within 8 bits (two's complement representation of negative numbers would otherwise produce unexpected values).

Example:

```
Network: 192.168.1.0   = 11000000.10101000.00000001.00000000
~Mask:   0.0.0.255      = 00000000.00000000.00000000.11111111
OR:      192.168.1.255  = 11000000.10101000.00000001.11111111
```

The broadcast address always has all host bits set to 1.

#### 4. calculate_host_range(network_address, broadcast_address)

Returns the first and last usable host addresses in the subnet.

Algorithm:

```
First usable host = network_address + 1 (increment last octet)
Last usable host  = broadcast_address - 1 (decrement last octet)
```

Example:

```
Network:   192.168.1.0
Broadcast: 192.168.1.255
Host Range: 192.168.1.1 - 192.168.1.254
```

The network address and broadcast address are reserved and cannot be assigned to hosts.

---

## Key Subnetting Concepts

### Why Subnetting Matters

Subnetting solves several critical networking problems:

1. Address Conservation: Instead of assigning an entire Class A, B, or C network to an organization, subnetting allows allocation of appropriately sized address blocks.

2. Network Segmentation: Large networks are divided into smaller broadcast domains, reducing broadcast traffic and improving performance.

3. Security Isolation: Different subnets can have different security policies applied at routers and firewalls.

4. Route Aggregation: Hierarchical subnet allocation enables route summarization, reducing the size of routing tables.

### CIDR Notation

Classless Inter-Domain Routing (CIDR) notation expresses the IP address and prefix length together:

```
192.168.1.100/24
```

The /24 indicates that the first 24 bits are the network prefix. The remaining 8 bits (32 - 24) are available for host addressing.

### Number of Hosts Per Subnet

For a prefix length of /n:

```
Number of addresses = 2^(32 - n)
Usable host addresses = 2^(32 - n) - 2
```

The subtraction of 2 accounts for the network address and broadcast address.

| Prefix | Subnet Mask        | Total Addresses | Usable Hosts | Typical Use        |
|--------|--------------------|-----------------|--------------|--------------------|
| /8     | 255.0.0.0          | 16,777,216      | 16,777,214   | Class A (large)    |
| /16    | 255.255.0.0        | 65,536          | 65,534       | Class B (medium)   |
| /24    | 255.255.255.0      | 256             | 254          | Class C (small)    |
| /25    | 255.255.255.128    | 128             | 126          | Small subnet       |
| /26    | 255.255.255.192    | 64              | 62           | Small subnet       |
| /28    | 255.255.255.240    | 16              | 14           | Point-to-point     |
| /30    | 255.255.255.252    | 4               | 2            | Point-to-point link|
| /32    | 255.255.255.255    | 1               | 0 (host only)| Single host route  |

---

## Bitwise Operations Reference

The calculator relies on four fundamental bitwise operations:

### AND (&)

```
1 & 1 = 1
1 & 0 = 0
0 & 1 = 0
0 & 0 = 0
```

Used to extract the network portion: IP & Mask = Network

### OR (|)

```
1 | 1 = 1
1 | 0 = 1
0 | 1 = 1
0 | 0 = 0
```

Used to set host bits to 1 for the broadcast address.

### NOT (~)

```
~1 = -2 (in Python, due to two's complement)
~0 = -1
```

Python's bitwise NOT returns the two's complement. The AND with 0xFF (~mask_parts[i] & 0xFF) constrains the result to 8 bits.

### Left Shift (<<)

```
0xFF << 8 = 0xFF00
```

Moves bits to the left by the specified number of positions. Used to generate the subnet mask by shifting 32 ones.

---

## Practical Examples

### Example 1: Home Network

```
IP:     192.168.1.100
Prefix: /24

Subnet Mask:    255.255.255.0
Network:        192.168.1.0
Broadcast:      192.168.1.255
Host Range:     192.168.1.1 - 192.168.1.254
Usable Hosts:   254
```

### Example 2: Office Subnet

```
IP:     10.0.5.130
Prefix: /26

Subnet Mask:    255.255.255.192
Network:        10.0.5.128
Broadcast:      10.0.5.191
Host Range:     10.0.5.129 - 10.0.5.190
Usable Hosts:   62
```

### Example 3: Point-to-Point Link

```
IP:     172.16.0.1
Prefix: /30

Subnet Mask:    255.255.255.252
Network:        172.16.0.0
Broadcast:      172.16.0.3
Host Range:     172.16.0.1 - 172.16.0.2
Usable Hosts:   2
```

---

## Extending the Project

Possible enhancements:

- Add support for subnetting a network into multiple smaller subnets (VLSM)
- Calculate and display the wildcard mask (used in ACLs and OSPF)
- Add binary display showing the bit-level operations
- Support for IPv6 subnetting
- Generate a visual subnet map showing address allocation
- Accept input in CIDR notation (e.g., 192.168.1.100/24 in a single input)
- Calculate supernet information (aggregate multiple subnets)
- Export results to JSON or CSV format
- Build a web interface using Flask for browser-based access



