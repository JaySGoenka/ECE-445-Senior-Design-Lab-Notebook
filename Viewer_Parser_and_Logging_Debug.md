# Viewer Parser and Logging Debug

I spent a good amount of time making the Python viewer tolerate real serial behavior. The first simple parser assumed that one serial read would equal one full packet, which is not a safe assumption. In practice the operating system can give the program part of a packet, multiple packets, or a packet with extra bytes before it.

The parser now uses a rolling buffer. It searches for the start bytes, checks the length, waits until enough bytes are present, checks CRC, and only then updates the displayed BMS state. This made the viewer much more useful because bad serial chunks do not immediately crash the GUI or shift every field by one byte.

Some cases I specifically wanted the parser to handle:

- random bytes before a valid frame
- half a frame arriving first and the rest arriving later
- two frames arriving in one serial read
- a bad CRC frame followed by a good frame
- no valid frames for long enough that the GUI should show a timeout

The timeout behavior matters more than I first thought. During bringup it is easy to stop the STM32, reset the board, or unplug the adapter. If the GUI just keeps the last numbers on screen, it looks like the pack is still healthy and updating. I added a communication status so stale data is visible.

CSV logging is also there for verification, not just convenience. I want logs for voltage comparison, fault response time, and general debugging when a value briefly glitches. Looking at a live plot is helpful, but a CSV gives something I can compare later against notes from the DMM or oscilloscope.

Current rough edges:

- The current sign convention still needs to match the final hardware direction.
- The viewer threshold colors need to stay aligned with firmware limits.
- I should keep testing with both simulated UART chunks and real STM32 output.

This viewer work ended up being part of firmware debugging too. When the GUI rejects a bad CRC or times out cleanly, it becomes easier to separate serial framing problems from actual BMS measurement problems.
