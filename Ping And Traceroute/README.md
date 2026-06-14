
# Network Diagnostic Tools (Ping & Traceroute)

A Python-based wrapper around system ping and traceroute utilities, providing a simple programmatic interface for network reachability testing and path tracing using the subprocess module.

---

## Overview

This project demonstrates how to leverage Python's subprocess module to interact with native operating system networking utilities. Instead of reimplementing ICMP protocols from scratch, these scripts act as clean wrappers around the system's built-in ping and tracert/traceroute commands, capturing and displaying their output within a Python environment.

This approach is valuable for network automation, monitoring scripts, and educational purposes where understanding how to interface with system commands is essential.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)
- Windows, Linux, or macOS

### Running the Scripts

```bash
# Clone the repository
git clone https://github.com/your-username/network-diagnostics.git
cd network-diagnostics

# Test network reachability
python ping_test.py

# Trace the route to a destination
python traceroute_test.py
```

Edit the hostname in each script to test different destinations.

---

## How It Works

### Architecture

Both scripts follow the same pattern:

1. Accept a target host (domain name or IP address)
2. Construct the appropriate system command as a list of arguments
3. Execute the command via subprocess.run()
4. Capture stdout and stderr
5. Display the results to the console

This design separates the Python orchestration layer from the actual network operations performed by the operating system.

---

## Ping Utility (ping_test.py)

### Purpose

Ping tests network reachability by sending ICMP Echo Request packets to a target host and waiting for ICMP Echo Reply packets. It measures:

- Round-trip time (RTT) in milliseconds
- Packet loss percentage
- Overall reachability of the target

### Code Breakdown

```
import subprocess

def ping_host(host):
    print(f"Pinging {host}...\n")

    try:
        # Windows uses -n, Linux/Mac uses -c
        command = ["ping", "-n", "4", host]
        
        result = subprocess.run(
            command,
            capture_output=True,
            text=True
        )
        
        print(result.stdout)
    
    except Exception as e:
        print("Error:", e)

ping_host("www.google.com")
```

Key details:

- The command is constructed as a list: ["ping", "-n", "4", host]
  - "ping" — the system executable
  - "-n" — flag for number of pings (Windows); use "-c" for Linux/macOS
  - "4" — send exactly 4 ICMP echo requests
  - host — the target destination
- subprocess.run() executes the command and waits for completion
- capture_output=True redirects stdout and stderr into the result object
- text=True returns output as a string instead of bytes
- The try-except block handles cases where ping is not available or the hostname is invalid

### Platform Compatibility

| OS      | Count Flag | Command Example              |
|---------|------------|------------------------------|
| Windows | -n         | ping -n 4 www.google.com     |
| Linux   | -c         | ping -c 4 www.google.com     |
| macOS   | -c         | ping -c 4 www.google.com     |

To make the script cross-platform automatically, add:

```
import platform
count_flag = "-n" if platform.system() == "Windows" else "-c"
command = ["ping", count_flag, "4", host]
```

### Sample Output

```
Pinging www.google.com...

Pinging www.google.com [142.250.195.68] with 32 bytes of data:
Reply from 142.250.195.68: bytes=32 time=12ms TTL=118
Reply from 142.250.195.68: bytes=32 time=11ms TTL=118
Reply from 142.250.195.68: bytes=32 time=13ms TTL=118
Reply from 142.250.195.68: bytes=32 time=10ms TTL=118

Ping statistics for 142.250.195.68:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 13ms, Average = 11ms
```

---

## Traceroute Utility (traceroute_test.py)

### Purpose

Traceroute maps the network path from the source host to a destination by sending packets with incrementally increasing Time-To-Live (TTL) values. Each router along the path decrements the TTL, and when it reaches zero, the router sends back an ICMP Time Exceeded message, revealing its identity. This allows traceroute to:

- Identify each hop (router) between source and destination
- Measure latency to each intermediate hop
- Detect routing loops or black holes
- Diagnose where packet loss occurs along a path

### Code Breakdown

```
import subprocess

def traceroute_host(host):
    print(f"Tracing route to {host}...\n")

    try:
        # Windows uses tracert, Linux/macOS uses traceroute
        command = ["tracert", host]
        
        result = subprocess.run(
            command,
            capture_output=True,
            text=True
        )
        
        print(result.stdout)
    
    except Exception as e:
        print("Error:", e)

traceroute_host("www.google.com")
```

Key details:

- The command is constructed as: ["tracert", host] for Windows
- For Linux/macOS, change to: ["traceroute", host]
- subprocess.run() with the same capture pattern as the ping script
- No count parameter needed; traceroute determines the number of hops dynamically (up to 30 by default)

### Platform Compatibility

| OS      | Command    | Example                        |
|---------|------------|--------------------------------|
| Windows | tracert    | tracert www.google.com         |
| Linux   | traceroute | traceroute www.google.com      |
| macOS   | traceroute | traceroute www.google.com      |

Cross-platform version:

```
import platform
cmd = "tracert" if platform.system() == "Windows" else "traceroute"
command = [cmd, host]
```

### Sample Output

```
Tracing route to www.google.com...

Tracing route to www.google.com [142.250.195.68]
over a maximum of 30 hops:

  1     1ms     1ms     1ms  192.168.1.1
  2    10ms     9ms    11ms  10.20.30.1
  3    12ms    11ms    13ms  142.250.195.68

Trace complete.
```

Note: Asterisks (*) indicate a hop that did not respond within the timeout period (common for routers that deprioritize ICMP responses).

---

## Why Use subprocess Instead of Raw Sockets?

### Advantages of the subprocess Approach

- No root/administrator privileges required (raw sockets typically need elevated permissions)
- Leverages battle-tested system implementations
- Works reliably across different network configurations
- Avoids complex ICMP packet crafting and parsing
- Handles all edge cases already managed by the OS

### Limitations

- Platform-dependent command syntax
- Output parsing requires text processing
- Less control over packet parameters (TTL, payload size, etc.)
- Dependent on system utilities being installed

---

## Practical Applications

### Network Monitoring

Integrate these scripts into automated monitoring systems:

- Periodic ping checks to verify server availability
- Alerting when packet loss exceeds a threshold
- Logging baseline latency for trend analysis

### Troubleshooting

Use these tools to diagnose:

- "Is the target reachable?" (ping)
- "Where is the connection failing?" (traceroute)
- "Is there unusual latency?" (ping RTT)
- "Is there a routing loop?" (traceroute with repeated hops)

### Educational

These scripts demonstrate:

- How to use subprocess to interact with system commands
- How network diagnostics work at a fundamental level
- How Python can orchestrate system-level operations

---

## Error Handling

The try-except block catches:

- FileNotFoundError — if the ping/traceroute command is not available on the system
- subprocess.TimeoutExpired — if the command hangs (add timeout parameter to subprocess.run())
- PermissionError — if the script lacks execute permissions
- Generic exceptions for unexpected failures

For more robust error handling, consider:

```
try:
    result = subprocess.run(command, capture_output=True, text=True, timeout=30)
    if result.returncode != 0:
        print(f"Command failed with return code {result.returncode}")
        print(result.stderr)
    else:
        print(result.stdout)
except subprocess.TimeoutExpired:
    print("Command timed out after 30 seconds")
except FileNotFoundError:
    print("Command not found. Is the network utility installed?")
```

---

## Extending the Project

Possible enhancements:

- Parse ping output to extract and log only the latency values
- Run traceroute and automatically identify the hop with highest latency
- Build a simple GUI with Tkinter for entering hosts and viewing results
- Schedule periodic tests and log results to a file with timestamps
- Add DNS lookup (nslookup/dig) as a third diagnostic tool
- Create a unified "network health check" that runs all diagnostics
- Add email/Slack alerting when a host becomes unreachable

---



## Security Considerations

- These scripts execute system commands with user-provided input (the host parameter). In a production environment, validate and sanitize the host parameter to prevent command injection attacks.
- Never pass unsanitized user input directly to subprocess with shell=True. The current implementation uses a list of arguments (shell=False by default), which is the safe approach.
- Some networks block ICMP traffic, which will cause ping and traceroute to fail even if the host is reachable via TCP/UDP.

---
