# LiberPhoneNT

![LiberPhoneNT official logo](assets/logo.png)

> **Working project name.** LiberPhoneNT is a provisional name and is not a final product or brand decision.

LiberPhoneNT is an experimental mobile operating-system project built on top of [LiberNT](https://github.com/LiberNT/LiberNT), with a smartphone-oriented system architecture and a user experience inspired by the Metro design language and Windows Phone 8.1.

The goal is to investigate and build a genuinely mobile operating system—not merely scale a desktop interface down to a phone display.

## Project identity

| Area | Direction |
| --- | --- |
| Working name | LiberPhoneNT (provisional) |
| Upstream foundation | [LiberNT](https://github.com/LiberNT/LiberNT) |
| Initial architecture target | ARM64 (subject to board and driver audit) |
| Design language | Metro Live |
| Shell inspiration | Windows Phone 8.1 / Lumia |
| Adaptive application framework | DMAF — Desktop-to-Mobile Adaptive Application Framework |

## Project principles

1. **Mobile-first interaction:** touch, compact layouts, gestures, and one-handed use are first-class requirements.
2. **Native system architecture:** define mobile services and hardware interfaces explicitly instead of assuming desktop components are phone-ready.
3. **Upstream separation:** keep LiberNT as the upstream operating-system foundation and maintain the mobile product layer in this repository.
4. **Evidence-based compatibility:** distinguish upstream capabilities, audited capabilities, implemented features, and hardware-tested features.
5. **Open development:** document design decisions, interfaces, limitations, and test results.

## Current status

The repository is at the documentation and feasibility stage. No smartphone-ready build, supported phone list, or working mobile shell is claimed at this time.

The first milestone is an ARM64 source and platform audit of LiberNT, followed by a target-device strategy and mobile architecture specification.

See:
- [Roadmap](docs/ROADMAP.md)
- [Architecture overview](docs/ARCHITECTURE.md)
- [LiberNT ARM64 feasibility audit](docs/LIBERNT-ARM64-AUDIT.md)

## Upstream and licensing

LiberNT is a separate upstream project. Its repository identifies the main project license as GNU GPL-3.0 and notes that inherited components may carry their own licenses. Before copying, modifying, linking, or redistributing upstream components, we must inspect the applicable file headers, notices, component licenses, and trademark guidance.

This repository does not claim affiliation with or endorsement by the LiberNT contributors or Microsoft. Windows and Windows Phone are trademarks of Microsoft Corporation. Product naming, visual assets, and branding for this project must be independently developed.

## Contributions and status labels

When documenting a feature, use one of these labels:

- **Upstream:** present in the upstream LiberNT project.
- **Audited:** verified in source or documentation for a specific revision.
- **Implemented:** added in this repository or an identified integration branch.
- **Tested:** verified by a reproducible test; include hardware or emulator details.
- **Blocked:** known dependency or missing capability prevents progress.

Do not describe a feature as supported merely because its CPU architecture is supported.
