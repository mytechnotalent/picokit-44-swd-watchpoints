![picokit-44-swd-watchpoints](https://raw.githubusercontent.com/mytechnotalent/picokit-44-swd-watchpoints/main/picokit-44-swd-watchpoints.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-44 SWD WATCHPOINTS

### Debug Probe Data Watchpoint on a Status Variable
#### Lesson 44 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

<br>

The forty-fourth Picokit lesson. The node cycles a status variable
once per step interval and seals it into an authenticated heartbeat. On top of
the normal lesson, the Debug Probe watchpoint lab sets a data watchpoint on the
status variable and halts whenever it changes.

<br>

## What it teaches

<br>

- A paced status state machine with a one-shot onboard blink.
- Sealing the status into an authenticated LoRa heartbeat.
- Setting a hardware data watchpoint on a global variable.
- Catching every write to a variable over SWD.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| Red / Yellow / Green | GP16 / GP18 / GP17 | annunciator status |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

<br>

The node runs `monitor_step` in a loop. Every 2 seconds it changes
`g_status`, and every 5 seconds it seals `{"n":44,"s":<seq>,"w":<status>}` with
the shared field key and sends it over LoRa.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_44_swd_watchpoints verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-44 SWD WATCHPOINTS // STATUS VARIABLE + DEBUG LAB ===
WATCHPOINT n=44 status=1 seq=1
WATCHPOINT n=44 status=2 seq=2
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=44 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Debug lab: data watchpoint on a status variable

The Debug Probe supports hardware data watchpoints, so a halt fires whenever
the watched address changes. Build with debug info, start OpenOCD, and attach
GDB:

```bash
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg
arm-none-eabi-gdb build/picokit_44_swd_watchpoints.elf
(gdb) target extended-remote localhost:3333
(gdb) monitor reset halt
(gdb) watch g_status
(gdb) continue
```

Each write to the status variable fires the watchpoint:

```text
Hardware watchpoint 2: g_status
Old value = 0
New value = 1
monitor_status_tick (now_us=...) at src/monitor.c
(gdb) continue
```

The same value is sealed into the heartbeat body
`{"n":44,"s":<seq>,"w":<status>}` and shown on the console as
`WATCHPOINT n=44 status=1 seq=1`.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-45-fault-analysis](https://github.com/mytechnotalent/picokit-45-fault-analysis)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-44-swd-watchpoints/blob/main/LICENSE)
