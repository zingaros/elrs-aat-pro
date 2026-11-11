# AAT-Pro Work Plan

Version: 0.1

Status: ESP32 commissioning prototype bench-verified; owner/site calibration saved and retained-fit status confirmed after reboot; active tracking disabled; motion safety, calibration validation, and UX redesign pending

Related Documents:

- architecture.md
- requirements.md
- backpack_audit.md

## Closeout Handoff (2026-11-11)

- Repositories are separate: `c:\DEV\Backpack` is the firmware workspace on branch `audit/aat-analysis`; `c:\DEV\elrs-aat-pro` is the planning workspace on `main`.
- Preserve existing work. Backpack contains staged/unstaged firmware changes and is not committed by this planning closeout. Planning documentation and the two ESP32 research images are included with user approval. Check Git status on resume; do not reset either repository.
- Hardware verified: ESP32 DevKit, HMC5883L and OLED on the same I2C bus. An unsoldered wire caused the earlier bus failure; it was repaired, and both devices now work. Firmware pin mapping is SDA GPIO22 / SCL GPIO21; no further wiring changes are needed.
- COM6 runs config-v7 `AAT_ESP32_Backpack_ServoTest` in Wi-Fi-only mode. AP `ExpressLRS AAT Backpack`, password `expresslrs`, address `10.0.0.1`. Pan GPIO27, detached elevation GPIO14, manual active-low reference button GPIO32, HMC/OLED SDA22/SCL21. HMC single-shot configuration 104/32/1 with a 500 ms retry restored changing X/Y; the user confirmed the configuration and "Compass fit saved" after reboot. Pan neutral is 1520 us. Owner fit candidates: 11/12 sectors, offsets (156.0,-63.5), scales (1.11,0.91). Site candidates: 12/12 sectors, offsets (150.5,-58.5); temporary scales (1.14,0.89) were not saved. Owner scales are preserved. Stored offsets are integer-rounded, so candidate values are not exact EEPROM readbacks. Active tracking remains disabled.
- HMC motion verified: before a right 90-degree turn, raw X/Y/Z was (126,-354,-603); after, (-129,-55,-512). Both had data-ready set, lock clear, and config A/B/mode 112/32/0. The uncalibrated sensor-frame X/Y angle changed about -87 degrees. This verifies sensor response, not absolute heading calibration.
- Current firmware code includes a host-testable heading helper (`src/aat_compass.h`), compass sweep fitter (`src/aat_compass_calibration.h`), and bounded pan command math (`src/aat_servo.h`). The dedicated commissioning build adds owner sweep and quick site offset refresh; active tracking remains disabled.
- Validation: 32 native tests pass, including the regression that keeps sweep motion independent of the manual pulse timeout. The standard AAT and commissioning builds compile; `html/scan.js` passes `node --check`. After reboot, the user confirms HMC readback 104/32/1 and "Compass fit saved". This verifies sensor initialization and retained calibration-valid status; individual stored offsets/scales were not independently compared after reboot. Remote CI is still pending.
- From `c:\DEV\Backpack`, rebuild/flash the servo test with `python -m platformio run -e AAT_ESP32_Backpack_ServoTest -t upload --upload-port COM6`. Run native tests with `C:\msys64\ucrt64\bin` on `PATH` using `python -m platformio test -e native-aat-test`.
- Reboot check completed: HMC reads 104/32/1 and the WebUI reports "Compass fit saved". Individual stored coefficients still need an independent reload comparison.
- Next task: define and validate the mechanical pan reference, startup/recovery behavior, and +/-170-degree travel protection. GPIO32 is a manual button, not an automatic home sensor. Keep active tracking disabled.
- The azimuth servo and HMC are mounted; a manual active-low reference button is wired to GPIO32. The user permits up to two full turns in either direction, not unlimited continuous rotation. Neutral is 1520 us, with leftward motion starting at 1480 us, slow rightward motion at 1565 us, about 12 degrees/second at 1450 us, and about 45 degrees/second at 1000/2000 us. Owner and site fits were saved; calibration-valid status survives reboot. Motion-reference and travel protection remain required before active tracking.
- DEC-08 performance baseline/threshold ratification remains explicitly deferred.

## Planning Baseline

- `backpack_audit.md` records the verified Backpack AAT implementation as of 2026-11-11.
- `elrs-aat-pro` is the planning repository. Firmware work intended for upstream contribution belongs in the separate Backpack fork and should be prepared as independent pull requests against ExpressLRS/Backpack.
- Preserve current AAT behavior and stored settings. In particular, do not disable the existing nonzero `project` setting for legacy installs.
- Build work must use existing Backpack conventions. Current AAT environment names are listed in `backpack_audit.md`; no distinct ESP32-S3 AAT environment is verified.
- Phases have no release dates. Do not start implementation that depends on an unresolved decision below.

## Implementation Progress

- Extracted navigation distance, bearing, and elevation calculations into a host-testable helper in the Backpack repository. Corrected westward-bearing normalization and longitude-delta overflow at the antimeridian.
- Added 32 native tests covering navigation, legacy modes/defaults, projection, bounded pan math, compass helpers, flat-axis rejection, site-offset preservation, migration helpers, and manual-pulse timeout behavior.
- Verified builds for `AAT_ESP_Backpack_via_UART`, `AAT_ESP_Backpack_via_WIFI`, and `AAT_ESP32_Backpack_via_UART`.
- All 32 native tests passed locally with MSYS2 UCRT64. Standard/commissioning ESP32 builds passed during development; the latest firmware still needs the full target regression matrix. Remote CI remains pending.
- Added an ESP32-only HMC5883L diagnostic environment. The `/aatcompass` request reads the sensor directly, retains the last valid XYZ sample, and reports sample count/age, data-ready/lock status, configuration registers, and raw X/Z/Y bytes; the diagnostic main loop performs no I2C operations. The image does not attach or update either servo and pauses loop-time OLED refreshes.
- The compass diagnostic now enters Wi-Fi-only mode on boot; serial logs confirmed AP startup and a Windows network scan saw `ExpressLRS AAT Backpack` at strong signal. The AP uses the existing `expresslrs` password and `10.0.0.1` address.
- User bench response from `/aatcompass` confirmed HMC identity `H43`, successful configuration, and an available raw sample: X=-326, Y=85, Z=-465. Sensor orientation for this sample is unknown; heading calibration and magnetic-interference checks remain pending.
- User supplied a four-point rotation sweep at 0, -90, -180, and -270 degrees: (-170,-109,-432), (-41,228,-420), (361,-69,-548), and (259,-392,-563) for X/Y/Z. The sensor responds to rotation, but this set is not calibration-quality: Z varies by 143 counts and the orientation/level and nearby magnetic environment were not recorded.
- Latest diagnostic rotation check succeeded: at the initial position raw X/Y/Z was (126,-354,-603), then after a right 90-degree turn it was (-129,-55,-512). Both samples reported data-ready, not locked, and config A/B/mode of 112/32/0. The uncalibrated sensor-frame X/Y angle changed about -87 degrees; absolute heading calibration remains deferred until the servo assembly is installed.
- User added the OLED to the same I2C bus as the HMC5883L. The configured addresses are distinct (OLED `0x3C`, HMC `0x1E`); both devices work after the unsoldered wire was repaired. The diagnostic endpoint is compass-only and makes no OLED I2C transactions.
- Mounted owner/site sweeps completed and calibration-valid status survived reboot. Prototype thresholds are 36 samples and 10/12 fitted sectors; magnetic-stability validation is not implemented or ratified. Calibration is separate from the manual button and neutral. Tracking remains gated on calibrated feedback, reference/recovery, and +/-170-degree travel protection.
- Documented the CI UART-environment regression matrix and the AAT Wi-Fi exception in `backpack_audit.md`.
- DEC-05/06 initial decisions are recorded: preserve v5 until persistent fields are needed and establish the existing ESP32 DevKit target as initial AAT-Pro while retaining ESP8266 Classic. Tracker-local GPS remains deferred; the HMC5883L is selected for optional pan-heading feedback, with production calibration and control integration still open.
- DEC-08 baseline capture and threshold ratification are intentionally deferred per user direction.
- Added hardware-independent pan command calculation: home-relative compass conversion, +/-170-degree target clamp, linear bounded-sweep error, deterministic shortest-path tie, and safe rejection when unhomed, compass-invalid, or outside limits. Native tests cover wrap, clamp, boundaries, and unavailable state. The HMC diagnostic driver is verified; calibrated heading is not yet wired to AAT tracking, and continuous-pan actuator output remains disabled.
- Added a host-testable calibrated heading helper and local v7 persistent calibration. Commissioning live heading/rate still use raw sensor values; saved correction and mounting/north alignment are not integrated into tracking.
- DEC-03 policy is partially resolved: rotating HMC5883L with SM-S4306R and intended +/-170-degree travel. GPIO32 is a manual button, not automatic homing. Neutral and bench speed points are measured; reference/recovery, calibrated control, loaded speed, and travel protection remain open. Positional-servo support requires a positional actuator; do not drive this continuous servo using positional mapping on compass failure.
- Non-performance Phase 0 follow-up: observe the dedicated native-test CI job and verify the CI target matrix on a remote run.

## Resume Checklist

- COM6 is flashed with the v7 commissioning wizard. Open the AAT WebUI, press and hold the manual reference button, start the owner sweep, release it to rotate, then press it again at the same visual mark. Review the fit before saving. The quick site sweep is available after a valid owner fit and updates only local offsets. Tracking remains disabled in this build. Restore the compass diagnostic with `python -m platformio run -e AAT_ESP32_Backpack_Compass -t upload --upload-port COM6` when sensor diagnostics are needed.
- Current bench status: the OLED and mounted HMC share the working I2C bus. The single-shot commissioning reader initializes as 104/32/1 and responds to hand rotation. Owner and site calibration completed; after reboot the WebUI still reports "Compass fit saved". Active tracking remains disabled.
- Phase 1 profile decision: reuse the existing ESP32 DevKit AAT target; preserve ESP8266 Classic. Pan neutral and compass fit use config v7 with tested v5/v6 migrations.
- DEC-08 is deferred. When resumed, investigate why measured servo intervals include 3.468 ms despite the 10 ms update gate, add percentile aggregation, then run matched 60-second active GPS/home captures with WebUI idle and 10 read-only `/config` requests/sec on ESP8266 and ESP32.
- Before tracker-local GPS/home work, resolve DEC-01, DEC-02, and DEC-09. Resolve DEC-03 before continuous pan, DEC-04/10 before safety actions, DEC-06 before sensor drivers, and DEC-07 before MAVLink source selection.
- The 32 native tests passed locally; observe the dedicated CI test job on a remote workflow run.

## Next Session Priorities

- Keep COM6 on the commissioning image and tracking disabled. No new flash or sweep is needed just to resume. Read the commissioning addendum in `backpack_audit.md` for the local-versus-baseline distinction.
- Review sweep safety first: neutral while waiting for button release, stopping before fitting/EEPROM writes, cancellation/timeout, stale compass rejection, and callback/main-loop ownership. Verify with focused state-machine tests before another actuated run.
- Compare exact stored v7 offsets/scales after reload and add byte-accurate v5/v6 fixtures. Current helper tests do not prove the full EEPROM layout or preservation of every legacy field.
- Validate magnetic stability, outliers/overflow, and full-turn evidence independently of fitted-sector coverage; a min/max fit can make a partial arc appear covered. Keep raw measurements distinct from calibrated bearing/rate.
- Define mechanical zero, manual-reference policy, startup/recovery, and +/-170-degree travel handling. GPIO32 does not provide automatic homing; the bench allows at most two turns, not unlimited motion.
- Rework calibration UX and consistency with current Backpack options (Phase 6). Owner/site operations currently both make a full turn; a shorter startup sweep and a complete first-owner servo/tilt setup sequence are not implemented.
- Preserve deferred DEC-08 work and pending remote CI/full-target builds. Do not widen into GPS, prediction, PID, or MAVLink before the commissioning gates are resolved.

## Completed Foundation

- [x] Record current AAT build, runtime, configuration, and WebUI behavior in `backpack_audit.md`.
- [x] Align architecture and requirements with the verified baseline; distinguish current behavior from proposed capabilities.
- [x] Establish legacy behavior and config preservation as explicit upgrade requirements.

## Decision Gates

Resolve each item before the dependent phase. Record the decision in the architecture and requirements before coding.

| ID | Decision required | Blocks |
| --- | --- | --- |
| DEC-01 | Home-source precedence among manual, CRSF aircraft telemetry, tracker-local GPS, and future MAVLink; define when each source may set or replace home. | GPS and navigation |
| DEC-02 | Tracker-GPS stability metric, including radius, coordinate/altitude datum, and handling of fix-quality interruptions during the 30-second qualification period. | Tracker-GPS home |
| DEC-03 | SM-S4306R with rotating HMC5883L, intended +/-170-degree travel, and owner/site calibration are selected. GPIO32 is manual, not an automatic home sensor. Define mechanical-reference policy, startup/recovery, robust field/coverage validation, calibrated rate control, and travel protection. Stop the continuous actuator on invalid feedback; positional support requires a positional actuator. | Continuous pan tracking |
| DEC-04 | Telemetry-loss state machine: timer source, relationship between the 3-second event and prediction grace interval, selected Hold/Park/Return Home actions, and recovery transition. | Watchdog and prediction |
| DEC-05 | Local config is v7 with v5/v6 migration helpers. Preserve legacy values including `project=0xff`; initialize only new fields. Byte-accurate fixtures and coefficient reload tests remain required before upstreaming. | Future persistent config additions |
| DEC-06 | Existing ESP32 DevKit is the first profile; retain ESP8266 Classic and CRSF aircraft GPS. HMC5883L commissioning uses SDA22/SCL21 with OLED on the shared bus. Production calibrated-heading integration remains pending; tracker-local GPS is deferred. | Future local sensor drivers/targets |
| DEC-07 | MAVLink transport, framing, source selection, and conflict resolution with CRSF telemetry. | MAVLink input |
| DEC-08 | Proposed: measure AatModule::Loop period/runtime and servo-write intervals; enforce 100 ms maximum loop gap, target p95 loop period <=50 ms and p95 servo interval <=20 ms; report max-minus-min jitter. Under 10 read-only `/config` requests/sec, retain hard limits and keep p95 loop period/runtime regression <=10%. Validate on each supported target before ratification. | Performance acceptance and release gates |
| DEC-09 | GPS jump-rejection threshold, detection inputs, and recovery behavior. | GPS and navigation |
| DEC-10 | Zenith-protection boundary and fallback behavior when bearing becomes unstable. | Safety and navigation |
| DEC-11 | Prediction-quality metric, evaluation window, and degradation threshold. | Adaptive prediction and automatic fallback |

## Phased Work

### Phase 0: Baseline Regression Protection

Requirements: REQ-PROD-002, REQ-COMPAT-001 through REQ-COMPAT-006, REQ-PERF-001 through REQ-PERF-004.

- [x] Build the current AAT environments and record toolchain/commands in `backpack_audit.md`.
- [x] Add focused tests for current navigation math, servo-mode mapping, projection limits, endpoint validation, and legacy defaults.
- [x] Capture current behavior before refactoring, including the default `project=0xff` value.
- [x] Document the existing CI UART regression-build matrix and AAT Wi-Fi build exception.

Exit criteria: existing AAT environments compile; baseline tests pass; unrelated Backpack targets have a documented regression-build matrix. Performance measurements and threshold ratification are deferred under DEC-08 and do not block non-timing-dependent implementation.

### Phase 1: Backpack Target and Configuration Foundation

Requirements: REQ-CONST-001 through REQ-CONST-005, REQ-TARGET-001 through REQ-TARGET-004, REQ-CFG-001 through REQ-CFG-003, REQ-UPSTREAM-001 through REQ-UPSTREAM-004.

Dependencies: Phase 0 baseline build/tests; DEC-05 and DEC-06 initial decisions are recorded. DEC-08 measurement/ratification is deferred at the user's request; do not make timing-dependent behavior changes. The existing ESP32 DevKit target already supplies the initial profile.

- [x] Reuse the existing ESP32 DevKit AAT environment as the initial AAT-Pro profile; leave the ESP8266 AAT Classic environment unchanged.
- Keep existing AAT target definitions unchanged unless an independently justified Backpack change is accepted.
- Add persistent fields only when required; follow the recorded v5 field-by-field migration policy and test against version 5 data.
- Separate compile-time hardware availability from runtime detection and user enablement without introducing a second target registry.

Exit criteria: current targets still build; new target builds are explicit and repeatable; migration tests prove old values survive; feature-off builds retain baseline behavior.

### Phase 2: GPS Sources, Home, and Navigation

Requirements: REQ-GPS-001 through REQ-GPS-006, REQ-NAV-001 through REQ-NAV-003.

Dependencies: Phase 1; DEC-01, DEC-02, and DEC-09.

- Define and implement tracker-GPS input and health reporting for selected hardware.
- Implement the agreed home-source precedence and tracker-GPS qualification rules.
- Add jump rejection using DEC-09 and tests for invalid fixes, interrupted qualification, source failure, coordinate wrap, and relative altitude.

Exit criteria: acceptance tests cover each home source and fallback; invalid or stale tracker GPS cannot replace a valid home; baseline CRSF telemetry-home behavior remains unchanged.

### Phase 3: Heading and Compass

Requirements: REQ-COMP-001 through REQ-COMP-005.

Dependencies: Phase 1; DEC-06.

- Implement only the sensor drivers selected in DEC-06.
- Add calibration persistence, heading validity checks, and manual-heading fallback.

Exit criteria: sensor absence or failure falls back without stopping basic tracking; heading and calibration tests pass on each selected hardware profile.

### Phase 4: Pan/Tilt Motion and Filtering

Requirements: REQ-PAN-001 through REQ-PAN-006, REQ-FLT-001 through REQ-FLT-006, REQ-PID-001 through REQ-PID-003.

Dependencies: Phase 0 functional regression checks and DEC-03 for actuator-facing continuous-pan behavior. DEC-08 performance measurement is deferred; it remains necessary before ratifying timing-dependent limits or release performance claims. Compass integration is required only for modes that need absolute heading.

- [x] Implement and host-test hardware-independent shortest-path and bounded pan command math; actuator output remains disabled until compass feedback, homing, +/-170-degree travel handling, startup/recovery behavior, servo calibration, and mechanical limits are defined in DEC-03.
- Add bounded rate/acceleration control, filtering, deadband, and tilt easing as independent features.
- Add optional pan and tilt PID only after open-loop motion limits are validated.

Exit criteria: wrap-boundary tests prove shortest-path selection; unknown orientation cannot command active continuous tracking; commanded speed/acceleration stays within configured limits; PID remains disabled by default.

### Phase 5: Safety and Diagnostics

Requirements: REQ-TLM-004 through REQ-TLM-006, REQ-SAFE-001 through REQ-SAFE-006, REQ-DIAG-001 through REQ-DIAG-005.

Dependencies: Phases 2 and 4; DEC-04, DEC-08, and DEC-10.

- Implement telemetry freshness tracking and the agreed loss/recovery state machine.
- Add zenith handling and sensor/servo health reporting supported by selected hardware.
- Make any extrapolation during an outage bounded and subordinate to the selected safety action.

Exit criteria: timing tests exercise loss at the specified threshold, recovery, invalid packets, and each configured action; safety transitions do not block telemetry processing or the tracking loop.

### Phase 6: Backpack WebUI Extensions

Requirements: REQ-UI-001 through REQ-UI-005, REQ-WEB-001 through REQ-WEB-010.

Dependencies: Phase 1 and stable data/config interfaces from the relevant feature phases.

- Extend existing Backpack status/configuration routes and navigation; do not create a separate framework.
- Deliver Status, Configuration, Servo, Diagnostics first; add GPS, Compass, and Prediction views only with corresponding runtime data.
- [ ] Rework the calibration UX: the current commissioning panel is a bench prototype, not an acceptable first-owner experience. Provide clear ordered steps, readiness checks, progress, cancellation/neutral controls, failure recovery, and completion/persistence confirmation. Distinguish owner calibration from site alignment and separate live measurements, candidate fits, and saved settings; keep low-level diagnostics secondary.
- [ ] Review overall AAT WebUI consistency against the existing options in the `Backpack` firmware workspace. Align navigation, layout, terminology, units, controls, defaults, validation, and save/apply behavior with current Backpack options. Resolve overlap or ambiguity between legacy positional-servo settings and continuous-pan calibration without changing existing installs' behavior or stored values.
- Exercise reads, writes, validation errors, persistence, and operation without an OLED.

Exit criteria: UI reflects actual capabilities and sensor health, invalid values are rejected safely, settings persist through reboot, and the performance acceptance criteria remain met under WebUI activity.

Calibration UX acceptance: a first-time owner can complete setup and a returning owner can perform site alignment without bench-session guidance; prerequisite and failure states are explicit, saved values are unambiguous, and the workflow is consistent with existing Backpack options on desktop/mobile and without an OLED.

### Phase 7: Filtering, Prediction, and Latency Compensation

Requirements: REQ-PRED-001 through REQ-PRED-005, REQ-LAT-001 through REQ-LAT-005, plus remaining REQ-FLT items.

Dependencies: Phases 0, 2, 4, and 5; DEC-04, DEC-08, and DEC-11.

- Deliver linear prediction before adaptive prediction.
- Define prediction-error metrics and fallback thresholds before enabling adaptive behavior.
- Add manual latency first; add automatic estimation only after measurement and quality criteria are verified.

Exit criteria: prediction is optional and disabled by default for new installs; prediction quality is measured; invalid or degraded prediction returns to baseline tracking; outage extrapolation never exceeds the approved safety interval.

### Phase 8: MAVLink and Simulation

Requirements: REQ-TLM-003 and REQ-PERF-004; simulation is an architecture capability pending a dedicated requirement.

Dependencies: Phase 1; DEC-07. Safety behavior must be complete before MAVLink can be selected as the sole position source.

- Add the agreed MAVLink parser/transport as a separable input path.
- Add deterministic simulation/test inputs for navigation, source loss, and actuator commands.

Exit criteria: CRSF-only builds remain unaffected; source arbitration and loss handling pass tests; simulation does not alter production behavior when disabled.

## Pull Request Rules

- Each phase may contain multiple small PRs, but each PR must build and provide standalone value.
- Keep baseline-preserving refactors separate from feature behavior changes.
- Include tests and update the requirement-to-test mapping with every behavior change.
- Do not combine target registration, sensor drivers, WebUI pages, and prediction into one dependency-heavy PR.
- Build and test the relevant Backpack target matrix before proposing a PR to `ExpressLRS/Backpack`.

## Requirement-to-Verification Map

| Requirement area | Primary verification |
| --- | --- |
| Targets and compatibility | PlatformIO compile matrix; legacy behavior/default regression tests |
| Config migration | Version 5 fixture migration tests; reboot persistence tests |
| GPS and navigation | Unit tests for qualification, source fallback, jump rejection, bearing, distance, and relative altitude |
| Compass and heading | Driver tests, calibration tests, sensor-loss fallback tests |
| Pan/tilt and filtering | Wrap-boundary, endpoint, rate/acceleration, and feature-disabled tests |
| Telemetry loss and safety | Deterministic timer/state-transition tests, including recovery and prediction bounds |
| WebUI | Route/config round-trip tests and timing under concurrent requests |
| Prediction and latency | Recorded/simulated trajectories, prediction-error thresholds, and fallback tests |
| MAVLink | Parser/source arbitration tests and CRSF-only build regression |
