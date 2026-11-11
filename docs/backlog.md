# AAT-Pro Work Plan

Version: 0.1

Status: Proposed; no implementation phases started

Related Documents:

- architecture.md
- requirements.md
- backpack_audit.md

## Planning Baseline

- `backpack_audit.md` records the verified Backpack AAT implementation as of 2026-11-11.
- `elrs-aat-pro` is the planning repository. Firmware work intended for upstream contribution belongs in the separate Backpack fork and should be prepared as independent pull requests against ExpressLRS/Backpack.
- Preserve current AAT behavior and stored settings. In particular, do not disable the existing nonzero `project` setting for legacy installs.
- Build work must use existing Backpack conventions. Current AAT environment names are listed in `backpack_audit.md`; no distinct ESP32-S3 AAT environment is verified.
- Phases have no release dates. Do not start implementation that depends on an unresolved decision below.

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
| DEC-03 | Pan actuator, absolute orientation feedback or homing method, startup/recovery behavior, and mechanical rotation limits. | Continuous 360-degree tracking |
| DEC-04 | Telemetry-loss state machine: timer source, relationship between the 3-second event and prediction grace interval, selected Hold/Park/Return Home actions, and recovery transition. | Watchdog and prediction |
| DEC-05 | Config schema extension and migration from VRX config version 5, including preservation of all existing fields and legacy projection behavior. | Target/config work |
| DEC-06 | Supported GPS and compass modules, bus/UART allocation, pin mapping, and which hardware profiles are first-class. | Hardware drivers and target definitions |
| DEC-07 | MAVLink transport, framing, source selection, and conflict resolution with CRSF telemetry. | MAVLink input |
| DEC-08 | Performance limits for tracking-loop period/jitter and WebUI load on each supported target. | Performance acceptance and release gates |
| DEC-09 | GPS jump-rejection threshold, detection inputs, and recovery behavior. | GPS and navigation |
| DEC-10 | Zenith-protection boundary and fallback behavior when bearing becomes unstable. | Safety and navigation |
| DEC-11 | Prediction-quality metric, evaluation window, and degradation threshold. | Adaptive prediction and automatic fallback |

## Phased Work

### Phase 0: Baseline Regression Protection

Requirements: REQ-PROD-002, REQ-COMPAT-001 through REQ-COMPAT-006, REQ-PERF-001 through REQ-PERF-004.

- Build the current AAT environments from the audit and record toolchain/commands.
- Add focused tests for current navigation math, servo-mode mapping, projection limits, endpoint validation, and legacy defaults where the existing test structure permits.
- Capture current behavior before refactoring, including the default `project=0xff` value.

Exit criteria: existing AAT environments compile; baseline tests pass; unrelated Backpack targets have a documented regression-build matrix; measured performance thresholds from DEC-08 are recorded.

### Phase 1: Backpack Target and Configuration Foundation

Requirements: REQ-CONST-001 through REQ-CONST-005, REQ-TARGET-001 through REQ-TARGET-004, REQ-CFG-001 through REQ-CFG-003, REQ-UPSTREAM-001 through REQ-UPSTREAM-004.

Dependencies: Phase 0; DEC-05 and DEC-06.

- Add or refine target/build definitions using the existing Backpack PlatformIO conventions.
- Keep existing AAT target definitions unchanged unless an independently justified Backpack change is accepted.
- Add any new config fields with a migration path and tests against version 5 data.
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

Dependencies: Phase 0; DEC-03 and DEC-08. Compass integration is required only for modes that need absolute heading.

- Implement continuous-pan behavior only after actuator orientation and homing are defined.
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
- Exercise reads, writes, validation errors, persistence, and operation without an OLED.

Exit criteria: UI reflects actual capabilities and sensor health, invalid values are rejected safely, settings persist through reboot, and the performance acceptance criteria remain met under WebUI activity.

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
