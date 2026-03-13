# UART Telemetry Protocol

I have been using the UART packet as the clean handoff between the STM32 and the Python viewer. The packet needs to be compact, but it also needs enough structure that the viewer can tell the difference between a real frame and random serial bytes.

The packet I am working with contains:

- start bytes `0xAA 0x55`
- a length byte
- STM32 timestamp in milliseconds
- six cell voltages in millivolts
- six cell temperatures in centi-degrees C
- pack voltage in millivolts
- signed pack current in deci-amps
- six status bytes
- CRC16-CCITT

The start bytes are mainly for resynchronization. If the viewer starts reading in the middle of a stream, it can scan forward until it finds the next likely packet boundary. The length byte is another quick sanity check before decoding.

I added CRC16 because UART by itself does not protect the data. The firmware calculates the CRC over the packet without the CRC field, and the viewer does the same calculation before accepting the frame. If the CRC does not match, the viewer drops the frame and keeps searching. This is important because a corrupted packet should not update the displayed battery state.

I kept the current field in the packet even though the current path is still being calibrated. Right now that is helping keep the viewer forward-compatible. Later, once the shunt and amplifier scaling are confirmed, the same field can carry real charge/discharge current.

One detail I am keeping for debugging is explicit byte order. The firmware manually fills the transmit buffer and the Python side decodes with a little-endian format. It is more verbose, but it makes packet mismatch problems easier to find.

Tests I still want:

- Decode normal packets from firmware and simulator.
- Add random bytes before a packet and confirm the parser recovers.
- Corrupt one byte and confirm CRC rejection.
- Truncate a packet and confirm the GUI does not update from it.
