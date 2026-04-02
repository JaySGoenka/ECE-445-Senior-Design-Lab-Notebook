# BMS Viewer Design Notes

Quick summary: this entry is the rough design list for what the BMS display should show during bring-up. I kept it focused on what I would actually need on the bench instead of making it a polished dashboard plan.

This page is more of a scratchpad for what I wanted the viewer to show. The viewer is not supposed to be a polished product dashboard. It is mainly the thing I use during bring-up to tell if the pack and firmware are doing something reasonable.

The most important values to show:

- pack voltage
- pack current, once the current path is calibrated
- six cell voltages
- min cell and max cell
- cell imbalance
- two temperature groups
- active fault bits
- communication state
- SOC/SOH fields, but only with the caveat that SOC depends on current calibration

I want min cell voltage to be easy to see because that is usually the quickest safety check. Pack voltage is useful, but it can hide one weak cell. The imbalance number is also useful because it tells us if balancing even matters for the current test.

The viewer should avoid acting like a serial terminal. It should not show every random byte. It should only update when the packet passes the start-byte, length, and CRC checks. If the packet is bad, the viewer should ignore it and keep looking for the next real frame.

Layout idea:

- top row for pack summary values
- six repeated cell boxes below that
- fault/status panel on the side
- plots below or in another tab
- logging button somewhere obvious

I originally listed more SOC features like Kalman SOC, coulomb counting SOC, pack power, and time remaining. Those are still good stretch goals, but for the final prototype the basic live measurements and fault display are more important. A fancy SOC number is not very useful if the current sense offset is wrong.

Implementation notes:

- voltage should be displayed in V, but transmitted in mV
- temperature should be displayed in C, but transmitted as integer scaled values
- current sign convention needs to be written down before calling the display done
- warning/fault colors should match the firmware thresholds
- CSV logs should include timestamps so I can compare them with DMM/scope notes

Acceptance check for myself:

- Can I plug in the STM32 and immediately tell if packets are live?
- Can I see which cell is lowest without reading a table carefully?
- Does the GUI clearly show timeout instead of leaving old data on screen?
- Can I save a log during a fault or calibration test?
