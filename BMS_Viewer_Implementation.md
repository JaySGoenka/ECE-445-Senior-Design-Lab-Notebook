# BMS Viewer Implementation

I have been building the Python viewer which is the main interface for the end user to our system. The goal is that when the board is connected, I can quickly tell whether the pack looks normal, whether a cell is drifting, and whether the communication link is still alive.

The viewer currently has:

- summary cards for pack voltage, current, average cell voltage, imbalance, SOC, and SOH
- six cell cards with voltage bars, temperature, and a state label
- a fault panel for OV, UV, OT, UT, imbalance, and communication status
- live plots for voltage, temperature, and current
- CSV logging for later analysis

I built the parser around a rolling byte buffer because serial reads are messy in practice. The viewer may receive one byte, half a packet, or several packets at once. The parser is searching for `0xAA 0x55`, checking the packet length, waiting until it has all bytes, verifying CRC, and only then decoding the values.

Demo mode has been useful while hardware is still being brought up. The simulator is generating six cell voltages, two grouped temperature readings, pack current, pack voltage, and status bits. It is also splitting packets into small chunks with random delays, which is a good way to test whether the parser handles UART fragmentation.

I added a communication watchdog because stale data is a real risk. If no valid frame arrives for about a second, the viewer marks the link as timed out. I do not want the GUI to keep showing old healthy values if the board has stopped sending data.

CSV logging is being kept because I will need evidence during verification. The logs should help compare pack voltage against a DMM, check the actual update interval, and measure how long fault indications take to appear.

Next things I am watching:

- Current display needs to match the final charge/discharge sign convention.
- Warning and fault colors should stay aligned with firmware thresholds.
- Logs should be used during real hardware tests, not just demo mode.
