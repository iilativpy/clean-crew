# Clean embedded architecture (Clean Architecture ch. 29)

Load this only for firmware/embedded/hardware-near code (drivers, devices, IoT, RTOS).

- Software doesn't wear out, but firmware becomes obsolete as hardware evolves. Don't let
  all your code become firmware: **minimize code that knows about the hardware**.
- "First make it work, then make it right, then make it fast" — embedded teams often stop
  after the first step ("app-titude"), leaving code that only runs on the target.
- **Target-hardware bottleneck**: if code can be tested only on the target, development stalls
  (hardware late, scarce, buggy). A **clean embedded architecture is testable off-target**.
- **Layers**: software / firmware / hardware, with explicit seams:
  - **HAL (Hardware Abstraction Layer)**: software depends on a HAL designed for the
    application's needs (e.g. `storeConfigValue()`), not on register-level details. Don't reveal
    hardware details to HAL users.
  - **The processor is a detail**: isolate vendor compiler extensions and processor headers
    behind your own interfaces.
  - **The OS is a detail**: put an **OSAL (OS Abstraction Layer)** between the application and
    the RTOS so you can change OS and test off-target.
- Program to **interfaces** and keep implementations **substitutable** (e.g. your own printf/log
  shim).
- **DRY conditional compilation**: don't scatter `#ifdef BOARD_V2` across the code base; hide
  board/target differences behind the HAL and choose the implementation at build/link time.
