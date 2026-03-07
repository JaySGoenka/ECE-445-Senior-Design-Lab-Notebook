# STM32 Firmware Bringup Notes

I spent most of my time on the STM32 side trying to keep the firmware simple enough that it could still be debugged on the bench. I did not want a lot of RTOS/task code hiding the basic timing problems, so the main structure is still a polling loop with explicit periods for measurement, telemetry, and slower temperature updates.

The first version of the loop was too mixed together. Voltage reads, fault checking, balancing decisions, and UART transmit were all close to each other, which made it harder to tell where a bad value was coming from. I split the work mentally into:

- read raw measurements from the monitor side
- convert them into firmware units
- build a pack summary
- apply limits/fault logic
- send only the summarized state to the viewer

That helped because the UART packet should not care about the exact way the LTC6811 read happened. The viewer only needs stable units and flags. I used millivolts and centi-degrees C so the packet stays integer-based and the Python side does not have to guess about float formatting.

One annoying issue was making sure invalid data does not quietly look like a real cell. If a read fails or a value is obviously out of range, it should set a status bit and not become the min cell, max cell, or pack voltage. Otherwise the GUI can show a very specific looking number that is actually just garbage from the measurement path.

For the cell summary I am tracking:

- per-cell voltage in mV
- min and max cell voltage
- which cell caused the min/max
- pack voltage from the sum of valid cells
- voltage spread for imbalance

I also left the current value in the data structure even while current sensing was not fully calibrated. This felt slightly awkward, but it kept the firmware/viewer interface from changing every time the hardware status changed. During demos or partial bringup the current field can be zero or simulated, and later it can use the ADC path once the scale factor is confirmed.

Things I checked or still need to re-check:

- Whether the firmware keeps sending packets even when measurement data is invalid.
- Whether the viewer shows stale/timeout state if UART stops.
- Whether the fault byte changes at the same time as the bad cell value.
- Whether the pack voltage is calculated from the same cells the viewer is displaying.

My main takeaway from the firmware work is that the BMS code should fail visibly. A quiet bad reading is worse than no reading because it makes the rest of the system look more correct than it is.
