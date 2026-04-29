# Verification and Integration

The software is far enough along that it can support board bring-up, but I am trying to be careful about the difference between something being implemented and something being verified. The firmware and viewer are already passing voltage, temperature, current field, and status information through the telemetry path, but the actual measurement accuracy still needs hardware evidence.

The main verification areas I am tracking are:

- Measurement processing: raw LTC6811-style readings are being converted and summarized correctly.
- Fault handling: threshold crossings are setting the right status bits and cell indices.
- Balancing behavior: discharge requests are only happening when balancing is allowed.
- Telemetry integrity: firmware and viewer agree on framing, length, byte order, and CRC.
- GUI behavior: live updates, fault display, timeout handling, plots, and logs are working consistently.

Bench checks I need to run:

- Compare displayed pack voltage with a calibrated DMM at different pack voltages.
- Compare individual cell readings against known cell simulator or battery values.
- Warm or substitute thermistors and confirm the correct temperature group changes in the viewer.
- Inject OV, UV, OT, and UT conditions and record how long the GUI takes to show the fault.
- Corrupt or truncate UART packets and confirm the parser rejects them without crashing.
- Run the viewer with both real UART and demo/simulator data so I can separate GUI issues from hardware issues.

Current sensing still needs the most care. The design is using a shunt and current-sense amplifier into the STM32 ADC, but the final current value depends on shunt resistance, gain, offset, and ADC reference. A wrong scale factor could look reasonable on screen while still being wrong, so I should not treat SOC/current as validated until it is checked against a known load.

Near-term plan:

- Finish voltage and temperature validation first.
- Calibrate current after the current-sense path is stable.
- Use CSV logs during fault testing so the response times are documented.

## Development Board Fallback

The main STM32 on the primary board was damaged, so I switched over to a development board as the active STM32 platform for continued bring-up work. That change was mainly about keeping progress moving while separating firmware validation from uncertainty in the damaged hardware.

To check whether the slave board path and main board interfaces still behave as expected, I also added a firmware-side simulation path. The goal of that simulation is not to claim the hardware is fully verified, but to answer a narrower question first: can the STM32 application still generate valid telemetry and transmit it over UART in the same packet format the viewer expects?

The mock firmware builds synthetic cell voltages, temperatures, pack voltage, current, and status bits, then sends those values over UART on a fixed transmit period. That gives me a controlled way to test the telemetry link, the packet structure, and the viewer behavior even when the full measurement chain is not available on the damaged main board.

For more detail and to understand exactly how the simulation is implemented, check the `mock_demo` folder under `senior_dsg_proj`. That folder is the current reference for the development-board-based STM32 simulation used during integration and UART bring-up.
