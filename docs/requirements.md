# Requirements Specification
## AAT-Pro
### Advanced ExpressLRS Backpack Antenna Tracker

Version: 0.2 Draft

Status: Requirements Definition

Related Documents:

- architecture.md
- backpack_audit.md
- backlog.md

---

# 1. Purpose

This document defines the functional and non-functional requirements for AAT-Pro.

These are target-state requirements. The verified current Backpack AAT behavior is documented in `backpack_audit.md`; this specification does not imply that proposed features are already implemented.

AAT-Pro is an advanced antenna tracker implemented as an extension of the ExpressLRS Backpack ecosystem.

AAT-Pro shall not become a parallel ecosystem.

The project shall preserve compatibility with existing AAT deployments while enabling more advanced tracking capabilities on ESP32-based hardware.

---

# 2. Project Constraints

## REQ-CONST-001

AAT-Pro shall be implemented as an ExpressLRS Backpack target.

Priority: MUST

---

## REQ-CONST-002

AAT-Pro shall reuse existing Backpack infrastructure whenever possible.

Priority: MUST

Examples:

- WebUI framework
- Target registration
- Flashing workflow
- Configuration storage
- Telemetry forwarding
- Build system

---

## REQ-CONST-003

AAT-Pro shall not introduce a standalone firmware ecosystem.

Priority: MUST

---

## REQ-CONST-004

AAT-Pro shall remain suitable for upstream integration.

Priority: MUST

---

## REQ-CONST-005

All advanced functionality shall remain modular.

Priority: MUST

---

# 3. Product Variants

## REQ-PROD-001

The firmware shall support the following target families:

```text
AAT_CLASSIC

AAT_PRO_ESP32

AAT_PRO_ESP32_S3
```

Priority: MUST

Status: Proposed target families. The current Backpack AAT environments are `AAT_ESP_Backpack_via_UART`, `AAT_ESP_Backpack_via_WIFI`, and `AAT_ESP32_Backpack_via_UART`; no distinct ESP32-S3 AAT environment was found in the reviewed target file.

---

## REQ-PROD-002

AAT Classic functionality shall remain unchanged.

Priority: MUST

---

## REQ-PROD-003

AAT-Pro functionality shall be additive.

Priority: MUST

---

## REQ-PROD-004

New AAT-Pro-only features shall be disabled by default for new configurations. This requirement shall not disable legacy AAT behavior or overwrite existing configuration values.

Priority: MUST

---

## REQ-PROD-005

Users upgrading existing AAT installations shall experience no behavior changes unless they explicitly enable a new AAT-Pro-only feature. Existing settings and behavior, including the current nonzero legacy `project` setting, shall be preserved.

Priority: MUST

---

# 4. Capability Requirements

## REQ-CAP-001

All advanced features shall be capability-driven.

Priority: MUST

---

## REQ-CAP-002

Tracking logic shall not directly depend on board names.

Priority: MUST

Preferred:

```cpp
if(capability.compass)
```

Forbidden:

```cpp
if(board == ESP32S3)
```

---

## REQ-CAP-003

Capabilities shall be determined using:

```text
Build Target Support
+
Runtime Detection When Required By Hardware
+
User Configuration
```

Software-only features do not require runtime hardware detection; their availability is determined by build support and user configuration.

Priority: MUST

---

## REQ-CAP-004

Missing optional capabilities shall not prevent operation.

Priority: MUST

---

## REQ-CAP-005

All optional capabilities shall support graceful fallback behavior.

Priority: MUST

---

# 5. Target System Requirements

In the reviewed Backpack implementation, AAT builds are PlatformIO environments in `targets/aat.ini`. AAT-Pro target additions shall follow the current Backpack build and upload conventions; this does not assert that proposed product-family names are already registered.

## REQ-TARGET-001

AAT-Pro targets shall be registered through the Backpack target system.

Priority: MUST

---

## REQ-TARGET-002

Target definitions shall reuse Backpack target registration mechanisms.

Priority: MUST

---

## REQ-TARGET-003

Target definitions shall specify:

```text
Platform
Capabilities
Firmware
Upload Methods
Default Pin Assignments
```

Priority: MUST

---

## REQ-TARGET-004

Target additions shall not require modifications to existing AAT Classic targets.

Priority: MUST

---

# 6. Telemetry Requirements

## REQ-TLM-001

The system shall support CRSF telemetry.

Priority: MUST

---

## REQ-TLM-002

The system shall support Backpack telemetry forwarding.

Priority: MUST

---

## REQ-TLM-003

The system shall support MAVLink telemetry.

Priority: SHOULD

---

## REQ-TLM-004

The system shall automatically detect telemetry loss.

Priority: MUST

Acceptance Criteria:

```text
No valid aircraft-position update from the selected telemetry source for 3 seconds
-> one Telemetry Loss Event
```

The event shall clear only after a valid aircraft-position update is accepted. Loss handling actions are specified separately by REQ-SAFE-002.

---

## REQ-TLM-005

The system shall report telemetry update rate.

Priority: SHOULD

---

## REQ-TLM-006

The system shall report packet loss statistics.

Priority: SHOULD

---

# 7. GPS Requirements

## REQ-GPS-001

The system shall support aircraft GPS data.

Priority: MUST

---

## REQ-GPS-002

The system shall support an optional tracker GPS.

Priority: MUST

---

## REQ-GPS-003

The system shall support automatic home acquisition.

Priority: MUST

---

## REQ-GPS-004

Tracker-local GPS home acquisition shall require:

```text
3D Fix
Satellite Count >= 10
HDOP <= 1.5
Stable Position >= 30 seconds
```

Priority: MUST

Acceptance Criteria:

- All listed fix-quality conditions remain valid for at least 30 seconds before a tracker-GPS home is accepted.
- The position-stability radius is a design input and must be specified before implementation (see `backlog.md`).
- These tracker-GPS quality gates do not silently change the existing CRSF telemetry-home behavior.

---

## REQ-GPS-005

The system shall reject implausible GPS jumps.

Priority: MUST

Acceptance Criteria:

- A rejected jump does not update the accepted position or home.
- The rejection threshold and recovery behavior are specified and tested as DEC-09 in `backlog.md` before implementation.

---

## REQ-GPS-006

Tracker GPS failure shall not prevent operation.

Priority: MUST

Fallback:

```text
Telemetry Home
```

The source-selection and precedence rules for manual, tracker-GPS, and telemetry home remain a design decision tracked in `backlog.md`.

---

# 8. Compass Requirements

## REQ-COMP-001

The system shall support an optional compass.

Priority: MUST

---

## REQ-COMP-002

The following sensors should be supported:

```text
QMC5883L
LIS3MDL
BMM150
```

Priority: SHOULD

---

## REQ-COMP-003

Compass calibration shall be supported.

Priority: MUST

---

## REQ-COMP-004

Compass health shall be monitored.

Priority: MUST

---

## REQ-COMP-005

Compass failure shall not stop tracking.

Priority: MUST

Fallback:

```text
Manual Heading Mode
```

---

# 9. Navigation Requirements

## REQ-NAV-001

The system shall calculate:

```text
Distance
Bearing
Elevation
```

Priority: MUST

---

## REQ-NAV-002

Geodetic coordinate calculations shall be used.

Priority: MUST

---

## REQ-NAV-003

Elevation shall support relative altitude calculations.

Priority: MUST

---

# 10. Pan/Tilt Requirements

## REQ-PAN-001

Standard servo mode shall be supported.

Priority: MUST

---

## REQ-PAN-002

Continuous 360-degree pan mode shall be supported only when the system has a known absolute pan orientation, obtained through position feedback or a defined homing procedure.

Priority: MUST

Acceptance Criteria:

- The mode cannot enter active tracking with unknown pan orientation.
- Startup, loss of orientation, and recovery behavior are defined for the selected actuator before implementation.

---

## REQ-PAN-003

Continuous pan tracking shall use shortest-path calculations.

Priority: MUST

Acceptance Criteria:

- A target change from 359 degrees to 1 degree commands a +2-degree move.
- A target change from 1 degree to 359 degrees commands a -2-degree move.
- Exact 180-degree ties use a documented, deterministic direction.

---

## REQ-PAN-004

Tilt servos up to 180 degrees shall be supported.

Priority: SHOULD

---

## REQ-PAN-005

Maximum rotation speed shall be configurable.

Priority: MUST

Acceptance Criteria:

- The configured limit is expressed in angular units per second and is not exceeded by the commanded trajectory.
- Supported range and units shall be set from the selected actuator in `backlog.md` before implementation.

---

## REQ-PAN-006

Maximum angular acceleration shall be configurable.

Priority: SHOULD

---

# 11. Filtering Requirements

## REQ-FLT-001

GPS filtering shall be supported.

Priority: MUST

---

## REQ-FLT-002

EMA filtering shall be supported.

Priority: MUST

---

## REQ-FLT-003

Servo deadband shall be supported.

Priority: MUST

---

## REQ-FLT-004

Deadband shall be configurable.

Priority: MUST

---

## REQ-FLT-005

Distance-adaptive deadband should be supported.

Priority: SHOULD

---

## REQ-FLT-006

Tilt easing should be supported.

Priority: SHOULD

---

# 12. PID Requirements

## REQ-PID-001

PID control shall be supported.

Priority: SHOULD

---

## REQ-PID-002

Independent PID controllers shall be available for:

```text
Pan
Tilt
```

Priority: SHOULD

---

## REQ-PID-003

PID shall remain disabled by default.

Priority: MUST

---

# 13. Prediction Requirements

## REQ-PRED-001

Prediction shall be optional.

Priority: MUST

---

## REQ-PRED-002

Linear prediction shall be supported.

Priority: MUST

---

## REQ-PRED-003

Adaptive prediction shall be supported.

Priority: SHOULD

---

## REQ-PRED-004

The system shall support short-term tracking continuation during telemetry outages.

Priority: SHOULD

Acceptance Criteria:

- Continuation is bounded by the outage policy in DEC-04 and cannot override the configured telemetry-loss action.

---

## REQ-PRED-005

Prediction failure shall automatically revert to standard tracking.

Priority: MUST

---

# 14. Latency Compensation Requirements

## REQ-LAT-001

Manual latency compensation shall be supported.

Priority: MUST

---

## REQ-LAT-002

Automatic latency compensation shall be supported.

Priority: SHOULD

---

## REQ-LAT-003

Latency estimation range:

```text
200 ms
to
1000 ms
```

Priority: MUST

---

## REQ-LAT-004

Prediction quality shall be monitored continuously.

Priority: SHOULD

---

## REQ-LAT-005

The system shall automatically reduce prediction aggressiveness if prediction quality degrades.

Priority: SHOULD

---

# 15. Safety Requirements

## REQ-SAFE-001

Telemetry loss shall be detected.

Priority: MUST

---

## REQ-SAFE-002

Configurable telemetry-loss behavior shall be supported.

Priority: MUST

Available Modes:

```text
Hold
Park
Return Home Direction
```

---

## REQ-SAFE-003

Zenith protection shall be implemented.

Priority: MUST

Acceptance Criteria:

- The boundary condition and fallback behavior are specified and tested as DEC-10 in `backlog.md` before implementation.

---

## REQ-SAFE-004

Servo health monitoring should be supported.

Priority: SHOULD

Acceptance Criteria:

- The quality metric, evaluation window, and degradation threshold are specified as DEC-11 in `backlog.md` before adaptive prediction is enabled.

---

## REQ-SAFE-005

Servo stall detection should be supported.

Priority: SHOULD

---

## REQ-SAFE-006

Startup self-tests should be supported.

Priority: SHOULD

---

# 16. Configuration Storage Requirements

## REQ-CFG-001

Configuration shall use Backpack-native persistence mechanisms.

Priority: MUST

---

## REQ-CFG-002

No independent storage system shall be introduced.

Priority: MUST

---

## REQ-CFG-003

Configuration migration shall not require manual intervention.

Priority: MUST

Acceptance Criteria:

- A valid existing Backpack VRX config version 5 retains all existing settings after upgrade.
- New config fields are initialized without resetting unrelated stored settings.
- Migration behavior is covered by tests for both a legacy config and a new config.

---

# 17. User Interface Requirements

## REQ-UI-001

All functionality shall remain usable without an OLED display.

Priority: MUST

---

## REQ-UI-002

Operation without display shall be supported.

Priority: MUST

---

## REQ-UI-003

Operation with OLED display shall be supported.

Priority: MUST

---

## REQ-UI-004

Single-button operation shall be supported.

Priority: MUST

---

## REQ-UI-005

Additional buttons shall remain optional.

Priority: SHOULD

---

# 18. WebUI Requirements

## REQ-WEB-001

AAT-Pro shall extend the existing Backpack WebUI infrastructure.

Priority: MUST

---

## REQ-WEB-002

No standalone WebUI framework shall be created.

Priority: MUST

---

## REQ-WEB-003

AAT-Pro pages shall integrate into existing Backpack navigation.

Priority: SHOULD

---

## REQ-WEB-004

A Status page shall be provided.

Priority: MUST

---

## REQ-WEB-005

A Configuration page shall be provided.

Priority: MUST

---

## REQ-WEB-006

A Servo page shall be provided.

Priority: MUST

---

## REQ-WEB-007

A Diagnostics page shall be provided.

Priority: MUST

---

## REQ-WEB-008

A GPS page should be provided.

Priority: SHOULD

---

## REQ-WEB-009

A Compass page should be provided.

Priority: SHOULD

---

## REQ-WEB-010

A Prediction page should be provided.

Priority: SHOULD

---

# 19. Diagnostics Requirements

## REQ-DIAG-001

The system shall expose telemetry statistics.

Priority: SHOULD

---

## REQ-DIAG-002

The system shall expose GPS health information.

Priority: SHOULD

---

## REQ-DIAG-003

The system shall expose compass health information.

Priority: SHOULD

---

## REQ-DIAG-004

The system shall expose prediction metrics.

Priority: SHOULD

---

## REQ-DIAG-005

Debug logging should be supported.

Priority: SHOULD

---

# 20. Performance Requirements

## REQ-PERF-001

Tracking calculations shall never block telemetry processing.

Priority: MUST

---

## REQ-PERF-002

The tracking loop shall execute deterministically.

Priority: MUST

Acceptance Criteria:

- Tracking execution does not depend on WebUI request timing.
- Maximum loop period and jitter are quantified on each supported target before implementation is considered complete; the thresholds are recorded in `backlog.md`.

---

## REQ-PERF-003

WebUI activity shall not degrade tracking performance.

Priority: MUST

Acceptance Criteria:

- Tracking loop timing is measured with and without concurrent WebUI requests on each supported target.
- Timing remains within the REQ-PERF-002 limits.

---

## REQ-PERF-004

The architecture shall remain compatible with ESP32-class hardware.

Priority: MUST

---

# 21. Compatibility Requirements

## REQ-COMPAT-001

AAT Classic users shall experience no functional change.

Priority: MUST

---

## REQ-COMPAT-002

Existing Backpack targets shall remain supported.

Priority: MUST

---

## REQ-COMPAT-003

Optional hardware failures shall not prevent operation.

Priority: MUST

---

## REQ-COMPAT-004

Current Backpack targets shall compile without modification.

Priority: MUST

Acceptance Criteria:

- Existing target definitions remain unchanged by an AAT-Pro feature addition.
- The current AAT environments listed in `backpack_audit.md` and the agreed existing-target regression matrix compile successfully.

---

## REQ-COMPAT-005

Disabling AAT-Pro features shall restore baseline behavior.

Priority: MUST

---

## REQ-COMPAT-006

The firmware shall compile cleanly when optional hardware is absent.

Priority: MUST

---

# 22. Upstream Contribution Requirements

## REQ-UPSTREAM-001

Features shall be divisible into independent pull requests.

Priority: MUST

---

## REQ-UPSTREAM-002

Each pull request shall provide standalone value.

Priority: MUST

---

## REQ-UPSTREAM-003

Each feature shall degrade gracefully when disabled.

Priority: MUST

---

## REQ-UPSTREAM-004

No pull request shall require all AAT-Pro functionality to be merged simultaneously.

Priority: MUST

---

# 23. Release Roadmap

The implementation sequence, dependencies, open design decisions, and exit criteria are maintained in `backlog.md`. Version labels below are provisional milestones, not release-date commitments.

## v0.1

```text
Refactoring Foundation
```

---

## v0.2

```text
Target Definition Layer
```

---

## v0.3

```text
Local GPS
```

---

## v0.4

```text
Compass Support
```

---

## v0.5

```text
Continuous 360-degree Tracking
```

---

## v0.6

```text
WebUI Extensions
```

---

## v0.7

```text
Filtering
```

---

## v0.8

```text
PID Support
```

---

## v0.9

```text
Prediction Engine
```

---

## v1.0

```text
Diagnostics
```

---

## v1.1

```text
Automatic Latency Compensation
```

---

## v1.2

```text
MAVLink Support
```

---

# 24. Success Criteria

AAT-Pro shall be considered successful when:

- AAT Classic remains unaffected.
- New Backpack targets integrate cleanly.
- No parallel ecosystem is created.
- Configuration is primarily WebUI-based.
- Advanced hardware remains optional.
- Features are mergeable as independent pull requests.
- Features are mergeable as independent pull requests.
- The architecture scales from ESP8266 AAT Classic to ESP32-S3 AAT-Pro Full.
 The architecture scales from ESP8266 AAT Classic to ESP32-S3 AAT-Pro Full.
