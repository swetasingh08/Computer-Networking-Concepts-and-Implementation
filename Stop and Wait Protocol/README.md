
# Stop-and-Wait ARQ Protocol Simulation

A Python socket-based simulation of the Stop-and-Wait Automatic Repeat reQuest (ARQ) protocol, demonstrating reliable data delivery over an unreliable channel with simulated frame loss and acknowledgment loss.

---

## Overview

This project implements the simplest reliable data transfer protocol: Stop-and-Wait. The sender transmits one frame at a time and waits for an acknowledgment (ACK) before sending the next frame. If the ACK does not arrive within a timeout period, the sender retransmits the same frame.

The simulation introduces artificial packet loss on both data frames (20% probability) and acknowledgments (30% probability) to demonstrate how the protocol handles unreliable channels. This makes the robustness mechanisms visible and testable.

---

## Quick Start

### Prerequisites

- Python 3.7+
- No external dependencies (standard library only)

### Running the Simulation

```bash
# Clone the repository
git clone https://github.com/your-username/stop-and-wait.git
cd stop-and-wait

# Start the receiver in one terminal
python receiver.py

# In another terminal, run the sender
python sender.py
```

The sender will transmit 5 frames through the simulated lossy channel. Both endpoints log their state transitions with timestamps to the console.

---

## How It Works

### The Stop-and-Wait Principle

In Stop-and-Wait, the protocol operates on a strict one-at-a-time basis:

1. Sender transmits Frame N
2. Sender starts a timer and waits for ACK N
3. Receiver receives Frame N
4. Receiver sends ACK N
5. Sender receives ACK N, cancels timer
6. Sender increments N and repeats from step 1

Only one frame is "in flight" at any given moment. This simplicity comes at the cost of efficiency — the sender is idle while waiting for each acknowledgment.

---

### Protocol Parameters

| Parameter        | Symbol | Value  | Description                                    |
|------------------|--------|--------|------------------------------------------------|
| Total Frames     | N      | 5      | Number of frames to transmit                   |
| Timeout          | T      | 2.0s   | Time before sender retransmits                 |
| Frame Loss Rate  | P_f    | 0.2    | Probability receiver drops an incoming frame   |
| ACK Loss Rate    | P_a    | 0.3    | Probability receiver's ACK is lost            |
| Host             | -      | 127.0.0.1 | Loopback address for local testing         |
| Port             | -      | 6000   | TCP port for communication                     |

---

## Architecture

### Sender (sender.py)

The sender maintains a single state variable:

- frame: The current sequence number being transmitted (1 through 5)

Core Loop Logic:

```
1. While frame <= TOTAL_FRAMES:
   a. Send current frame to receiver
   b. Set socket timeout to TIMEOUT seconds
   c. Wait for ACK
   d. If ACK received and ACK number matches frame:
         Increment frame (move to next)
   e. If wrong ACK received:
         Retransmit same frame (stay on current frame)
   f. If timeout occurs:
         Retransmit same frame (stay on current frame)
   g. Sleep 1 second between iterations
2. Close connection
```

Key design decisions:

- sock.settimeout(TIMEOUT) configures the socket to raise socket.timeout after 2 seconds of no response
- The sender compares the received ACK number to the current frame number — only a matching ACK allows progress
- A wrong ACK (duplicate ACK from a previous frame) triggers retransmission, handling the case where a delayed ACK arrives after timeout
- The 1-second sleep between iterations throttles output for readability

### Receiver (receiver.py)

The receiver maintains a single state variable:

- expected: The next in-order frame number expected (starts at 1)

Core Loop Logic:

```
1. Receive frame from sender
2. Simulate frame loss:
   If random() < 0.2: discard frame, do not send ACK
3. If frame == expected:
   Set ACK = frame
   Increment expected
4. If frame != expected:
   Set ACK = expected - 1 (duplicate ACK for last good frame)
5. Simulate ACK loss:
   If random() < 0.3: discard ACK, do not send
6. Send ACK to sender
```

Key design decisions:

- Frame loss is simulated before sequence validation — a lost frame generates no ACK at all
- Out-of-order frames (which shouldn't occur in Stop-and-Wait) trigger a duplicate ACK, which tells the sender the last in-order frame was received
- ACK loss is simulated after ACK calculation but before transmission
- The receiver tracks the expected frame number to generate correct cumulative ACKs

---

## Protocol Scenarios

### Scenario 1: Normal Operation (No Loss)

```
Sender                                Receiver
  |                                      |
  |-- Frame 1 ------------------------> |-- Received, ACK 1 sent
  |<-- ACK 1 -------------------------- |
  |                                      |
  |-- Frame 2 ------------------------> |-- Received, ACK 2 sent
  |<-- ACK 2 -------------------------- |
  |                                      |
  |-- Frame 3 ------------------------> |-- Received, ACK 3 sent
  |<-- ACK 3 -------------------------- |
```

Time between frames: approximately 1 second (controlled by sleep)

### Scenario 2: Frame Loss

```
Sender                                Receiver
  |                                      |
  |-- Frame 2 ------------------------> |-- [LOST - random < 0.2]
  |   [Timer starts: 2 seconds]          |   (no ACK sent)
  |   [Timer expires: timeout]           |
  |-- Frame 2 (retransmit) -----------> |-- Received, ACK 2 sent
  |<-- ACK 2 -------------------------- |
```

The sender's timeout mechanism recovers from the lost frame without any special signaling from the receiver.

### Scenario 3: ACK Loss

```
Sender                                Receiver
  |                                      |
  |-- Frame 3 ------------------------> |-- Received
  |   [Timer starts]                     |-- ACK 3 generated
  |                                      |-- [ACK LOST - random < 0.3]
  |   [Timer expires: timeout]           |
  |-- Frame 3 (retransmit) -----------> |-- Duplicate received!
  |                                      |-- ACK 3 sent again
  |<-- ACK 3 -------------------------- |
```

The receiver correctly handles the duplicate frame (sends ACK 3 again), and the sender correctly ignores it as it already moved on. If the sender had already moved to frame 4, the duplicate ACK 3 would be a wrong ACK and would be ignored.

---

## State Machine Diagrams

### Sender State Machine

```
                    +-----------+
                    |   IDLE    |
                    +-----------+
                          |
                          | Send Frame N, Start Timer
                          v
                    +-----------+
            +------>| WAITING   |<------+
            |       +-----------+       |
            |         |         |       |
            |   ACK=N |         | Wrong |
            |   arrives         | ACK   |
            |         |         |       |
            |         v         v       |
            |    +--------+  +--------+ |
            |    | NEXT   |  |RETRANS-| |
            |    | FRAME  |  | MIT    |-+
            |    +--------+  +--------+
            |         |
            |         v
            |    +-----------+     +-----------+
            +----| COMPLETE  |     |  TIMEOUT  |
                 +-----------+     +-----------+
                                        |
                                        +--- Retransmit Frame N
```

### Receiver State Machine

```
                    +-----------+
                    |  WAITING  |
                    +-----------+
                          |
                          | Frame arrives
                          v
                    +-----------+
                    | LOSS TEST |--[lost]--> Drop frame, return to WAITING
                    +-----------+
                          |
                          | [not lost]
                          v
                    +-----------+
                    | SEQ CHECK |
                    +-----------+
                     |         |
              [match]|         |[mismatch]
                     v         v
              +--------+   +--------+
              | ACCEPT |   |DISCARD |
              | ACK=N  |   |ACK=N-1 |
              +--------+   +--------+
                     |         |
                     v         v
                  +-------------+
                  | ACK LOSS    |
                  | TEST        |
                  +-------------+
                   |          |
             [lost]|          |[not lost]
                   v          v
              Drop ACK    Send ACK
```

---

## Sample Output

Sender console:

```
[14:32:15] Sender: Sending Frame 1
[14:32:15] Sender: ACK 1 received
[14:32:16] Sender: Sending Frame 2
[14:32:18] Sender: Timeout, retransmitting Frame 2
[14:32:18] Sender: ACK 2 received
[14:32:19] Sender: Sending Frame 3
[14:32:19] Sender: ACK 3 received
[14:32:20] Sender: Sending Frame 4
[14:32:22] Sender: Timeout, retransmitting Frame 4
[14:32:22] Sender: ACK 4 received
[14:32:23] Sender: Sending Frame 5
[14:32:23] Sender: ACK 5 received
[14:32:24] Transmission Completed
```

Receiver console:

```
[14:32:15] Stop-and-Wait Receiver ready...
[14:32:15] Connected by ('127.0.0.1', 54321)
[14:32:15] Receiver: Frame 1 received
[14:32:15] Receiver: ACK 1 sent
[14:32:16] Receiver: Frame 2 received
[14:32:16] Receiver: Frame 2 lost!
[14:32:18] Receiver: Frame 2 received
[14:32:18] Receiver: ACK 2 sent
[14:32:19] Receiver: Frame 3 received
[14:32:19] Receiver: ACK 3 lost!
[14:32:20] Receiver: Frame 4 received
[14:32:20] Receiver: ACK 4 sent
[14:32:22] Receiver: Frame 4 received
[14:32:22] Receiver: ACK 4 sent
[14:32:23] Receiver: Frame 5 received
[14:32:23] Receiver: ACK 5 sent
```

Notice the duplicate Frame 4 reception caused by the lost ACK 3 — the sender never received ACK 3, timed out on Frame 3, but since it had already moved to Frame 4, it retransmitted Frame 4 instead. This demonstrates a subtle edge case in the Stop-and-Wait protocol.

---

## Performance Analysis

### Throughput Calculation

For Stop-and-Wait over a link with:

- Transmission time (T_t): time to push frame onto wire
- Propagation delay (T_p): time for signal to travel to receiver
- Processing time (T_pr): negligible for this simulation

The total time per frame is:

```
Time per frame = T_t + 2*T_p + T_ack
```

Utilization (efficiency) of the link:

```
U = T_t / (T_t + 2*T_p)
```

For a 1 Gbps link with 1000-byte frames and 10ms propagation delay:

```
T_t = 8000 bits / 1e9 bps = 8 microseconds
T_p = 10 milliseconds

U = 8e-6 / (8e-6 + 0.02) = 0.0004 = 0.04%
```

This demonstrates why Stop-and-Wait is extremely inefficient over high-bandwidth, high-latency links (like satellite or long-haul fiber). The sender spends most of its time idle, waiting for acknowledgments. Sliding window protocols (Go-Back-N, Selective Repeat) address this by keeping multiple frames in flight.

---

## Comparison: Stop-and-Wait vs Sliding Window

| Feature              | Stop-and-Wait           | Go-Back-N                    | Selective Repeat         |
|----------------------|-------------------------|------------------------------|--------------------------|
| Frames in flight     | 1                       | Up to W (window size)        | Up to W                  |
| Receiver buffer      | 1 frame                 | 1 frame (no buffering)       | W frames (buffering)     |
| Loss recovery        | Retransmit 1 frame      | Retransmit entire window     | Retransmit only lost     |
| Implementation       | Simplest                | Moderate                     | Most complex             |
| Efficiency (low loss)| Poor                    | Good                         | Best                     |
| Efficiency (high loss)| Poor                   | Degrades                     | Good                     |

---

## Extending the Project

Possible enhancements:

- Make frame loss and ACK loss probabilities configurable via command-line arguments
- Add sequence numbers larger than 1 bit (Stop-and-Wait only needs 1-bit sequence numbers: 0 and 1 alternating)
- Implement logging to a file for post-simulation analysis
- Calculate and display throughput statistics (frames successfully delivered per second)
- Add a configurable propagation delay to simulate network latency
- Implement alternating bit protocol with proper 0/1 sequence numbers
- Add a GUI visualization showing frames and ACKs moving between sender and receiver

