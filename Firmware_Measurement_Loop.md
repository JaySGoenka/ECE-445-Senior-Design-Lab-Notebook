# Firmware Measurement Loop

I have been keeping the STM32 firmware as a simple periodic loop. I considered whether we needed a more complicated scheduler, but the current tasks are still simple enough to handle with `HAL_GetTick()` timing.

The loop is currently being organized around these periods:

- Cell voltage polling is running every 200 ms.
- UART telemetry is being sent every 200 ms.
- Temperature polling is running every 500 ms.
- Heartbeat/debug timing is available around 250 ms.

This gives about a 5 Hz telemetry rate, which is faster than the viewer requirement. I am leaving temperature slower because it does not change as quickly as voltage, and polling it too often does not add much useful information right now.

On the voltage side, I am using the LTC6811 scaling where each count is 100 uV. That means a reading of `42000` is `4.2000 V`. After the voltage read, the firmware is scanning the six used cells, finding the min and max, recording which cell caused each extreme, and summing the pack voltage in millivolts for the viewer.

I added checks so obviously invalid voltage readings do not get folded into the pack summary. I still want invalid data to be visible as a fault/status condition, but I do not want one bad reading to quietly become the pack minimum or maximum and confuse the rest of the system.

For temperature, I am using the current two-thermistor setup. Cells 1-3 share one temperature value and cells 4-6 share the other. This is not perfect per-cell sensing, but it matches the current prototype and is enough to see whether one half of the pack is warming up.

Things I still need to check on hardware:

- Compare LTC6811 cell readings against a DMM.
- Check that summed pack voltage matches the pack terminals closely enough.
- Confirm the two thermistor groups show up on the correct cells in the viewer.
- Make sure the temperature scaling is the same in firmware and Python.
