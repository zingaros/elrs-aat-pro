# AAT-Pro Backlog

## Product Scope

- Preserve the existing ExpressLRS Backpack CRSF aircraft-GPS/home path, configuration, legacy modes and stored settings.
- Keep AAT-Pro additions modular and default-off.
- Tracker-local GPS, tracker-GPS home acquisition and GPS source arbitration are out of scope.

## Current Status

- **Primary feature:** opt-in continuous-rotation pan with compass feedback, mechanical-home confirmation, a bounded +/-170-degree target and neutral on invalid feedback.
- **Completed checks:** 106 native tests passed; the standard ESP32 AAT, CompassPan, CompassPan Trace and ServoTest variants built. Earlier firmware passed powered directional tracking, the +170-degree clamp and a 360-degree simulated GPS orbit. With the aircraft and remote connected, a live-GPS trace check passed after home re-reference; it did not verify physical motion.
- **Latest bench state:** The CompassPan Trace image with OLED and elevation changes is flashed on COM6. After the owner re-confirmed home, passive trace showed valid reference, fresh GPS and compass, and a neutral request near the target; current servo-power state was not confirmed. The new OLED and elevation behavior has not been hardware-verified. The owner verified cable clearance up to two turns in either direction for this setup.
- **Known limitation:** the three-sample median filters tracking commands only; the raw safety guard can still react to a spike. The cause and duration of the observed spikes remain unknown.

Detailed test results, investigations and earlier plans are retained in [test and engineering history](backlog-history.md).

## Open Work

### Before release

- [x] Restore positional pan initialization in CompassPan firmware and add the Backpack GPS OLED view and positional elevation tracking to mode 3. Software checks pass; the new behavior is not hardware-verified.
- [x] Show mode-relevant settings in the WebUI while keeping active commissioning controls available.

### Later minor release

- [ ] Improve transient compass-spike handling, including its effect on the raw safety guard. Capture every compass sample with timing and status, then evaluate candidate filtering against genuine limit crossings in simulation. Do not mask real travel violations; no change is implemented yet.
- [ ] Refine diagnostics, calibration progress, deadband/filter tuning and settling from field evidence. Characterize pulse-to-rate behavior under representative load before claiming an angular-speed limit.
- [ ] Consider acceleration shaping, tilt easing and optional PID only when justified; PID and acceleration remain disabled by default.

### Parked or excluded

- Performance threshold ratification remains deferred.
- Prediction, MAVLink input, user-facing simulation and additional GPS processing are parked. Tracker-local GPS, tracker-GPS home acquisition and GPS source arbitration are out of scope.

## Verification and Safety

- Prefer native tests, deterministic simulations, fault injection, WebUI checks and firmware builds before hardware testing. Software completion does not prove physical direction, travel, PWM or load-dependent speed.
- Obtain explicit owner approval for powered tests and retain the physical cutoff, freshness gates and neutral-on-invalid behavior. The owner-confirmed cable clearance above need not be rechecked as a routine question.
- The original unexpected-motion event is historical and non-reproduced. Compass spikes remain a documented limitation.

## Decision Register

| ID | Status |
| --- | --- |
| DEC-01 | Closed: use existing Backpack CRSF aircraft-GPS/home behavior. |
| DEC-02 | Dropped: tracker-local GPS and its home-acquisition workflow. |
| DEC-03 | Use the bounded home-relative route and +/-170-degree target clamp. |
| DEC-04 | Parked: additional telemetry-loss Hold/Park/Return-Home policies. |
| DEC-05 | Preserve legacy configuration values, including `project=0xff`, and test migrations. |
| DEC-06 | Closed for the current profile: retain the ESP32 DevKit AAT and ESP8266 Classic, with compass feedback on the supported ESP32 profile. |
| DEC-07 | Parked: MAVLink transport and arbitration. |
| DEC-08 | Parked: runtime and servo-write timing measurements and threshold ratification. |
| DEC-09 | Out of current scope: new GPS jump rejection or recovery behavior. |
| DEC-10 | Parked: zenith-protection boundary and fallback. |
| DEC-11 | Parked: prediction-quality metric and degradation threshold. |

## Contribution and Verification

- Keep changes independently buildable and baseline-preserving; include tests and update directly related requirements and documentation.
- Preserve configuration migration, CRSF navigation, compass validity, bounded pan, safety/recovery and WebUI behavior in their respective tests. Do not claim deferred performance thresholds as measured.

## Supporting Documents

- [Test and engineering history](backlog-history.md)
- [Architecture](architecture.md)
- [Requirements](requirements.md)
- [Backpack audit](backpack_audit.md)
