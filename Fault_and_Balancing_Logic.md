# Fault and Balancing Logic

I have been working on the fault logic after the measurement summary is updated. The firmware is checking the current pack state against the limits we have been using in the design: overvoltage near 4.20 V, undervoltage near 3.00 V, overtemperature near 60 C, and undertemperature near 0 C.

I am sending faults to the viewer using six status bytes. The first byte is a bit field:

- bit 0 means overvoltage
- bit 1 means undervoltage
- bit 2 means overtemperature
- bit 3 means undertemperature
- bit 4 means invalid data

The next bytes are being used for the cell index related to the fault. This felt like a good compromise because the viewer can quickly check one byte to know whether something is wrong, but it can still show which cell triggered the problem.

For balancing, I am keeping the logic conservative. Passive balancing should only run when the pack is otherwise healthy. If there is an OV, UV, OT, UT, invalid-data condition, or another software fault, the firmware clears the discharge requests instead of trying to keep balancing.

The threshold balancing code is clearing old requests every pass, checking only valid cell readings, and then enabling discharge for cells above the selected threshold. I am also using a rotating modulo pattern so the same local cell position is not always selected first. With the current settings, this keeps the prototype behavior limited and easier to observe on the bench.

The main note here is that balancing is not protection. Balancing helps improve cell matching, but if the pack is already outside the safe region, the system should stop optional balancing and focus on fault handling.

Checks I still want to run:

- Trigger OV/UV/OT/UT conditions and confirm the correct status bit appears.
- Confirm that balancing requests stop as soon as a fault is active.
- Repeat the software balancing simulation and keep the result as evidence that spread decreases without crossing limits.
