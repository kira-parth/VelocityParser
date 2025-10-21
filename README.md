# VelocityParser
Ultra-Low Latency NASDAQ ITCH 5.0 Parser | &lt;100ns Market Data Processing

<img width="1913" height="939" alt="Screenshot 2025-10-20 143831" src="https://github.com/user-attachments/assets/91020461-0980-46d3-94db-e432758f5584" />


A hardware-accelerated market data parser achieving sub-100ns packet processing latency on FPGA. Designed for real-time financial data feeds with zero packet loss and configurable message filtering.


# 🎯 Overview
TickStream-FPGA is a production-grade FPGA implementation of a NASDAQ ITCH 5.0 protocol parser optimized for high-frequency trading applications. The design processes Ethernet/IP/UDP packets in real-time, extracts ITCH market data messages, and applies configurable filtering to reduce downstream processing overhead.


Key Features

* Sub-100ns Latency: End-to-end packet processing in <100 nanoseconds
* 10M+ pps Throughput: Handles over 10 million packets per second
* Zero Packet Loss: Validated with comprehensive testbench
* Configurable Filtering: Runtime-configurable 16-bit message type filter
* AXI-Stream Interface: Industry-standard streaming interface
* Production Ready: Statistics counters, error handling, runtime configuration

# 📊 Performance Metrics

<img width="671" height="210" alt="image" src="https://github.com/user-attachments/assets/1c645b96-1ed4-4571-8e99-0ce83ae19de9" />

# FPGA Architecture

       ┌─────────────┐
       │    IDLE     │
       └──────┬──────┘
              │ eth_in_tvalid
              ▼
       ┌─────────────┐
       │  RECEIVING  │◄───┐
       └──────┬──────┘    │
              │           │ !eth_in_tlast
              │ eth_in_tlast
              ▼           │
       ┌─────────────┐    │
       │ PROCESSING  │    │
       └──────┬──────┘    │
              │           │
         ┌────┴────┐      │
         ▼         ▼      │
    ┌────────┐ ┌──────┐   │
    │ OUTPUT │ │ DROP │   │
    └────┬───┘ └───┬──┘   │
         │         │      │
         └─────────┴──────┘
              │
              ▼
         ┌─────────┐
         │  IDLE   │
         └─────────┘



# Packet Structure
Ethernet Frame (Your Input)

Offset | Field              | Size    | Notes
-------|-------------------|---------|---------------------------
0      | Dest MAC          | 6 bytes | FF:FF:FF:FF:FF:FF (broadcast)
6      | Src MAC           | 6 bytes | Source MAC address
12     | EtherType         | 2 bytes | 0x0800 (IPv4)
14     | IP Header         | 20 bytes| IPv4 protocol header
34     | UDP Header        | 8 bytes | Source/Dest ports
42     | ITCH Message      | Variable| Actual market data



# ITCH Message Format

Offset | Field              | Size    | Example
-------|-------------------|---------|---------------------------
0      | Message Length    | 2 bytes | Big-endian length
2      | Message Type      | 1 byte  | 'A', 'E', 'X', etc.
3      | Message Body      | Variable| Type-specific data


# Memory Organization
Packet Buffer

* Size: 32 x 32-bit words (128 bytes total)
* Organization: Ring buffer with word-level addressing
* Access Pattern: Sequential write during RECEIVING state
  
Word 0-1: |  Ethernet header (MAC addresses)
----------|---------------------------------
Word 2-3: |  EtherType + IP header start
Word 4-8: |  IP header (protocol, addresses)
Word 9-10:|  UDP header (ports, length)
Word 11+: |  ITCH message payload

# Configuration Registers
Register Map

<img width="670" height="275" alt="image" src="https://github.com/user-attachments/assets/3f54ff85-90e1-4a5c-9ae8-600b73d933ac" />


# Message Type Filter Encoding
Bit Position | ITCH Type | Message Name
-------------|-----------|------------------
0            | 'S' (0x53)| System Event
1            | 'R' (0x52)| Stock Directory
2            | 'H' (0x48)| Trading Action
3            | 'Y' (0x59)| Reg SHO
4            | 'A' (0x41)| Add Order
5            | 'F' (0x46)| Add Order MPID
6            | 'E' (0x45)| Order Executed
7            | 'C' (0x43)| Order Executed Price
8            | 'X' (0x58)| Order Cancel
9            | 'D' (0x44)| Order Delete
10           | 'U' (0x55)| Order Replace
11           | 'P' (0x50)| Trade
12           | 'Q' (0x51)| Cross Trade
13           | 'B' (0x42)| Broken Trade
14           | 'I' (0x49)| NOII
15           | 'N' (0x4E)| RPII

# Output Data Format
297-bit Output Structure

Bits [296:265] | FPGA Timestamp (32 bits)
---------------|----------------------------
Bits [264:257] | ITCH Message Type (8 bits)
Bits [256:225] | Word 11 from buffer (32 bits)
Bits [224:193] | Word 12 from buffer (32 bits)
Bits [192:161] | Word 13 from buffer (32 bits)
Bits [160:129] | Word 14 from buffer (32 bits)
Bits [128:97]  | Word 15 from buffer (32 bits)
Bits [96:65]   | Word 16 from buffer (32 bits)
Bits [64:33]   | Word 17 from buffer (32 bits)
Bits [32:1]    | Word 18 from buffer (32 bits)
Bit  [0]       | Reserved (always 0)


