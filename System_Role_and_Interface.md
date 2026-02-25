# System Role and Interface

I have been thinking of my part of the project as the connection between the battery monitor hardware and the person looking at the battery state. The LTC6811 is collecting the actual cell data, the STM32 is making decisions from it, and the desktop viewer is where that information becomes readable during testing.

The flow I am working with is:

- LTC6811 reads cell voltages and thermistor channels.
- STM32 polls the monitor, summarizes pack values, checks limits, and handles balancing decisions.
- STM32 sends a structured UART packet.
- The Python viewer checks, decodes, displays, and logs the packet.

The main thing I keep coming back to is that the UART packet is the contract between the embedded side and the viewer. I am using that boundary to keep the work testable even when the boards are not fully ready. If the packet format stays consistent, I can keep building the viewer with demo data and then plug in real UART data later.

For the viewer, I am focusing on values that are useful during bring-up: pack voltage, pack current, cell voltages, temperature readings, cell imbalance, SOC/SOH fields, and fault flags. Minimum cell voltage and active fault state are probably the most important because they tell us quickly whether the pack is near an unsafe condition.

I have also been avoiding the viewer turning into a plain serial terminal. It should only update from valid BMS frames. If the data is missing, corrupted, or stale, the GUI should show that state clearly instead of continuing to display old values as if they are live.

Current notes to keep in mind:

- Current measurement is still being integrated, so I am keeping the interface ready for it without treating it as fully verified.
- SOC and current should be shown carefully until calibration is done.
- Firmware thresholds and viewer warning colors need to stay consistent.
