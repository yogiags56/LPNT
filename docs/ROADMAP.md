# LiberPhoneNT Development Roadmap

**Status:** Initial planning baseline  
**Project name:** Provisional  
**Upstream:** [LiberNT/LiberNT](https://github.com/LiberNT/LiberNT)

This roadmap is staged around technical risk. The project must prove that the upstream foundation can be adapted to a phone-class device before investing heavily in the shell or application ecosystem.

## Phase 0 — Repository and project baseline

- [x] Establish the LPNT repository.
- [x] Add project overview and initial roadmap.
- [x] Record the upstream repository and separate product-layer direction.
- [ ] Establish contribution, issue, and decision-record conventions.
- [ ] Complete a license and third-party component review before code reuse.

## Phase 1 — LiberNT ARM64 feasibility audit

**Milestone ID: LPN-001**

- [x] Pin the exact upstream commit to be audited: `20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9`.
- [x] Map ARM64 bootloader source and ACPI/PSCI handoff at source level.
- [x] Inspect ARM64 HAL, GIC, DMA, PCI routing, and secondary CPU startup source.
- [x] Inventory initial ARM64 board/display evidence, including Raspberry Pi 5.
- [x] Audit the GOP framebuffer and RPi5 display-only miniport paths.
- [x] Record touchscreen/HID evidence and its explicit unsupported test skip.
- [x] Identify major mobile gaps: modem/SIM/telephony, camera, sensors/GPS, battery/charging, thermal, and phone-grade suspend/resume.
- [x] Produce a preliminary capability matrix with source references and confidence levels.
- [ ] Build and boot ARM64 images in a virtual environment and on a documented board.
- [x] Inventory explicit stubs and unimplemented entry points in `halarm64.c`, with source evidence and preliminary impact assessment.
- [ ] Complete call-site/call-graph analysis for every identified stub and audit the remaining ARM64 HAL source files.
- [ ] Validate display, input, storage, USB, and power on Raspberry Pi 5 or another selected board.

**Exit criteria:** a reproducible audit report tied to an upstream commit, with no unverified phone-readiness claims. The initial source review and a focused `halarm64.c` deep audit are documented; full call-graph and cross-file HAL review plus hardware/build validation remain open.

## Phase 2 — Target device and bring-up strategy

**Milestone ID: LPN-002**

- [ ] Define a realistic first hardware target.
- [ ] Confirm boot firmware, boot chain, memory map, storage, display, and input documentation.
- [ ] Determine availability and licensing of board support packages and drivers.
- [ ] Define recovery, serial-console, and debug procedures.
- [ ] Decide whether the first milestone is a development board, emulator, or an actual phone.

**Exit criteria:** one target platform with a documented boot and driver feasibility plan.

## Phase 3 — Mobile system architecture

**Milestone ID: LPN-003**

- [ ] Define the mobile system service model.
- [ ] Specify hardware abstraction boundaries and driver ownership.
- [ ] Define process lifecycle, app activation, background execution, and resource limits.
- [ ] Define notifications, permissions, settings, and device configuration services.
- [ ] Define security boundaries, app identity, package format, signing, and update strategy.
- [ ] Define power states and lifecycle transitions.

**Exit criteria:** reviewed architecture specification and interface contracts.

## Phase 4 — Graphics, input, and shell foundations

**Milestone ID: LPN-004**

- [ ] Establish a display/compositor strategy suitable for a phone.
- [ ] Establish touch input, orientation, scaling, and multi-density behavior.
- [ ] Build a minimal interactive shell before implementing visual polish.
- [ ] Implement Lock Screen and notification surface.
- [ ] Implement Start Screen with Live Tiles.
- [ ] Implement All Apps, navigation, and system Settings.
- [ ] Add Light, Dark, and Adaptive Wallpaper theme foundations.

**Exit criteria:** shell boots and remains usable on the selected target with real touch input.

## Phase 5 — Native application platform

**Milestone ID: LPN-005**

- [ ] Define native application APIs and lifecycle.
- [ ] Define layout, accessibility, localization, and adaptive UI conventions.
- [ ] Define package installation, updates, permissions, and app data isolation.
- [ ] Implement a minimal set of system applications.
- [ ] Establish developer documentation and sample applications.

**Exit criteria:** third-party sample application can be built, installed, launched, suspended, and removed using documented interfaces.

## Phase 6 — DMAF

**Milestone ID: LPN-006**

DMAF means **Desktop-to-Mobile Adaptive Application Framework**. It is intended to help adapt desktop applications into mobile-appropriate experiences; it must not simply shrink desktop windows to fit a phone.

- [ ] Define supported source application models and adaptation boundaries.
- [ ] Define responsive layout and input abstraction.
- [ ] Define navigation, lifecycle, storage, and permission integration.
- [ ] Build a proof-of-concept application adaptation.
- [ ] Document unsupported desktop behaviors and fallback rules.

**Exit criteria:** a demonstrable app adaptation with usable mobile interaction and documented limitations.

## Phase 7 — Device integration and release engineering

**Milestone ID: LPN-007**

- [ ] Integrate audio, camera, sensors, location, cellular, and connectivity where supported by the target.
- [ ] Implement battery, charging, thermal, suspend, and resume behavior.
- [ ] Add automated build and static validation workflows.
- [ ] Add boot, input, graphics, lifecycle, and power regression tests.
- [ ] Define recovery and update procedures.
- [ ] Publish alpha images only for explicitly tested hardware.

**Exit criteria:** repeatable builds and a hardware-specific test report. No generic device compatibility claim without device-level evidence.

## Tracking rules

Every milestone should have:
- a scope and explicit non-goals;
- source revision and hardware target;
- implementation tasks and dependencies;
- reproducible validation steps;
- known limitations and evidence links.

Progress percentages must be calculated from completed, weighted tasks—not estimated from elapsed time or documentation volume.
