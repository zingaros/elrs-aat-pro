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

The ESP32 continuous-pan prototype has passed powered directional tracking and a simulated GPS orbit. A settled stop at the negative travel clamp and other field evidence remain open; see the backlog for current status.

## Resume

Start with [the backlog](docs/backlog.md) for current priorities and [test and engineering history](docs/backlog-history.md) for past bench results and handoffs. See also [architecture](docs/architecture.md), [requirements](docs/requirements.md), and [the Backpack implementation audit](docs/backpack_audit.md).