# Integration Work Sessions

These are rough notes from the integration side, mostly from trying to make the firmware, board, and viewer useful at the same time instead of each piece only working by itself.

## Firmware and Viewer Packet Matching

I checked the STM32 packet layout against the Python decoder. The main things that can silently break this are byte order, field size, and the CRC range. I kept the firmware transmit buffer filled manually so it is obvious which byte each field occupies. It is more tedious than sending a struct, but it avoids padding/alignment surprises.

The viewer side decodes little-endian values and rejects packets if the start bytes, length, or CRC do not match. I used simulated packets first because it is faster to change values and force edge cases without needing the hardware to be in a specific state.

## Dev Board Fallback

When the main STM32 hardware was not reliable, I used the development board path to keep the software moving. The goal was not to pretend the whole BMS was verified. It was just to keep the UART telemetry, packet parser, GUI display, and logging testable while the main board issues were being sorted out.

The simulated firmware sends realistic-looking cell voltages and temperatures, plus status flags. This gave me a way to test the viewer behavior before the LTC6811 path was ready again.

## Hardware Bringup Notes

On the hardware side I helped check the isoSPI problem with the oscilloscope and helped narrow it down to the LTC6811 being damaged. This was a good reminder that embedded debugging needs actual signal checks. If the firmware says communication failed, that is only a symptom.

I also helped with the STM32 UART pinout changes so the board had a usable serial interface for the viewer. That included checking alternate functions and making sure the UART selected in firmware matched the physical pins.

## Remaining Integration Risks

- Cell voltage accuracy still needs DMM comparison.
- Temperature grouping needs to be checked against the physical thermistor locations.
- Current sensing needs calibration before SOC should be trusted.
- Fault flags should be tested on hardware, not only with simulated values.
- The viewer should be run during hardware tests so the CSV logs become verification evidence.

The main integration lesson for me was to keep each boundary testable. If the LTC6811 path is down, UART and GUI can still be tested. If the GUI is acting weird, simulated packets can test it without the board. That kept me from blocking completely when one part of the system was damaged.
