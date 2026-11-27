# LiberNT ARM64 Feasibility Audit — Source-Level Findings

**Status:** Preliminary source-level audit; not a hardware validation
**Audited upstream:** `LiberNT/LiberNT`
**Pinned upstream revision:** `20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9` (2026-11-27)
**Audit date:** 2026-11-28
**Scope:** Public README, repository source paths, selected ARM64 HAL/bootloader/graphics/input files. No build or device test was run.

## 1. Executive summary

LiberNT has a real ARM64 code path: its bootloader contains ARM64-specific loader code, its HAL includes ARM64 interrupt/DMA/ACPI/PSCI work, and the kernel has ARM64 secondary-processor startup code. The upstream README also declares ARM64 builds, UEFI/GOP early display, and Raspberry Pi board support.

However, the ARM64 HAL source explicitly describes itself as a **minimal stub implementation to satisfy kernel linkage while the real ARM64 HAL is brought up**. Several functions still return `STATUS_NOT_IMPLEMENTED`. This is a major maturity limitation and means that the existence of ARM64 source must not be confused with a complete, production-ready HAL.

There is also an RPi5 display-only miniport that takes over the firmware GOP framebuffer and uses the HVS display pipeline. This is stronger than boot-time GOP alone, but the inspected implementation does not establish a generic smartphone display stack or prove accelerated composition on phone hardware.

Input evidence is weak for touch: an upstream DirectInput test explicitly skips HID touchscreen tests as unsupported. This is not a complete inventory of every input driver, but it is a concrete warning that touchscreen support cannot be assumed.

**Current conclusion:** ARM64 is a viable investigation target, not yet a validated smartphone platform. A Raspberry Pi 5 is a more evidence-backed first development board than an arbitrary smartphone, but must still pass build, boot, display, input, and power tests before being selected as the official target.

## 2. Pinned source baseline

- Repository: https://github.com/LiberNT/LiberNT
- Revision: https://github.com/LiberNT/LiberNT/commit/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9
- Upstream README at pinned revision: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/README.md

The README states that kernel, HAL, and FreeLoader target amd64, arm64, and riscv64; that UEFI boot and GOP framebuffer early display are supported; and that the project includes PCIe/PCI, USB xHCI, SD/eMMC, and board support including Raspberry Pi and RP1. These are upstream declarations, not independent LPNT test results.

## 3. Source-level findings

### 3.1 Bootloader and platform handoff

- `bootloader/arch/arm64/winldr.c` contains ARM64 loader logic and populates PSCI configuration from ACPI FADT flags, selecting SMC or HVC conduit.
- The ARM64 loader source includes ACPI MADT-related handling.
- The source confirms that ARM64 boot handoff and firmware-described platform data are actively implemented. It does not prove successful boot on a particular board or phone.

### 3.2 HAL, interrupts, DMA, and processor startup

- `ntos/hal/armcommon/halarm64.c` header labels the file a minimal stub implementation while the real ARM64 HAL is being brought up.
- The same source contains ARM64 GIC interrupt handling, PCI configuration/routing, DMA mapping/coherency/SMMU structures, and PSCI system reset/off code.
- The file also contains explicit `STATUS_NOT_IMPLEMENTED` returns; for example, around source lines 1122, 2870, and 3354 at the pinned revision.
- `ntos/hal/arm64/start.c` implements secondary processor startup paths using PSCI CPU_ON, with spin-table/parking fallback logic.

Interpretation: there is meaningful ARM64 HAL engineering, but the explicit stub designation and unimplemented paths are blockers for assuming complete platform support. The remaining HAL work must be enumerated function-by-function and validated on the target.

### 3.3 Display and graphics

- Upstream README declares UEFI/GOP framebuffer and early display support.
- `drivers/directx/rpi5vc4/rpi5vc4.c` identifies itself as a Raspberry Pi 5 WDDM display-only miniport for the BCM2712 HVS/PixelValve pipeline.
- Its documented display path acquires the firmware GOP framebuffer through `DxgkCbAcquirePostDisplayOwnership` and installs its own HVS display list.
- This is board-specific display evidence, not a universal ARM64 GPU driver or mobile compositor.

### 3.4 Touch and input

- `modules/rostests/winetests/dinput/device8.c` contains an explicit skip message: `HID touchscreen unsupported, skipping tests`.
- This test result is a warning about the tested DirectInput/HID path; it does not prove that every possible touch controller or driver is absent.
- No phone-specific touchscreen controller, multitouch stack, gesture policy, or orientation integration was established by this audit.

### 3.5 Power and mobile peripherals

- PSCI system reset/off and CPU_ON code is present. These are firmware-level power/processor operations, not a complete phone power-management policy.
- The inspected evidence does not establish battery gauge/charging drivers, thermal policy, phone-grade suspend/resume and wake sources, cellular modem/SIM/telephony/SMS, camera/ISP, GPS, or a supported smartphone sensor stack.
- Generic USB, storage, ACPI, and networking components must be mapped to an exact target board before claiming support.

## 4. Capability matrix

| Capability | Evidence at pinned revision | LPNT assessment |
| --- | --- | --- |
| ARM64 build target | Explicit upstream README declaration | Upstream-declared; build not run by LPNT |
| ARM64 bootloader | ARM64 loader source and ACPI/PSCI handoff code | Source confirmed; board boot untested |
| ARM64 HAL | GIC, DMA, PCI routing, PSCI code; file explicitly calls itself a minimal stub and has unimplemented returns | Partial/in-progress; major risk |
| Secondary CPU startup | PSCI CPU_ON and fallback logic in ARM64 startup source | Source confirmed; hardware behavior untested |
| UEFI / GOP | Upstream README declaration and GOP framebuffer paths | Early display path; not a mobile graphics stack |
| Raspberry Pi 5 display | RPi5 WDDM display-only miniport using GOP handoff and HVS | Board-specific source exists; not tested by LPNT |
| Touchscreen | DirectInput test skips HID touchscreen as unsupported | Unverified; high-risk gap |
| USB / SD / eMMC | Upstream README feature declarations | Target-specific validation required |
| Cellular / SIM / telephony / SMS | No phone-specific evidence established | Unknown / major gap |
| Camera / ISP | No phone-specific evidence established | Unknown / major gap |
| Sensors / GPS | No phone-specific evidence established | Unknown / major gap |
| Battery / charging / thermal | No phone-specific mobile policy established | Unknown / major gap |
| Suspend / resume / wake | Some general power framework and PSCI operations exist; no phone-grade path established | Unknown / major gap |

## 5. Target strategy recommendation

Do not select a smartphone yet. Based on the evidence inspected, use a staged target strategy:

1. **First bring-up candidate: Raspberry Pi 5** — it has explicit upstream board/display evidence, making it the strongest current ARM64 candidate for validating the HAL and graphics path.
2. **Virtual ARM64 environment** — useful for repeatable bootloader/kernel experiments, but it cannot validate real peripherals or mobile power.
3. **Smartphone candidate** — defer selection until boot firmware, device tree/ACPI, display, touch, storage, UART, power, and driver licensing are mapped for one exact model and revision.

This is a recommendation for investigation, not a claim that Raspberry Pi 5 is already fully supported or selected as the official LPNT target.

## 6. Risks and constraints

1. The ARM64 HAL is explicitly described as a minimal stub; unimplemented paths must be mapped before product-layer work depends on them.
2. GOP is a firmware framebuffer handoff, not proof of accelerated composition, panel control, touch integration, or power-efficient rendering.
3. The RPi5 graphics path is hardware-specific and cannot be generalized to unrelated smartphone SoCs.
4. Touchscreen evidence is insufficient and includes an explicit unsupported test skip.
5. Architecture support does not imply peripheral, modem, camera, sensor, or power-management support.
6. The pinned upstream revision must be refreshed deliberately; findings are valid only for the audited revision.

## 7. Remaining audit tasks

- [ ] Build LiberNT ARM64 from the pinned revision and retain full build logs.
- [ ] Boot the ARM64 image in a documented virtual machine and record boot milestones.
- [ ] Boot on Raspberry Pi 5 and validate UART, storage, display, USB, and input independently.
- [ ] Enumerate every ARM64 HAL stub and `STATUS_NOT_IMPLEMENTED` path with call sites and impact.
- [ ] Trace GIC, timer, interrupt-controller, ACPI/device-tree, and DMA/SMMU initialization end-to-end.
- [ ] Inventory actual touch/input drivers and determine whether a supported HID or I2C multitouch path exists.
- [ ] Inspect RPi5 display driver initialization and test whether it supports stable interactive rendering.
- [ ] Inspect board power, thermal, suspend/resume, wake, and battery-related support.
- [ ] Reassess smartphone candidates only after a concrete board baseline is validated.

## 8. Audit limitations

This is a source-level review, not a build, boot, performance, or hardware validation. No upstream code was copied into LPNT. No phone hardware was tested. No smartphone compatibility claim is made. Upstream source statements and source presence are kept separate from tested capabilities.

## 9. Source references

- Pinned upstream revision: https://github.com/LiberNT/LiberNT/commit/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9
- Upstream README: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/README.md
- ARM64 HAL: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/ntos/hal/armcommon/halarm64.c
- ARM64 bootloader: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9bc9253088959a6cd1fe5a9956bec9/bootloader/arch/arm64/winldr.c
- ARM64 processor startup: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9bc9253088959a6cd1fe5a9956bec9/ntos/hal/arm64/start.c
- Raspberry Pi 5 display miniport: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9bc9253088959a6cd1fe5a9956bec9/drivers/directx/rpi5vc4/rpi5vc4.c
- DirectInput touchscreen test: https://github.com/LiberNT/LiberNT/blob/20ac4b4b9bc9253088959a6cd1fe5a9956bec9/modules/rostests/winetests/dinput/device8.c
