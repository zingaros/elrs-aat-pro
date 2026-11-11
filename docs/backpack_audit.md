# Backpack AAT Implementation Audit

Audit date: 2026-11-11

## Scope

This document records behavior verified in the `Backpack` repository clone in this workspace on the audit date. It focuses on the current AAT target and its direct runtime, configuration, and WebUI paths. It is not a claim that the listed features are absent from every other Backpack target or module.

The audit distinguishes existing behavior from the proposed AAT-Pro architecture. Source paths below are relative to the sibling `Backpack` repository.

## Build and Target Integration

The AAT target definitions are in `targets/aat.ini`:

| Environment | Platform base | Interface | Notes |
| --- | --- | --- | --- |
| `AAT_ESP_Backpack_via_UART` | `env_common_esp12e` | UART | ESP8266-class build; defines `TARGET_VRX_BACKPACK` and `AAT_BACKPACK`; assigns servo, button, LED, and OLED pins. |
| `AAT_ESP_Backpack_via_WIFI` | Extends `AAT_ESP_Backpack_via_UART` | Wi-Fi | Reuses the ESP8266 AAT build settings. |
| `AAT_ESP32_Backpack_via_UART` | `env_common_esp32` | UART | Defines the same target/AAT macros with ESP32-specific pins and the ESP32 servo library. |

The target-specific source selection excludes other Backpack application entry points. Servo and OLED support are compiled conditionally from pin macros. No distinct ESP32-S3 AAT environment is defined in `targets/aat.ini`; this observation does not establish whether an ESP32-S3 build might work through another environment.

## Runtime Behavior

### Telemetry and GPS

`CrsfModuleBase` assembles incoming CRSF frames, checks declared frame lengths, and validates CRC before dispatch. `AatModule::onCrsfPacketIn` forwards CRSF GPS frames to `SendGpsTelemetry`; the AAT tracking path does not consume MAVLink packets.

The GPS packet handler stores latitude, longitude, speed, heading, altitude, satellite count, and an update flag. The tracking calculations use latitude, longitude, altitude, and satellite count; stored speed and heading are not used by the current tracking calculations.

### Home and Navigation

When no home is set, the first processed GPS update whose satellite count meets `config.GetAatSatelliteHomeMin()` becomes home. The default minimum is 5 satellites. The reviewed AAT path does not additionally require a reported 3D fix, an HDOP threshold, or a stable-position interval, and it does not reject implausible GPS jumps.

For subsequent positions, the module calculates great-circle distance and initial bearing from home to the aircraft. Elevation is calculated from the difference between aircraft and home altitude and is constrained to 0 through 90 degrees. The code uses spherical Earth radius 6,371,000 meters for distance.

### Projection

The existing projection is linear azimuth extrapolation based on observed azimuth change and the average interval between processed GPS updates. It is used when the `project` config value is nonzero, an average update interval and azimuth rate are available, and target distance exceeds 3 meters. Extrapolation time is capped at one average GPS interval, and projected angular speed is limited to 100 degrees per second.

The default config sets `project` to `0xff`, which is nonzero and therefore enables this existing projection when its runtime prerequisites are met. This is narrower than a general prediction engine: the reviewed path does not implement adaptive prediction, latency estimation, or a configurable multi-mode prediction policy.

### Servo Output

Servo output runs after home is set and the initial five-second startup delay has elapsed. `servoUpdate` is gated to at most one update per 10 milliseconds and rejects equal low/high endpoints before mapping positions. This is an update interval, not a guarantee of achieved servo update frequency.

The configured modes are:

- `TwoToOne`: maps the full -180 to +179 degree bearing range to the azimuth endpoints, for a mechanical 2:1 arrangement.
- `Clip180`: clips the bearing to 90 degrees either side of center.
- `Flip180`: folds bearings outside that range and reverses the elevation mapping.

Output is rate-smoothed using the configured servo smoothness value. Optional azimuth fast-flip logic handles large moves, with a two-second minimum interval between flips. The implementation uses positional microsecond servo commands; it does not implement continuous-rotation pan position feedback, configurable angular acceleration, or an absolute pan heading sensor.

### Display and Battery

An SSD1306 OLED is compiled in when OLED pins are defined. It displays startup/connection state, GPS and tracking information, and battery voltage. Display updates during tracking are throttled to a 16 millisecond interval. Battery sampling uses `analogRead(A0)` and the configured scale and offset. These facilities are conditional and are not prerequisites for the core tracking calculation.

The OLED can display an `OLD` indicator after GPS data has been stale for more than three average GPS intervals. In the reviewed tracking loop, that indicator is not a telemetry-loss action: it does not select Hold, Park, or Return Home behavior or stop servo updates.

## Configuration and Persistence

The AAT settings are fields of the packed VRX Backpack configuration structure in `lib/config/config.h`. AAT settings and battery calibration are guarded by `AAT_BACKPACK`. `VrxBackpackConfig` reads and writes the structure through the existing `ELRS_EEPROM` provider; `Commit` writes the structure at offset zero and commits the provider.

The VRX configuration version is 5. On a version mismatch, `Load` calls `SetDefaults`; there is no field-by-field migration in this path. Existing AAT defaults include:

| Setting | Default |
| --- | --- |
| Minimum satellites for home | 5 |
| Projection value | `0xff` |
| Servo smoothing | 5 |
| Azimuth center direction | North (`0`) |
| Azimuth fast flip | Enabled (`1`) |
| Azimuth endpoints | 500 to 2500 microseconds |
| Elevation endpoints | 1000 to 2000 microseconds |
| Battery scale / offset | 292 / -2 |

These are current legacy defaults. Changes to config layout or defaults need to account for existing stored configurations and the reset-on-version-mismatch behavior.

## WebUI Integration

The AAT WebUI uses the existing Backpack WebUI infrastructure. `GetConfiguration` in `lib/WIFI/devWIFI.cpp` calls `WebAatAppendConfig` when `AAT_BACKPACK` is defined. The AAT proxy adds the current AAT and battery settings to the existing JSON configuration response.

`WebAatInit` in `src/devwifi_proxy_aat.cpp` registers `/aatconfig`. Its handler updates servo settings, center direction, endpoints, battery calibration, and minimum satellite count. It also accepts bearing/elevation arguments as live target overrides. A request with `commit=1` calls the existing config `Commit`; other requests only modify in-memory settings. The `project` value is included in the JSON response but the AAT handler does not provide a setter for it.

This is an existing configuration surface, not the set of separate Status, GPS, Compass, Prediction, Servo, and Diagnostics pages proposed for AAT-Pro.

## Proposed Capability Comparison

“Not found” below means not found in the reviewed AAT target/runtime path, not necessarily absent from all Backpack code.

| Proposed capability | Current AAT path |
| --- | --- |
| CRSF aircraft GPS tracking | Implemented through CRSF GPS frames. |
| Local tracker GPS | Not found. |
| Compass input, calibration, or health monitoring | Not found. |
| Home qualification using 3D fix, HDOP, and position stability | Not found; current acquisition uses the configured satellite minimum. |
| GPS jump rejection | Not found. |
| MAVLink aircraft position input | Not found in the AAT module path. |
| Linear prediction | Limited azimuth extrapolation exists as described above. |
| Adaptive prediction and automatic latency compensation | Not found. |
| Configurable telemetry-loss actions | Not found; the OLED stale indicator does not control tracking behavior. |
| Continuous-rotation pan with absolute orientation feedback | Not found; current servo modes map positional servo outputs. |
| PID, servo fault detection, and startup sensor self-tests | Not found. |
| Extended AAT-Pro diagnostics and multi-page WebUI | Not found; existing AAT settings are integrated into the Backpack WebUI. |

## Source Locations

- `targets/aat.ini` - AAT build environments, platform bases, compile flags, and pin assignments.
- `src/module_aat.cpp` and `src/module_aat.h` - GPS handling, home acquisition, navigation, projection, servo output, OLED, and battery sampling.
- `src/module_crsf.cpp` - CRSF frame assembly and CRC validation.
- `src/Vrx_main.cpp` - VRX application selection of the AAT module.
- `lib/config/config.h` and `lib/config/config.cpp` - packed config, version check, defaults, and persistence.
- `lib/WIFI/devWIFI.cpp` and `src/devwifi_proxy_aat.cpp` - JSON config integration and `/aatconfig` handler.
- `html/scan.js` - existing AAT WebUI form and live override requests.
