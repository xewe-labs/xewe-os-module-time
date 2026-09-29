# xewe-os-module-time — network time and automatic timezone detection

XeWe OS module · created 2026-09-15 (split out of xewe-os, where it was developed from 2026-07) · Solo: Max Dokukin · Status: Active (0.1.0)

## Overview

NTP time sync and timezone handling. A module for [XeWe OS](https://github.com/xewe-labs/xewe-os),
built on the [XeWeOS framework](https://github.com/xewe-labs/xewe-library-os). On first boot it
detects the timezone from the public IP address by racing three geo-IP services in parallel, asks
the user to confirm the resulting local time, and stores the offset in NVS; if detection fails it
asks for the offset by hand. On every boot it syncs the clock over SNTP and applies the stored
offset. The Scheduler module uses it as its clock.

## Highlights

- Timezone detection races three HTTP endpoints (ipwho.is, ipwhois.app, ip2location.io) in three FreeRTOS tasks; the first valid answer wins through an atomic compare-and-swap, the others are aborted and joined (`Time::begin_routines_init`, `fetch_tz_task`)
- Waits at most 6 s (30 × 200 ms) for a winner, then falls back to manual entry (`GMT±HH:MM`)
- SNTP with three servers: `pool.ntp.org`, `time.google.com`, `time.cloudflare.com`
- `GMT±HH:MM` offsets are converted to POSIX `TZ` strings (sign inverted) and applied with `setenv`/`tzset`

## How it works

```
first boot:  SNTP init → 3 tasks × GET geo-IP → first valid offset → "Is your time …?" → store tz_gmt_str
             (no answer in 6 s or "no") → prompt GMT±HH:MM → store
later boots: apply stored offset → SNTP sync wait (≤ 50 × 200 ms) → print current time
```

- **`Time` class** (`src/Time/`) — a `xewe::os::Module` with id `time`; requires Wifi, asks for setup on first boot, can be disabled.
- **API for other modules** — `get_current_time()` (`tm`), `get_current_time_str()`, `print_current_time()`.
- **Race context** — `TzRace` holds an abort flag, a claimed flag, the result buffer, a binary "winner" semaphore and a counting "done" semaphore used to join the three workers before the context goes out of scope.

### Commands

**Prefix:** `$time` · requires Wifi · asks for the timezone on first boot · can be disabled

| Command | Description | Sample Usage |
| :--- | :--- | :--- |
| **`set_zone`** | Set the timezone offset. | `$time set_zone GMT-08:00` |
| **`fetch`** | Sync the current time from the network. | `$time fetch` |

### Requirements

| | |
|---|---|
| Modules | [xewe-os-module-wifi](https://github.com/xewe-labs/xewe-os-module-wifi) |
| Libraries | XeWeOS (>=0.1.0) and its dependencies (XeWeUtils, XeWeSerial, XeWeNvs, XeWeCli, ArduinoJson) |
| Boards | ESP32-C3, ESP32-C6, ESP32-S3 (arduino-esp32 3.x) |

Metadata and dependencies are declared in [`module.properties`](module.properties).

### Layout

| Path | |
|---|---|
| `src/Time/` | the module (`Time.h`, `Time.cpp`) |
| `xewe-os-module-time.ino` | validation firmware: framework + required modules + this module |
| `scripts/validate.sh` | assembles and compiles the validation firmware |
| `module.properties` | metadata read by xewe-os `setup.sh` and `validate.sh` |

## Results

| Metric | Value | Baseline / note |
|---|---|---|
| Source | 331 lines (`Time.h` 67, `Time.cpp` 264) | `wc -l` |
| Timezone sources | 3 geo-IP endpoints, 5 s HTTP timeout each | manual `GMT±HH:MM` fallback |
| NTP servers | 3 | ESP-IDF `esp_netif_sntp` |

A module has no measured results; the table lists what it provides.

## Getting started

### Use in XeWe OS

Choose `time` in xewe-os `setup.sh`; Wifi is added automatically.

### Validate

Clone the modules this one requires next to this repo, then:

```bash
scripts/validate.sh                         # compile for c3, c6 and s3
scripts/validate.sh -b c3                   # one board
scripts/validate.sh -b c3 -p /dev/ttyACM0   # compile, upload and run on a board
```

The script copies this module and its required modules into `build/xewe-os-module-time/` and compiles it with
`arduino-cli`. Environment variables:

* `XEWE_MODULES_DIR` - where required module repos are cloned (default: the folder containing this repo)
* `XEWE_LIBRARIES_DIR` - folder with `xewe-library-*` clones to build against instead of installed libraries

### Use in firmware

Copy `src/Time/` (and the required modules' folders) into the sketch's `src/`, then:

```cpp
#include "src/Wifi/Wifi.h"
#include "src/Time/Time.h"

Wifi wifi(os);
Time time_module(os, wifi);
```

Declare required modules before this one. The geo-IP lookups use plain HTTP.

## Documents

- [module.properties](module.properties)
- Firmware: [xewe-os](https://github.com/xewe-labs/xewe-os) · registry: [xewe-os-modules](https://github.com/xewe-labs/xewe-os-modules) · framework: [xewe-library-os](https://github.com/xewe-labs/xewe-library-os)
- License: GPL-3.0. See [LICENSE.txt](LICENSE.txt).
