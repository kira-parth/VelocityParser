# VelocityParser
Ultra-Low Latency NASDAQ ITCH 5.0 Parser | &lt;100ns Market Data Processing
<img width="1913" height="939" alt="Screenshot 2025-10-20 143831" src="https://github.com/user-attachments/assets/91020461-0980-46d3-94db-e432758f5584" />


A hardware-accelerated market data parser achieving sub-100ns packet processing latency on FPGA. Designed for real-time financial data feeds with zero packet loss and configurable message filtering.
#🎯 Overview
TickStream-FPGA is a production-grade FPGA implementation of a NASDAQ ITCH 5.0 protocol parser optimized for high-frequency trading applications. The design processes Ethernet/IP/UDP packets in real-time, extracts ITCH market data messages, and applies configurable filtering to reduce downstream processing overhead.
Key Features

.Sub-100ns Latency: End-to-end packet processing in <100 nanoseconds
.10M+ pps Throughput: Handles over 10 million packets per second
.Zero Packet Loss: Validated with comprehensive testbench
.Configurable Filtering: Runtime-configurable 16-bit message type filter
.AXI-Stream Interface: Industry-standard streaming interface
.Production Ready: Statistics counters, error handling, runtime configuration


