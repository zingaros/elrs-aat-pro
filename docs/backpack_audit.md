# Backpack AAT Implementation Audit

Audit date: 2026-11-11

## Scope

This document records behavior verified in the `Backpack` repository clone in this workspace on the audit date. It focuses on the current AAT target and its direct runtime, configuration, and WebUI paths. It is not a claim that the listed features are absent from every other Backpack target or module.

The audit distinguishes the original positional-servo baseline from proposed AAT-Pro capabilities. Baseline sections below are historical; the addendum records subsequent local commissioning changes, not an upstream release. Source paths below are relative to the sibling `Backpack` repository.

## Commissioning Addendum (2026-11-11)

- COM6 runs `AAT_ESP32_Backpack_ServoTest`, an ESP32 Wi-Fi-only commissioning build. AP: `ExpressLRS AAT Backpack`, password `expresslrs`, address `10.0.0.1`. Pan signal GPIO27; elevation GPIO14 remains detached. HMC/OLED share SDA22/SCL21. GPIO32 is a manual active-low reference button connected to ground, not an automatic position sensor.
- `/aatservo` exposes bounded 1000-2000 us pulse commands, raw compass/status telemetry, operator-marked owner calibration, and site-offset refresh. Manual commands expire to neutral after 3 seconds without keepalive. Sweeps have a 14-second timeout and sensor-loss/manual-stop exits. Tracking is bypassed; no sweep starts on boot. Neutral-before-release behavior and stopping before fit/EEPROM work still need safety verification.
- Continuous HMC reads became static with persistent LOCK. Single-shot configuration 104/32/1 plus a 500 ms conversion retry restored motion response; the user confirmed that readback after reboot. The cause of persistent LOCK is not established by this workaround. A successful read counter alone does not prove a new conversion.
- Pan neutral is 1520 us; leftward motion starts at 1480 us and slow rightward motion at 1565 us. Bench rates: about 12 deg/s at 1450 us and 45 deg/s at 1000/2000 us. Load/supply conditions were not recorded; these are not production speed limits.
- Owner fit: 11/12 sectors, candidate offsets (156.0,-63.5), scales (1.11,0.91). Site fit: 12/12 sectors, candidate offsets (150.5,-58.5); candidate scales (1.14,0.89) are not persisted. Site refresh preserves owner scales. Stored offsets are rounded to integers and scales to permille, so displayed candidates are not exact EEPROM values.
- Local VRX config is v7 with v5/v6 migrations and new neutral/compass fields. After reboot the user confirmed "Compass fit saved", not each coefficient independently. 32 native tests passed; standard ESP32 AAT and commissioning builds and JavaScript syntax were checked during development. Final all-target builds, serialized migration tests, and remote CI remain pending.
- The fitter is an axis-aligned min/max correction with provisional sample/sector gates, not a validated general ellipse fit. Field stability, outlier/overflow rejection, asynchronous state consistency, shorter startup sweeps, calibrated feedback, mechanical reference, and travel protection remain outstanding. Live heading/rate are raw sensor-frame diagnostics, not calibrated north or guaranteed angular speed.
- The bench allows up to two turns in either direction, not unlimited rotation. Do not restore positional firmware onto the powered continuous-pan actuator. Calibration UX and consistency with existing Backpack options require redesign; see Phase 6 in `backlog.md`.

## Build and Target Integration

The AAT target definitions are in `targets/aat.ini`:

| Environment | Platform base | Interface | Notes |
| --- | --- | --- | --- |
| `AAT_ESP_Backpack_via_UART` | `env_common_esp12e` | UART | ESP8266-class build; defines `TARGET_VRX_BACKPACK` and `AAT_BACKPACK`; assigns servo, button, LED, and OLED pins. |
| `AAT_ESP_Backpack_via_WIFI` | Extends `AAT_ESP_Backpack_via_UART` | Wi-Fi | Reuses the ESP8266 AAT build settings. |
| `AAT_ESP32_Backpack_via_UART` | `env_common_esp32` | UART | Defines the same target/AAT macros with ESP32-specific pins and the ESP32 servo library. |

The target-specific source selection excludes other Backpack application entry points. Servo and OLED support are compiled conditionally from pin macros. No distinct ESP32-S3 AAT environment is defined in `targets/aat.ini`; this observation does not establish whether an ESP32-S3 build might work through another environment.

### Baseline Timing Structure

`Vrx_main::loop()` samples `millis()`, updates devices, and calls `AatModule::Loop()` once per Arduino loop iteration. There is no fixed active tracking-loop schedule. `AatModule::Loop()` processes GPS, samples the battery, updates the servo when eligible, and processes CRSF. `servoUpdate()` limits servo writes to no more than one per 10 milliseconds; before home is set, the AAT loop delays for 20 milliseconds. These are distinct timing points and do not establish achieved loop period or jitter.

The architecture's 10 Hz minimum and 20 Hz target do not yet specify whether they refer to main-loop cadence, completed tracking calculations, or servo updates. DEC-08 must define the timing points and jitter statistic before hardware measurements can be treated as acceptance results. Measurements must include idle WebUI and concurrent `/aatconfig` requests on each supported target.

An opt-in `AAT_ESP32_Backpack_Perf` environment records AAT loop period/runtime, servo-write intervals, ESP-NOW/MSP receipt, MAC acceptance/rejection, queue drops, CRSF telemetry, GPS/home, bind state, and marker state; `/aatperf?reset=1` clears counters and `/aatperf` returns a JSON snapshot. The diagnostic image was built and uploaded to the ESP32 on COM6 with user approval. It preserves the normal AAT running/binding state, initializes ESP-NOW, and starts the `ExpressLRS AAT Backpack` AP in AP+STA mode at boot; the AP password is `expresslrs` and its address is `10.0.0.1`. The device remains on this diagnostic build. Standard AAT firmware does not start Wi-Fi automatically; it requires the GPIO25 button when wired, or the VRX Backpack Wi-Fi-mode MSP command.

The current diagnostic build does not contain a compiled UID. If no group address is stored, this diagnostic enters binding immediately and `/aatperf` reports `binding: true`; it accepts `MSP_ELRS_BIND`, stores the TX group address, and reboots into normal ESP-NOW operation. This immediate behavior is diagnostic-only; standard firmware retains its normal autobind flow. The diagnostic also suppresses the normal 120-second bind-expiry reboot so it remains available during testing; standard firmware retains that timeout. A shared `MY_BINDING_PHRASE` on both builds is an alternative; keep the phrase local and do not put it in logs or chat. TX Backpack telemetry mode must not be `OFF` for GPS frames to reach AAT.

Serial capture showed repeated `Binding expired` and `Rebooting into wifi update mode` messages on the earlier diagnostic build. The current diagnostic suppresses that timeout and persists a bound marker. On the latest T20 bind attempt, serial output showed `Processing Binding Packet...` with group address `48:F4:75:E4:3E:57`; after reboot `/aatperf` reported `binding: false`, `bound_marker: true`, and `connection_state: running`. T20 pairing is confirmed.

The first reported sample had 2,765 loop intervals, 23,309 us minimum and 316,228 us maximum period, a 292,919 us min/max range, 26,050 us maximum loop duration, and zero servo updates. It was captured with the static configuration WebUI open under minimal load; no ELRS/GPS data or servo output was present. This is a pre-home idle sample, not active tracking or concurrent WebUI-request data, and must not set DEC-08 limits. Capture separate 60-second active-tracking windows with the WebUI idle and under request load, without manual bearing/elevation overrides.

An active-tracking sample was subsequently captured with the T20 in ELRS MAVLink link mode. GPS and home were active. The sample contained 22,108 loop intervals, 20 us minimum and 61,200 us maximum loop period (61,180 us range), 25,038 us maximum loop duration, 2,737 servo updates, and servo intervals of 3,468 to 21,148 us (17,680 us range). The receive path recorded 542 ESP-NOW datagrams/MSP packets, all 542 MAC-accepted with zero rejects or queue drops; all 542 were CRSF telemetry packets, including 53 GPS frames. WebUI load condition and capture duration were not recorded. The 3,468 us servo minimum is below the nominal 10 ms servo-update gate and must be investigated before these figures are used as acceptance thresholds.

A second, larger active-tracking sample in the same reported ELRS MAVLink link mode contained 110,523 loop intervals, 20 us minimum and 61,200 us maximum loop period (61,180 us range), 25,038 us maximum loop duration, 13,878 servo updates, and servo intervals of 3,468 to 23,285 us (19,817 us range). It recorded 2,842 ESP-NOW datagrams/MSP packets, all MAC-accepted with zero rejects or queue drops; all 2,842 were CRSF telemetry packets, including 273 GPS frames. GPS and home were active. Capture duration and WebUI load condition were not provided. The repeated 3,468 us servo minimum remains below the nominal 10 ms gate and requires instrumentation review; do not set DEC-08 limits from this sample yet.

### Verified Baseline Build

Verified on 2026-11-11 with PlatformIO Core 6.2.0. The AAT environments use Espressif platform packages `espressif8266@4.2.1` and `espressif32@6.7.0`, as declared by the target/common configuration.

From the Backpack repository root, build each current AAT environment with:

```text
python -m platformio run -e AAT_ESP_Backpack_via_UART
python -m platformio run -e AAT_ESP_Backpack_via_WIFI
python -m platformio run -e AAT_ESP32_Backpack_via_UART
```

All three builds completed successfully. The native regression suite was also run with `python -m platformio test -e native-aat-test`; all 12 navigation, servo-mode, projection-limit, endpoint-validation, and legacy-default tests passed using MSYS2 UCRT64 GCC/G++ 16.1.0. On Windows, ensure `C:\msys64\ucrt64\bin` is on `PATH` when running the native test environment.

The Backpack GitHub Actions workflow runs the target validator, builds every PlatformIO environment selected from `targets/*.ini` whose name contains `UART`, and runs the native AAT suite in a dedicated job. The AAT Wi-Fi environment does not match the workflow's UART selector; it was built explicitly using the command above. The new native-test CI job has not yet been observed on a remote run.

## Runtime Behavior

### Telemetry and GPS

`CrsfModuleBase` assembles incoming CRSF frames, checks declared frame lengths, and validates CRC before dispatch. `AatModule::onCrsfPacketIn` forwards CRSF GPS frames to `SendGpsTelemetry`; the AAT tracking path does not consume MAVLink packets.

The GPS packet handler stores latitude, longitude, speed, heading, altitude, satellite count, and an update flag. The tracking calculations use latitude, longitude, altitude, and satellite count; stored speed and heading are not used by the current tracking calculations.

The AAT receiver obtains Backpack messages over ESP-NOW. The TX Backpack forwards CRSF telemetry frames when its Backpack telemetry mode is not `OFF`. The receiver accepts packets only from its configured UID/group MAC, except while in binding mode; a shared `MY_BINDING_PHRASE` generates the matching UID during build configuration. Wi-Fi is only the configuration/update interface, not the radio-to-tracker link. The current diagnostic build supports ESP-NOW and AP+STA concurrently; its receive counters can distinguish no packets, MAC rejects, queue drops, telemetry frames, and GPS frames.

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
