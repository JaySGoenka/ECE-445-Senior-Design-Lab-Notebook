# isoSPI Communication Debugging

I helped debug the isoSPI/LTC6811 communication issue when the battery monitor side was not responding correctly. This was one of the more useful hardware debugging sessions for me because the problem was not obvious from firmware alone. From the STM32 side it mostly looked like bad or missing responses, but that did not tell us whether the problem was code, routing, transformer/coupling parts, or the LTC6811 itself.

The first step was to stop guessing from firmware return values and look at the actual signals. We used the oscilloscope to check whether the expected activity was present around the isoSPI path. I was mainly looking for whether commands were leaving the controller side and whether anything reasonable was coming back from the LTC6811 side.

What I observed:

- The transmit side showed activity when the firmware attempted communication.
- The expected response from the monitor side was not present or was not shaped like a valid response.
- Repeating the command sequence did not make the issue intermittent; it looked consistently wrong.
- The issue followed the monitor IC side more than the STM32/software side.

After checking the surrounding connections and comparing against what the signals should roughly look like, the most likely failure was a damaged LTC6811. The scope was important here because without it I could have wasted more time changing SPI timing or packet code. The firmware cannot fix a monitor IC that is not physically driving the bus correctly.

Things I learned from this debug:

- A failed isoSPI transaction should be treated as a hardware/software boundary problem until the waveform is checked.
- It is useful to probe both the controller-side command and the monitor-side response instead of only looking at one point.
- If the command waveform exists but the response is dead or distorted, firmware changes are probably not the first thing to try.
- The LTC6811 and isoSPI parts should be handled carefully during bringup because damage can look like a protocol bug.

This also affected how I wrote the firmware. I added and kept invalid-data handling because communication failures should propagate into a clear status instead of creating fake pack readings. The viewer should show that the BMS link/measurement path is not valid, not keep updating as if the cells were being read normally.
