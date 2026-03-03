# Hardware Bringup Checks

These are basic hardware bring-up notes I wanted to keep because a lot of the firmware work depends on the board being in a known state. I mostly helped from the firmware/software side, but I still needed to check enough hardware to avoid chasing fake software bugs.

Before blaming firmware, I tried to check:

- board power rails are present
- STM32 is actually powered and not held in reset
- UART TX/RX lines are on the pins the firmware is using
- ground is shared with the USB-serial adapter
- isoSPI/LTC6811 side has activity when commands are sent
- obvious solder/damage issues around the monitor IC and connectors

One useful habit was to check signals at the pin or connector, not just assume the schematic connection made it to the board. For UART especially, it is easy to have firmware configured correctly but the physical TX/RX direction swapped or connected to a different alternate-function pin.

The oscilloscope was useful for separating "code did not receive a response" from "nothing is physically responding." For the isoSPI problem, seeing the command side active and the LTC6811 response side wrong helped point us toward damaged hardware instead of endlessly changing firmware timing.

Basic notes for future bring-up:

- Start with power and reset before communication.
- Use a simple known UART packet or heartbeat first.
- Probe communication lines during the actual firmware transaction.
- Write down which exact pins and headers were used.
- If a chip seems dead, compare the waveform to a known-good or expected case before making more firmware changes.

This is rough, but it kept me grounded. A lot of embedded bugs look the same from the terminal, so the bench measurements matter.
