# CS 371 – Programming Assignment 2: Error Control & Flow Control in RDT

> **Course:** CS 371: Introduction to Computer Networking – Spring 2026, University of Kentucky
> **Assigned:** February 27, 2026 | **Due:** March 9, 2026
> **Authors:** Biplav Khatiwada, Hamblet Arroyo Alvarado

---

## Overview

This assignment extends PA1's client-server application to use **UDP sockets** and implements reliability mechanisms on top of an inherently unreliable transport layer.

Two tasks are implemented:

- **Task 1** — UDP-based Pipelined Protocol with packet loss measurement
- **Task 2** — Go-Back-N retransmission with sequence numbers and ACKs to eliminate packet loss

---

## Files

| File | Description |
|---|---|
| `pa2_task1.c` | Task 1 — UDP pipelined protocol with loss tracking |
| `pa2_task2.c` | Task 2 — Go-Back-N with SN, ACK, and retransmission |

---

## Build

> **Requires:** Ubuntu 22.04, x86 CPU (tested on NSF CloudLab)

```bash
# Task 1
gcc -o pa2_task1 pa2_task1.c -pthread

# Task 2
gcc -o pa2_task2 pa2_task2.c -pthread
```

---

## Usage

```bash
# Start the server
./pa2_task1 server <server_ip> <server_port>

# Start the client
./pa2_task1 client <server_ip> <server_port> <num_client_threads> <num_requests>
```

Same usage applies for `pa2_task2`.

**Example:**

```bash
# Terminal 1 — start the server
./pa2_task1 server 127.0.0.1 12345

# Terminal 2 — run 4 client threads, each sending 1,000,000 messages
./pa2_task1 client 127.0.0.1 12345 4 1000000
```

---

## Task 1 — UDP Pipelined Protocol

Extends PA1's Stop-and-Wait protocol to UDP with a sliding window (pipeline) approach:

- Uses `SOCK_DGRAM` (UDP) instead of TCP
- Sends up to `WINDOW_SIZE` (32) packets in-flight before waiting for ACKs
- Tracks per-thread packet loss via `tx_cnt`, `rx_cnt`, and `lost_pkt_cnt`
- As the number of client threads grows, the server becomes saturated and **packet loss becomes observable** — this is the "fan-in" effect

**Output per thread:**
```
Thread N: tx=... rx=... lost=... messages=... avg_rtt=... us rate=... messages/s
```

---

## Task 2 — Go-Back-N with Retransmission

Builds on Task 1 and adds reliability over UDP:

- Each packet carries a **sequence number** (`seq_num`) and **client ID**
- Client maintains a send buffer and an `acked[]` array to track acknowledgements
- On `epoll_wait` timeout (100ms), all unacknowledged packets from `base` to `nextseqnum` are **retransmitted**
- The window slides forward as cumulative ACKs arrive
- Result: `tx_cnt == rx_cnt` regardless of how many client threads are active — **zero packet loss**

---

## Protocol Design

```
Client                        Server
  |                              |
  |--- [seq=0] pkt ----------->  |
  |--- [seq=1] pkt ----------->  |
  |      ...  (window=32)        |
  |--- [seq=31] pkt -----------> |
  |                              |
  | <-- [seq=0] ACK ------------ |
  | <-- [seq=1] ACK ------------ |
  |      ...                     |
  |   (timeout) retransmit lost  |
  |--- [seq=K] pkt (resend) -->  |
```

---

## Development Environment

- **OS:** Ubuntu 22.04 (x86)
- **Platform:** [NSF CloudLab](https://cloudlab.us/) 

> Code **must** run on x86 Ubuntu 22.04 on NSF CloudLab before submission.

---

## References

- [Practical TCP/IP Sockets in C](https://cs.baylor.edu/~donahoo/practical/CSockets2/)
- [Linux epoll(7) man page](https://man7.org/linux/man-pages/man7/epoll.7.html)
