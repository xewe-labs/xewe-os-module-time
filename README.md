# xewe-os-module-time

NTP time sync and timezone handling. A module for [XeWe OS](https://github.com/xewe-labs/xewe-os), built on the
[XeWeOS framework](https://github.com/xewe-labs/xewe-library-os).

## Commands

**Prefix:** `$time` · requires Wifi · asks for the timezone on first boot · can be disabled

Synchronises the clock over NTP and detects or stores the timezone.

| Command | Description | Sample Usage |
| :--- | :--- | :--- |
| **`set_zone`** | Set the timezone offset. | `$time set_zone GMT-08:00` |
| **`fetch`** | Sync the current time from the network. | `$time fetch` |

## Requirements

| | |
|---|---|
| Modules | [xewe-os-module-wifi](https://github.com/xewe-labs/xewe-os-module-wifi) |
| Libraries | XeWeOS (>=0.1.0) and its dependencies (XeWeUtils, XeWeSerial, XeWeNvs, XeWeCli, ArduinoJson) |
| Boards | ESP32-C3, ESP32-C6, ESP32-S3 (arduino-esp32 3.x) |

Metadata and dependencies are declared in [`module.properties`](module.properties).

## Layout

| Path | |
|---|---|
| `src/Time/` | the module |
| `xewe-os-module-time.ino` | validation firmware: framework + required modules + this module |
| `scripts/validate.sh` | assembles and compiles the validation firmware |

## Validate

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

## Use in firmware

Copy `src/Time/` (and the required modules' folders) into the sketch's `src/`, then:

```cpp
#include "src/Wifi/Wifi.h"
#include "src/Time/Time.h"

Wifi wifi(os);
Time time_module(os, wifi);
```

Declare required modules before this one.

## License

GPL-3.0. See [LICENSE.txt](LICENSE.txt).
