# STM32 UART Pinout Changes

I helped with the STM32 pinout changes when we needed a cleaner UART path for the desktop viewer. This was partly firmware work and partly board/interface cleanup, because the UART pins have to match what the STM32 alternate function table actually supports and what is practical to route on the board.

The main goal was to get a dedicated TX/RX pair for telemetry instead of treating serial output as a temporary debug feature. The viewer depends on UART as the real interface, so it needed to be assigned intentionally in the pinout instead of added after everything else.

The checks I went through were:

- confirm which USART instance was available on the STM32 package
- check alternate function mapping for candidate TX/RX pins
- avoid pins already needed for SPI/isoSPI control, ADC, or boot/config signals
- make sure the pins were accessible for bringup/debug
- update firmware initialization to match the selected UART instance and pins

One thing I had to keep reminding myself is that "UART works in firmware" is not enough if the board pinout routes the wrong alternate function. The STM32 can have several possible pins for a USART, but not every pin supports every peripheral. It is easy to make a pinout that looks reasonable at the connector level and still does not match the MCU AF mapping.

On the firmware side I kept the UART code packet-based instead of printf-based. Printing debug strings is helpful early, but for this project the Python viewer needs binary frames with start bytes, length, timestamp, values, status, and CRC. That means the selected UART pins are part of the actual system interface, not just a debug console.

Notes from this work:

- The UART connector should have a clear ground reference next to TX/RX.
- TX/RX direction needs to be labeled from the STM32 point of view.
- The firmware baud rate and viewer baud rate should be written down together.
- The pinout should leave room to connect a USB-serial adapter without disturbing the rest of the board.

This change made the software work easier because I could treat the serial link as a stable boundary. Once the STM32 can send the same packet format from either the real board or a dev board, the viewer does not need to know which hardware is currently connected.
