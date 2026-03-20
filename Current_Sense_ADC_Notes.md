# Current Sense and ADC Notes

The expected path is:

- pack current through shunt resistor
- small shunt voltage into current-sense amplifier
- amplifier output into STM32 ADC
- firmware converts ADC counts to current
- viewer displays signed current and logs it

The risky part is that the math can look right while the number is still wrong. The final current depends on the actual shunt value, amplifier gain, ADC reference, and zero-current offset. If any of those are off, SOC will drift and the viewer can show a believable but incorrect current.

Things I need to measure or confirm:

- ADC value at zero current
- output voltage from the amplifier at zero current
- ADC value with a known load
- sign direction for charge vs discharge
- whether the signal is noisy enough to need more filtering

Firmware-side idea:

- store raw ADC value while debugging
- convert to milliamps or deci-amps after offset correction
- keep current field in UART packet even if value is temporarily zero/simulated
- do not use current-based SOC as final until calibration is done

For the viewer, I should probably display current but avoid making it look more precise than it is. Showing too many decimal places would be misleading before calibration. For now, one decimal place is probably enough.

Main reminder: current is important, but voltage and temperature are the first things I should validate. SOC depends on current, so current calibration has to come before any serious SOC claim.
