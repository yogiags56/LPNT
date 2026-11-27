# LiberPhoneNT Architecture Overview

**Status:** Concept baseline; interfaces are not yet implemented.

## 1. Architectural boundary

LiberPhoneNT is a mobile product layer built on an independently maintained upstream foundation:

- **Upstream layer — LiberNT:** kernel, HAL, boot infrastructure, drivers, system libraries, and existing operating-system components.
- **Mobile platform layer — LPNT:** mobile-oriented services, hardware integration, lifecycle, permissions, package management, and device configuration.
- **Experience layer — Metro Live Shell:** Lock Screen, Start Screen, Live Tiles, app navigation, notifications, and system UI.
- **Application layer:** native mobile applications and, later, DMAF-based adapted applications.

The mobile layer must not silently assume that an upstream desktop capability is already suitable for a phone.

## 2. Conceptual component map

```text
+------------------------------------------------------+
|                 Applications                         |
|       Native Apps             DMAF Apps               |
+------------------------------------------------------+
|              Metro Live Shell                        |
| Lock Screen | Start | Live Tiles | All Apps | UI      |
+------------------------------------------------------+
|               Mobile Platform Services               |
| App Lifecycle | Notifications | Permissions | Power  |
| Package Manager | Settings | Accounts | Connectivity  |
+------------------------------------------------------+
|              Mobile Hardware Abstraction             |
| Display | Touch | Audio | Camera | Sensors | Modem    |
| Storage | USB | I2C | SPI | GPIO | Battery | Thermal  |
+------------------------------------------------------+
|             LiberNT Upstream Foundation               |
| Kernel | HAL | Boot | Drivers | System Components     |
+------------------------------------------------------+
|                   ARM64 Hardware                      |
+------------------------------------------------------+
```

This is a logical target architecture, not a claim that these components currently exist.

## 3. Hardware abstraction requirements

Each hardware interface must document:
- supported device or board identifiers;
- bus and firmware dependencies;
- driver source and license;
- initialization and shutdown sequence;
- interrupt and concurrency behavior;
- power-state behavior;
- test coverage and known limitations.

The first hardware audit should prioritize boot, display, touch, storage, serial debugging, and power control. Cellular, camera, sensors, and audio follow according to the selected target device.

## 4. Mobile service requirements

The platform service design must eventually cover:

- **Application lifecycle:** launch, foreground, background, suspension, termination, and restoration.
- **Input and display:** touch events, orientation, density, scaling, accessibility, and multi-window policy.
- **Notifications:** delivery, grouping, priority, and user controls.
- **Permissions and security:** explicit access to sensitive hardware and user data.
- **Power:** battery state, charging, thermal constraints, background limits, suspend, and resume.
- **Connectivity:** Wi-Fi, Bluetooth, cellular data, SIM and telephony where hardware and drivers permit.
- **Package management:** installation, signing, updates, rollback, and removal.

These are requirements for investigation and design, not implemented features.

## 5. Metro Live experience principles

The visual direction is inspired by Windows Phone 8.1 and Metro Live, while implementation and branding remain independent.

- Typography and content hierarchy should remain clear at phone sizes.
- Live Tiles should expose useful at-a-glance information without overwhelming the Start Screen.
- Navigation must be touch-first and predictable.
- Motion should communicate state, not obstruct interaction.
- Dark, Light, and Adaptive Wallpaper themes should share accessible contrast rules.
- Layouts must adapt to different resolutions and aspect ratios without stretching content.

## 6. DMAF boundary

DMAF is a future adaptation framework, not a compatibility promise. Its purpose is to let selected desktop application models present mobile-appropriate layouts and interaction patterns.

The framework must define which application APIs it can adapt, which behaviors require application changes, and which desktop behaviors cannot be represented safely or usefully on mobile.

## 7. Open decisions

The following decisions remain intentionally unresolved:
- first target device or development board;
- graphics/compositor implementation strategy;
- mobile application ABI and package format;
- service isolation and IPC model;
- modem and telephony stack;
- native application toolkit;
- DMAF source application models;
- release and update mechanism.

Decisions should be recorded only after source and hardware feasibility evidence is available.
