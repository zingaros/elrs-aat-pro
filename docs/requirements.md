# Backpack AAT Continuous-Pan Requirements

Status: Draft target-state requirements. The local prototype implements some, but not all, of these requirements; see [architecture](architecture.md) for current behavior and [backlog](backlog.md) for remaining work and verification. Historical requirement references in [test history](backlog-history.md) refer to earlier versions of this specification.

The goal is an opt-in continuous-rotation pan option within the existing ExpressLRS Backpack AAT, not a separate Pro product. ESP32-S3, MAVLink, tracker-local GPS, new prediction systems, PID and other proposed Pro features are outside the active scope. Preserve Backpack's existing optional projection and configuration.

## Backpack integration and compatibility

### REQ-CONST-001: Native integration

Continuous pan shall use the existing Backpack AAT application and target conventions, rather than a standalone firmware ecosystem. Reuse Backpack telemetry forwarding, build and update mechanisms, configuration storage and WebUI. Any target additions shall follow existing Backpack conventions.

### REQ-PROD-002: Preserve existing behavior

Existing positional modes, CRSF telemetry, aircraft-GPS home acquisition, navigation, elevation, WebUI and supported targets shall retain their behavior when continuous pan is not selected. Missing optional compass hardware shall not prevent normal positional operation.

### REQ-PROD-004: Opt-in

Continuous pan shall be disabled by default for new configurations and shall not be enabled by upgrading an existing configuration. Existing saved values, including the nonzero legacy `project` value, shall be preserved.

### REQ-TARGET-005: Supported hardware

Offer continuous pan only on builds with the required pins, compass support and hardware. Preserve existing ESP8266 and other Backpack targets; do not require a new hardware family. The local ESP32 DevKit and HMC5883L-compatible compass are the current prototype profile, not a claim that every ESP32 board or compass is supported.

## Telemetry and navigation

### REQ-GPS-001: Existing aircraft GPS

Use the existing Backpack CRSF aircraft-GPS path and its satellite-count-based home acquisition, navigation and optional azimuth projection. Do not introduce tracker-local GPS, a separate home workflow, new GPS source selection or new GPS jump-rejection behavior.

### REQ-NAV-001: Existing navigation and elevation

Continue using Backpack's distance, bearing and relative-altitude elevation calculations and existing positional elevation configuration. Continuous-pan integration shall not change legacy positional elevation behavior.

## Pan modes and transitions

### REQ-PAN-001: Positional and continuous modes

Retain the existing positional modes and add continuous pan as an explicit option. Use the existing mode field where practical. The mode distinction shall prevent positional commands from being sent to a continuous-rotation actuator.

### REQ-PAN-002: Valid orientation and setup

Continuous tracking shall not start with unknown orientation or invalid setup. A valid calibrated heading, required north alignment, mechanical-home confirmation, pulse calibration, fresh GPS and compass feedback, and applicable safety gates shall be established before tracking. Startup, reference loss and recovery shall be defined and tested.

### REQ-PAN-007: Safe mode transitions

Permit switching between positional and continuous pan without reboot when safe. On either transition, inhibit pan output and clear stale controller state. Require explicit confirmation that the correct actuator is installed before enabling output; entering continuous mode also requires the gates in REQ-PAN-002. Neither controller may arm the wrong actuator. Preserve ordinary legacy operation when no mode transition is requested.

The current prototype is boot-latched and requires reboot. The Backpack Wi-Fi configuration lifecycle shall be investigated separately; do not claim a completely reboot-free configuration workflow until verified. A software neutral request or detached output does not establish a physical stop.

## Travel policies

### REQ-PAN-008: Bounded default

Default to the existing home-relative +/-170-degree target clamp and bounded route. The controller shall not choose a rear-wrap shortcut across the boundary; a target crossing the rear may require a return through home. Out-of-range measured pan or invalid reference shall inhibit tracking and require safe recovery.

### REQ-PAN-003: Unrestricted shortest path

Provide an explicitly selected unrestricted policy only after the owner confirms the hardware can safely rotate indefinitely, for example with a slip ring or no winding cables. Use the shortest angular path without applying the bounded policy's travel-limit assumptions.

Acceptance criteria:

- From a current heading of 359 degrees to a target of 1 degree, command a +2-degree move; from 1 to 359 degrees, command a -2-degree move.
- An exact 180-degree tie uses a documented, deterministic direction.
- Bounded operation remains the default; unrestricted operation requires explicit selection and physical-setup confirmation.

### REQ-PAN-009: Safe policy changes

Inhibit pan output and safely revalidate required setup when changing travel policy. Both policies retain valid heading, reference, GPS freshness, neutral and output-envelope safeguards. A compass cannot count cable turns or detect a whole revolution missed between samples; unrestricted operation shall not be presented as safe for limited-travel cabling.

## Compass, calibration and output safety

### REQ-COMP-001: Compass feedback

Continuous pan shall use a supported, calibrated compass and monitor heading validity and freshness. A missing, stale or invalid compass shall inhibit continuous tracking and select neutral; it shall not silently fall back to unverified heading. Positional tracking without the optional compass remains available.

### REQ-COMP-003: Commissioning and reference

Use the existing Backpack WebUI and operator controls for compass fit, mounting-direction verification, north alignment, mechanical-home confirmation and minimum-motion pulse calibration. Preserve the existing protections unless a simpler approach is verified to be equivalent:

- Validate compass-fit coverage and retain the prior valid fit on a failed sweep. Preserve owner axis scales when an existing site-offset refresh succeeds.
- Keep compass calibration, north alignment, mechanical home and servo neutral distinct. Confirm mechanical home again after boot; do not silently establish it by timeout.
- Permit powered sweeps, pulse calibration and manual direction checks only through explicit operator action, with stop, sensor-loss, timeout and applicable safety exits. Failed or cancelled calibration shall not save partial results as valid.
- Changing settings that invalidate calibration or alignment shall block tracking until the affected setup is restored.

The precise current prototype sequence and its limitations are documented in [architecture](architecture.md).

### REQ-SAFE-001: Freshness and failure

Continuous tracking shall select neutral when required GPS or compass data is stale, reference or calibration is invalid, or another tracking safety gate fails. Preserve at least the existing prototype's compass freshness and configuration checks, GPS three-interval freshness rule with a hard age cap below three seconds, and reference-loss recovery unless an equivalent safe change is verified. Recovery shall not bypass a required new mechanical-home confirmation.

The freshness timeout is not an immediate emergency stop. A physical power cutoff remains necessary for unexpected motion; software readback is not proof of an electrical pulse or physical stop.

### REQ-FLT-003: Deadband and pulse envelope

Retain configurable deadband, calibrated directional start pulses and validated neutral-relative output limits. Changing neutral or directional limits shall invalidate affected pulse calibration. The pulse envelope shall remain within the supported 1000-2000 us range. Do not claim a measured angular-speed limit from pulse offsets alone.

### REQ-SAFE-007: Preserve safety under filtering

Filtering shall not mask genuine travel violations or allow movement contrary to raw safety gates. Preserve the current raw-heading guard unless a replacement is verified against real limit crossings and faulty feedback. Compass spikes can still invalidate reference in the current prototype; do not claim the tracking median eliminates that risk.

## Configuration and user interface

### REQ-CFG-001: Backpack storage

Use Backpack-native configuration storage and validation, with no independent storage system. Preserve existing settings during supported migrations and initialize only new fields. Cover legacy and local prototype configurations, serialized storage and current-version reloads; do not silently reset unrelated settings.

### REQ-WEB-001: Existing WebUI

Integrate mode and travel-policy selection, explicit confirmations, calibration controls, readiness and block reasons into the existing Backpack WebUI and endpoints. Show applicable controls without hiding active commissioning stop/neutral actions. Do not require separate pages or a new WebUI framework.

### REQ-UI-001: Optional display

Do not make an OLED a prerequisite for tracking or configuration. Where the existing display is supported, preserve its normal Backpack behavior and make setup/reference prompts available as appropriate.

## Verification and release

### REQ-VERIFY-001: Software verification

Add focused tests for legacy settings and migrations, positional modes, inhibited mode transitions and re-arming, stale/fault recovery, bounded clamp and rear wrap, unrestricted 359/1 wrap and deterministic 180-degree ties, and safe policy changes. Build the existing AAT targets and relevant continuous-pan targets; check WebUI/API and Wi-Fi lifecycle behavior. Existing prototype test results apply only to the tested firmware.

### REQ-VERIFY-002: Hardware verification

Validate physical direction, neutral/fault handling and both travel policies only with explicit owner approval, suitable hardware and an accessible physical cutoff. A bounded setup with limited cable travel shall not be used to validate unrestricted rotation. Record software and physical observations separately; simulated tracking and pulse readback do not establish physical safety.

The implementation sequence and outstanding evidence are maintained in [backlog](backlog.md). Firmware implementation and verification require access to the separate Backpack source.
