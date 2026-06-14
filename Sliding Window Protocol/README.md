# Sliding Window Protocol Theory & Specifications (Go-Back-N)

This documentation details the underlying theoretical mechanics, structural design, and operational state machines of the **Sliding Window Protocol** (specifically the **Go-Back-N (GBN)** variant) as implemented in the accompanying Python socket-based network simulation.

---

## 1. Core Theoretical Concepts

The Sliding Window Protocol is a fundamental **data link and transport layer protocol** designed to provide reliable, in-order delivery of data frames over an unreliable physical or logical channel. It addresses two primary challenges in computer networking: **Flow Control** and **Error Control**.

### Flow Control (The Pipelining Principle)

In primitive protocols like *Stop-and-Wait*, the sender must wait for an acknowledgment (ACK) for every single frame before transmitting the next. This results in poor network utilization (low throughput), especially over links with high propagation delays.

The Sliding Window Protocol introduces **pipelining**, allowing the sender to transmit multiple concurrent packets before receiving an acknowledgment. The maximum number of outstanding, unacknowledged frames permitted on the wire at any given moment is bounded by the **Window Size ($N$)**.

### Error Control (Go-Back-N Mechanism)

In a **Go-Back-N** ARQ (Automatic Repeat reQuest) structure:

* **The Sender Window** maintains a sequence of consecutive sequence numbers allowed for transmission.
* **The Receiver Window** has a strict size of $1$. It only looks for a single, specific sequence number (`expected`).
* If a frame is lost, corrupted, or arrives out of order, the receiver silences or rejects it. It does not buffer out-of-order packets.
* Upon a sender-side **Timeout**, the sender must roll back its execution state and retransmit *all* frames currently in flight within its window, starting from the oldest unacknowledged frame (hence, "Go-Back-$N$").

---

## 2. Mathematical & Parameter Bounds

The simulation operates within a deterministic set of variables that govern the behavior of both execution loops:

$$W = 4 \quad (\text{Window Size})$$

$$S_{\max} = 6 \quad (\text{Total Frames to Transmit})$$

$$T_o = 3.0\text{s} \quad (\text{Socket Timeout Value})$$

$$P_{\text{loss}} = 0.2 \quad (\text{Simulated Channel Packet Loss Probability})$$

### Sequence Number Space

For Go-Back-N to function correctly without ambiguity, the sequence number space ($M$) must satisfy the condition:

$$M \ge W + 1$$

Since our simulation tracks absolute frame counts ($1$ through $6$), the sequence allocation remains conflict-free.

---

## 3. Protocol State Machines & Logic

### Sender State Machine

The sender tracks two pointers dynamically adjusting across the execution lifecycle:

1. `base`: The sequence number of the oldest unacknowledged frame.
2. `next_frame`: The sequence number of the next frame to be sent out.

```
       [   Window Size (W = 4)   ]
      +---+---+---+---+---+---+---+---+
...   | 1 | 2 | 3 | 4 | 5 | 6 |   |   |  ...
      +---+---+---+---+---+---+---+---+
        ^               ^
        |               |
      base          next_frame
(Acknowledged)    (Eligible to Send)

```

#### State Transition Rules:

* **Transmission Condition:** Frames are continuously pushed to the network socket loopback if and only if:

$$\text{next\_frame} < \text{base} + W \quad \text{AND} \quad \text{next\_frame} \le S_{\max}$$


* **Cumulative ACKs:** The sender utilizes **Cumulative Acknowledgments**. When the receiver sends back an `ACK x`, it implies that *all* frames up to and including $x$ have been successfully received. The sender updates its base via:

$$\text{base} = x + 1$$


* **Timeout Event:** If no valid ACK arrives within $T_o$, a `socket.timeout` exception is raised. The sender assumes packet drops have halted progress, yielding:

$$\text{next\_frame} = \text{base}$$



The pipeline resets transmission backward to the `base` pointer position.

### Receiver State Machine

The receiver maintains a single, steady-state scalar variable: `expected`.

#### State Transition Rules:

1. **Frame Arrival ($f$):** A frame arrives over the TCP stream buffer.
2. **Loss Simulation Test:** A pseudo-random float is evaluated against $P_{\text{loss}}$. If $\text{random}() < 0.2$, the frame is synthetically discarded ("LOST") to evaluate protocol resiliency.
3. **Sequence Validation:**
* **If $f == \text{expected}$:** The frame is accepted in-order. The tracking state advances:

$$\text{expected} = \text{expected} + 1$$


* **If $f \neq \text{expected}$:** The frame is out-of-order (likely caused by a preceding frame drop). The frame is discarded.


4. **ACK Transmission:** The receiver calculates and transmits a cumulative ACK back to the sender channel:

$$\text{ACK}_{\text{sent}} = \text{expected} - 1$$



---

## 4. Failure Recovery Lifecycle Execution Trace

The diagrammatic sequence below outlines how the underlying logic handles a simulated frame drop, demonstrating the Go-Back-N synchronization loop:

```text
  Sender Loop State                                        Receiver Loop State
==============================================================================
[base=1, next=1] ---- Sends Frame 1 --------------------> [expected=1]
                      Validates Match (1==1) -> ACK 1 ->  [expected shifts to 2]
[base=2, next=2] <--- Receives ACK 1 (base=1+1=2)

[base=2, next=2] ---- Sends Frame 2 --------------------> [CRASH / SIMULATED DROP]
                      (Packet never enters processing)

[base=2, next=3] ---- Sends Frame 3 --------------------> [expected=2]
                      Mismatched Seq (3!=2) -> Discard!
                      Sends Cumulative ACK 1 ---------->  (Still only claims up to 1)

[base=2, next=4] ---- Sends Frame 4 --------------------> [expected=2]
                      Mismatched Seq (4!=2) -> Discard!
                      Sends Cumulative ACK 1 ---------->

[base=2, next=5] <--- Receives ACK 1 (Ignored, base stays 2)
[Window is Full: next_frame (5) == base (2) + W (4) -> Sender halts transmission]

################### SENDER SOCKET TIMEOUT TRIGGERS (3.0 Seconds) ###################

[base=2, Reset Loop] -> next_frame = base (2)

[base=2, next=2] ---- Resends Frame 2 ------------------> [expected=2]
                      Validates Match (2==2) -> ACK 2 ->  [expected shifts to 3]
[base=3, next=3] <--- Receives ACK 2 (base=2+1=3)

```

## 5. Performance and Limitations

While Go-Back-N drastically increases link utilization compared to Stop-and-Wait under nominal network conditions, it exhibits performance degradation over highly noisy channels ($P_{\text{loss}} \gg 0$).

Because the receiver drops all out-of-order packets, a single lost frame causes the sender to retransmit subsequent frames that may have already reached the receiver perfectly intact. This redundant transmission overhead wastes bandwidth—a limitation solved by more complex protocols such as **Selective Repeat (SR)**, which utilize individual acknowledgments and receiver-side buffering.