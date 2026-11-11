# ELRS-AAT-Pro

Planning repository for an advanced antenna tracker built as an ExpressLRS Backpack extension.

## Goals

- GPS Home
- Compass Support
- Continuous 360° Tracking
- Prediction Engine
- Auto Latency Compensation
- MAVLink Support
- Extended WebUI

## Status

Planning and bench implementation in progress. Firmware development is in the separate sibling `Backpack` repository; this repository contains specifications, audit, and work plan.

The ESP32 commissioning prototype has verified pan neutral, owner compass calibration, site-offset refresh, and retained calibration status after reboot. Continuous-pan tracking remains disabled pending motion-reference and travel-safety work. Calibration UX and consistency with existing Backpack options need redesign.

## Resume

Start with [the closeout handoff and backlog](docs/backlog.md), then consult [architecture](docs/architecture.md), [requirements](docs/requirements.md), and [the Backpack implementation audit](docs/backpack_audit.md). The handoff records the current COM6 image, pins, bench results, commands, and remaining gates.