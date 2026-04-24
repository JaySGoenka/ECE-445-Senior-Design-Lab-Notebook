# SOC and SOH Estimation Notes

I looked more closely at the SOC/SOH calculation steps because these values can look precise even when the inputs are not fully proven. Cell voltage, current calibration, elapsed time, and the assumed pack capacity all affect the final number. For our prototype, I think it is better to show the method clearly and keep the limitations visible.

The SOC calculation is a hybrid of voltage estimate and coulomb counting:

- start with an SOC estimate from average cell voltage
- after startup, subtract charge used based on pack current and elapsed time
- ignore tiny currents below the noise floor so offset drift does not slowly drain SOC
- if the pack is resting at low current, blend SOC back toward the voltage-based estimate
- force SOC near zero if the minimum cell reaches the empty-cell cutoff
- clamp the final value between 0% and 100%

The voltage estimate uses an approximate Li-ion open-circuit voltage curve. It is not just a straight line from 3.0 V to 4.2 V because the middle of the discharge curve is flatter. The rough lookup points I am using are:

- 3.00 V is 0%
- 3.30 V is about 5%
- 3.50 V is about 12%
- 3.60 V is about 22%
- 3.70 V is about 35%
- 3.75 V jumps up near 76%
- 3.80 V is about 80%
- 3.90 V is about 88%
- 4.00 V is about 94%
- 4.10 V is about 98%
- 4.20 V is 100%

That curve is only a startup/rest correction, not the whole SOC answer. Under load, cell voltage sags, so using voltage directly can make the pack look more discharged than it really is. That is why the current integration matters once the pack is actively discharging.

For coulomb counting, the basic step is:

`SOC_drop_percent = Ipack * dt * 100 / (capacity_Ah * 3600)`

The capacity value I am using right now is 2.2 Ah. Current below about 0.10 A is treated as zero for SOC integration so sensor noise and offset do not accumulate forever. I also cap each time step so one delayed packet or timestamp jump does not create a huge fake SOC drop.

Rest correction is a small compromise. If the current stays below about 0.15 A for a few seconds, the pack is treated as close enough to resting that voltage is more useful again. Instead of snapping SOC straight to the voltage estimate, I blend partway toward it. This avoids sudden jumps but still corrects slow drift from coulomb counting.

Cutoff behavior is based on the minimum cell, not just average voltage. If the lowest cell is at or below 3.00 V, SOC should be treated as 0%. If the lowest cell is only slightly above that, around 3.05 V, SOC should be limited to a very low value. This is important because a healthy-looking average can hide one weak cell.

SOH is based on measured discharge capacity. The idea is:

- start a discharge cycle when real discharge current is present
- only trust a cycle for SOH if it started from a high SOC, around 90% or more
- integrate discharged amp-hours during the cycle
- end the cycle when current drops close to idle and the minimum cell is near the cutoff voltage
- compare measured discharged Ah against nominal capacity

The SOH equation is:

`SOH_percent = measured_discharge_capacity_Ah / nominal_capacity_Ah * 100`

Using the 2.2 Ah nominal capacity, a measured discharge of 2.178 Ah would be about 99% SOH. A measured discharge of 2.156 Ah would be about 98% SOH. This makes sense as a capacity-based estimate, but it depends heavily on having a real full-to-empty discharge and a calibrated current measurement.

Things I still need to be careful about:

- zero-current offset has to be measured
- the current sign convention has to be right
- current should be checked against a known load
- SOC should not keep drifting when current is actually zero
- SOH should not be updated from a partial discharge that did not start near full
- voltage-based SOC should only be trusted more when the pack is resting

Main note: SOC and SOH are useful user-facing estimates, but protection should still rely more directly on cell voltage, temperature, and fault flags. I should not present SOC/SOH as fully validated until current calibration and discharge testing are actually done.
