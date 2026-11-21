# ExpressLRS Backpack AAT Continuous Pan

Planning repository for an optional continuous-rotation pan mode in the existing ExpressLRS Backpack antenna tracker, not a separate Pro product.

## Goals

- Preserve Backpack's CRSF aircraft-GPS tracking, positional modes, configuration and WebUI.
- Add opt-in continuous pan with compass feedback and safety checks on supported existing hardware.
- Keep bounded +/-170-degree travel as the default; plan an unrestricted option only for hardware that can safely rotate indefinitely.
- Allow safe mode changes without a reboot where possible, subject to the existing Backpack Wi-Fi workflow.

ESP32-S3, MAVLink, tracker-local GPS and new prediction systems are outside the active scope.

## Status

Planning and bench implementation are in progress. Firmware development is in a separate `Backpack` repository; this repository contains specifications, audit and work plan, not firmware source.

The ESP32 continuous-pan prototype passed powered directional tracking and a simulated GPS orbit using bounded travel. Its mode selection currently requires a reboot. Unrestricted travel and reboot-free mode switching are planned, not implemented; a settled stop at the negative travel clamp and other field evidence remain open.

## Resume

Start with [the backlog](docs/backlog.md) for current priorities and [test and engineering history](docs/backlog-history.md) for past bench results and handoffs. See also [architecture](docs/architecture.md), [requirements](docs/requirements.md), and [the Backpack implementation audit](docs/backpack_audit.md).