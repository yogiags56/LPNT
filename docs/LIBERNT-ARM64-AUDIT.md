# LiberNT ARM64 Feasibility Audit — Initial Evidence

**Status:** Preliminary source/documentation audit  
**Audited upstream:** `LiberNT/LiberNT`, default branch `master`  
**Audit date:** 2026-11-28  
**Scope:** Public upstream README and repository tree. This is not a build or hardware test.

## 1. Executive summary

LiberNT explicitly targets ARM64 and documents prebuilt ARM64 images, ARM64 source builds, UEFI boot, GOP framebuffer early display, modern bus/storage support, and Raspberry Pi board support including RP1 I/O.

This is meaningful evidence that LiberNT has an ARM64 operating-system foundation. It is **not evidence that it is ready to run as a smartphone OS**. The reviewed material does not establish a supported smartphone model, touchscreen driver, cellular modem/SIM/telephony stack, camera stack, mobile sensor integration, smartphone battery/charging policy, or phone-grade suspend/resume behavior.

**Current conclusion:** ARM64 is a plausible development direction; smartphone feasibility remains unproven and must be assessed against a specific board/device and source revision.

## 2. Evidence confirmed in upstream documentation

The upstream README states:

- LiberNT targets amd64, arm64, and riscv64, with ARM64 prebuilt images and source builds.
- UEFI boot is supported, with GOP framebuffer and early display support.
- The project lists PCIe, PCI, Plug and Play, USB xHCI, SD/eMMC, and storage/bus drivers.
- Board support includes Raspberry Pi and its RP1 I/O controller.
- The HAL and FreeLoader are described as running on ARM64.
- ARM64 builds can be configured with `./configure.sh -a arm64`.

These are upstream project statements, not independent test results from this LPNT audit.

## 3. Repository-tree evidence

The public upstream tree contains ARM64-specific paths, including:

- `bootloader/arch/arm64/`
- `bootloader/uefi/`
- `bootloader/include/arm64.h`
- `hal/halarm/` (directory family requires deeper source review)
- ARM64-specific NTDLL paths under `dll/ntdll/arm64/`

The presence of architecture-specific source files confirms that ARM64 implementation work exists in the tree. It does not, by itself, establish correctness on a particular smartphone or board.

## 4. Smartphone capability matrix

| Capability | Preliminary evidence | Status for LPNT |
| --- | --- | --- |
| ARM64 build target | Explicitly documented upstream | Upstream-declared |
| ARM64 bootloader code | ARM64-specific source paths present | Source presence confirmed; behavior not tested here |
| UEFI boot | Explicitly documented upstream | Upstream-declared |
| Early display | GOP framebuffer documented | Early display only; phone display pipeline unverified |
| USB / SD / eMMC | Listed in upstream feature overview | Upstream-declared; target-device validation needed |
| Raspberry Pi board support | Explicitly documented upstream | Board family evidence; exact board/revision must be pinned |
| Touchscreen | No evidence in the reviewed material | Unknown / audit required |
| Smartphone GPU acceleration | No phone-specific evidence in reviewed material | Unknown / audit required |
| Cellular modem, SIM, telephony, SMS | No evidence in reviewed material | Unknown / audit required |
| Wi-Fi / Bluetooth on target phone | Generic networking claims are insufficient | Unknown / device audit required |
| Camera / ISP | No evidence in reviewed material | Unknown / audit required |
| Sensors / GPS | No evidence in reviewed material | Unknown / audit required |
| Battery / charging / thermal policy | No smartphone-specific evidence in reviewed material | Unknown / audit required |
| Suspend / resume / wake sources | No phone-specific evidence in reviewed material | Unknown / audit required |

## 5. Risks and constraints

1. **No target phone selected.** Driver and firmware feasibility cannot be established without a precise device model and hardware revision.
2. **UEFI/GOP is not a mobile graphics stack.** It can provide boot-time display but does not prove accelerated composition, panel control, refresh-rate management, or power-efficient rendering.
3. **Architecture support is not peripheral support.** ARM64 CPU support does not imply touchscreen, modem, camera, sensors, or power management.
4. **Board support is not interchangeable.** Raspberry Pi support cannot be generalized to unrelated smartphone SoCs.
5. **Upstream changes frequently.** A reproducible audit must pin an exact upstream commit and re-run when that baseline changes.

## 6. Next audit tasks

- [ ] Pin the exact upstream `master` commit SHA used for this audit.
- [ ] Inspect ARM64 HAL implementation and platform selection.
- [ ] Inspect ARM64 boot flow, device-tree/ACPI handoff, interrupt controller, timers, and early UART.
- [ ] Inventory actual ARM64 board drivers and identify exact tested boards.
- [ ] Search for touchscreen/input drivers and verify whether they are usable on ARM64.
- [ ] Inspect display, graphics, and compositor paths beyond GOP.
- [ ] Inspect power-management, battery, thermal, suspend, and resume implementations.
- [ ] Identify whether any modem, SIM, telephony, camera, and sensor drivers exist for a candidate target.
- [ ] Select a realistic first development target before making phone-support claims.

## 7. Source references

- Upstream repository: https://github.com/LiberNT/LiberNT
- Upstream README: https://github.com/LiberNT/LiberNT/blob/master/README.md
- ARM64 bootloader directory: https://github.com/LiberNT/LiberNT/tree/master/bootloader/arch/arm64
- UEFI bootloader directory: https://github.com/LiberNT/LiberNT/tree/master/bootloader/uefi

## 8. Audit limitations

This document records a first-pass review of upstream documentation and repository paths only. No upstream code was copied into LPNT, no LiberNT build was run, no phone hardware was tested, and no smartphone compatibility claim is made.
