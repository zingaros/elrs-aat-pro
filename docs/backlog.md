# AAT-Pro Backlog

## Product Scope

- Preserve the existing ExpressLRS Backpack CRSF aircraft-GPS/home path, configuration, legacy modes and stored settings.
- Add continuous-rotation pan as an opt-in option on supported existing Backpack hardware, not as a separate Pro product.
- Reuse Backpack navigation, telemetry, configuration, WebUI, build conventions, positional modes and elevation behavior.
- ESP32-S3, MAVLink, new prediction systems and other proposed Pro features are not active requirements.
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

### Simplify continuous pan as a Backpack option

The current prototype selects continuous pan with mode 3 and latches that selection at boot. The reboot requirement follows from that implementation, not from continuous pan itself. A mode distinction remains necessary because positional and continuous servos interpret the same pulse differently.

- [ ] **Inspect the firmware baseline:** Locate the Backpack prototype source and review mode initialization, servo attachment and output, Wi-Fi lifecycle, configuration, safety guards, commissioning and tests. This checkout contains documentation and trace archives, not the firmware source; firmware changes and verification require access to that source.
- [x] **Narrow the active documentation:** Update the README, requirements and architecture to describe the existing Backpack plus the optional continuous-pan feature, distinguishing current behavior from the target design and bounded routing from unrestricted shortest-path tracking. Historical evidence remains available.
- [ ] **Support safe mode changes:** Reuse the existing mode field and Backpack conventions where practical. Inhibit pan output and clear stale controller state when switching between positional and continuous modes; require explicit confirmation that the correct actuator is installed before enabling output. Entering continuous mode also requires valid calibration, reference, freshness and safety gates. Preserve legacy settings, GPS and elevation behavior. Investigate whether the existing Wi-Fi configuration workflow independently requires a reboot; do not claim a completely reboot-free workflow until verified.
- [ ] **Offer travel policies:** Keep the home-relative +/-170-degree clamp and guarded return through home as the default. Add an explicitly selected unrestricted policy only after the owner confirms the hardware can safely rotate indefinitely, such as with a slip ring or no winding cables. Use shortest angular paths with a deterministic 180-degree tie in unrestricted mode. Inhibit output and revalidate setup when the policy changes; retain valid heading, reference, GPS freshness, neutral and output-envelope safeguards for both policies.
- [ ] **Integrate and verify:** Use the existing WebUI and configuration storage, validate inputs and migrate settings without resetting legacy values. Add focused tests for legacy modes and migration, transitions and re-arming, stale/fault recovery, bounded clamp and rear wrap, unrestricted 359/1 wrap and 180-degree ties, and policy changes. Build existing AAT targets and relevant continuous-pan targets; check WebUI/API and Wi-Fi behavior. Verify physical direction, neutral/fault handling and both policies only with owner approval, suitable hardware and a physical cutoff.

Compass heading cannot count cable turns or detect missed whole revolutions; unrestricted mode is unsafe with limited cable travel. A software neutral request or detached output does not prove a physical stop. Preserve existing commissioning and safety protections unless a simpler approach is verified to be equivalent.

### Later minor release

- [ ] Improve transient compass-spike handling, including its effect on the raw safety guard. Capture every compass sample with timing and status, then evaluate candidate filtering against genuine limit crossings in simulation. Do not mask real travel violations; no change is implemented yet.
- [ ] Refine diagnostics, calibration progress, deadband/filter tuning and settling from field evidence. Characterize pulse-to-rate behavior under representative load before claiming an angular-speed limit.

### Parked or excluded

- Performance threshold ratification remains deferred.
- Prediction, MAVLink input, user-facing simulation, additional GPS processing, acceleration shaping and PID are not active requirements. Tracker-local GPS, tracker-GPS home acquisition and GPS source arbitration are out of scope.

## Verification and Safety

- Prefer native tests, deterministic simulations, fault injection, WebUI checks and firmware builds before hardware testing. Software completion does not prove physical direction, travel, PWM or load-dependent speed.
- Obtain explicit owner approval for powered tests and retain the physical cutoff, freshness gates and neutral-on-invalid behavior. The owner-confirmed cable clearance above need not be rechecked as a routine question.
- The original unexpected-motion event is historical and non-reproduced. Compass spikes remain a documented limitation.

## Decision Register

| ID | Status |
| --- | --- |
| DEC-01 | Closed: use existing Backpack CRSF aircraft-GPS/home behavior. |
| DEC-02 | Dropped: tracker-local GPS and its home-acquisition workflow. |
| DEC-03 | Keep the bounded home-relative route and +/-170-degree clamp as the default; plan an explicitly selected unrestricted policy for suitable hardware. |
| DEC-04 | Parked: additional telemetry-loss Hold/Park/Return-Home policies. |
| DEC-05 | Preserve legacy configuration values, including `project=0xff`, and test migrations. |
| DEC-06 | Closed for the current profile: retain the ESP32 DevKit AAT and ESP8266 Classic, with compass feedback on the supported ESP32 profile. |
| DEC-07 | Out of active scope: MAVLink transport and arbitration. |
| DEC-08 | Parked: runtime and servo-write timing measurements and threshold ratification. |
| DEC-09 | Out of current scope: new GPS jump rejection or recovery behavior. |
| DEC-10 | Parked: zenith-protection boundary and fallback. |
| DEC-11 | Out of active scope: prediction-quality metric and degradation threshold. |

## Contribution and Verification

- Keep changes independently buildable and baseline-preserving; include tests and update directly related requirements and documentation.
- Preserve configuration migration, CRSF navigation, compass validity, bounded pan, safety/recovery and WebUI behavior in their respective tests. Do not claim deferred performance thresholds as measured.

## Supporting Documents

- [Test and engineering history](backlog-history.md)
- [Architecture](architecture.md)
- [Requirements](requirements.md)
- [Backpack audit](backpack_audit.md)
