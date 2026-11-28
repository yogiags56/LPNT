# ARM64 HAL Deep Audit — `halarm64.c`

**Audit type:** Source-level function inventory  
**Upstream repository:** [LiberNT/LiberNT](https://github.com/LiberNT/LiberNT)  
**Pinned revision:** `20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9`  
**Audited file:** `ntos/hal/armcommon/halarm64.c`  
**Source file blob:** `72db8e402887927fda98d752f43133373c098d8f`  
**Audit date:** 2026-11-28  
**Validation:** Static source review only; no build, boot, or device tests performed.

## 1. Executive finding

The file contains substantial ARM64-specific code for GIC interrupt handling, PCI configuration-space access, MSI routing, physical/MMIO mapping, PSCI, and DMA operations. It is not accurate to describe the entire file as empty or nonfunctional.

At the same time, several HAL entry points are explicit stubs or return unimplemented status. PCI bus-handler/PnP integration is specifically called out by the source as incomplete. This makes the current HAL unsuitable for assuming complete ARM64 device enumeration or production use without additional implementation and validation.

This report is deliberately scoped to the single file named above. It is not a complete inventory of every ARM64 HAL source file or every dependency called by this file.

## 2. Explicit incomplete entry points

| Function / area | Source evidence | Likely impact | Priority |
| --- | --- | --- | --- |
| `HalAdjustResourceList` | Ignores its input, invokes `UNIMPLEMENTED_STUB()`, returns `STATUS_NOT_IMPLEMENTED` (around line 2860) | HAL resource requirement adjustment cannot be relied upon for devices that require this path | High |
| `HalAllocateCrashDumpRegisters` | Ignores both parameters, invokes `UNIMPLEMENTED_STUB()`, returns `NULL` (around line 3324) | Crash-dump DMA/map-register path is unavailable through this entry point | Medium |
| `HalpAssignSlotResources` | Ignores every parameter and returns `STATUS_NOT_IMPLEMENTED` (around line 3336) | Internal slot-resource assignment is unavailable | High |
| `HalAssignSlotResources` | Ignores every parameter, invokes `UNIMPLEMENTED_STUB()`, returns `STATUS_NOT_IMPLEMENTED` (around line 3359) | HAL slot-resource assignment is unavailable to callers | High |
| PCI bus-handler integration | Source comment explicitly says ARM64 implementations log ACPI PCI configuration but do not integrate with bus-handler infrastructure; TODO requests full PnP integration (around line 1383) | PCI config access and ACPI root discovery may exist, but full PCI bus PnP/resource integration is not established | Critical |
| `HalpRegisterPciDebuggingDeviceInfo` | Empty function body (around line 1377) | PCI debugging-device registration is absent; debugger-related functionality may be affected | Low |
| `HaliSetSystemInformation` | Handles `HalRegisterSecondaryInterruptInterface`; other information classes log as unhandled and return `STATUS_NOT_IMPLEMENTED` (around line 1110) | Only a narrow subset of the system-information setter contract is implemented | Medium |

Line numbers refer to the pinned source revision and can shift in later upstream commits.

## 3. PCI findings — partial implementation, not total absence

The PCI-related section must be interpreted carefully:

- `HalpConfigurePciRootBridge` stores ACPI root-bridge data in an internal array and logs segment, bus range, configuration-space base, windows, and _OSC information.
- PCI configuration-space access helpers are present, including ECAM/MMCONFIG and root-bridge access paths.
- A PCI routing query callback can be registered.
- MSI message address generation has ITS, GICv2m, and platform-doorbell paths.
- MSI vector-range reporting checks the available ITS/GICv2m/doorbell facilities.

However:

- The source itself explicitly identifies missing full PCI bus-handler integration for PnP.
- `HalpRecordPciMaxGsi` logs route information.
- `HalpSetPciRoutingMap` scans entries to compute/log the maximum GSI but does not persist the supplied routing table in this function.
- `HalpRegisterPciDebuggingDeviceInfo` is empty.
- Slot-resource assignment functions return `STATUS_NOT_IMPLEMENTED`.

**Assessment:** low-level PCI configuration and parts of interrupt/MSI handling exist, but this is not proof of complete PCI device enumeration, resource arbitration, and driver start on arbitrary ARM64 platforms.

## 4. DMA findings — real implementation with important boundaries

The file contains a substantive DMA implementation:

- `HalpArm64AllocateMapRegisterBase` allocates map-register metadata.
- `HalAllocateAdapterChannel` validates parameters, allocates map-register metadata, raises to DISPATCH_LEVEL as needed, invokes the driver's execution callback, and processes the callback action.
- `HalpArm64TranslateDmaAddress` handles identity translation and ACPI DMA windows, including overflow and device-address-width checks.
- `HalpArm64AllocateDmaMemory` allocates contiguous memory constrained by DMA windows/address limits.
- `HalpConfigureDmaAdapter` validates and records DMA windows.
- `HalpSetDmaAdapterCoherency` records adapter coherency state.
- `HalAllocateCommonBuffer` allocates and tracks common buffers, selecting cache behavior based on coherency information.

Important limitations:

1. `HalAllocateCrashDumpRegisters` remains an explicit stub.
2. Correct DMA behavior depends on firmware-provided DMA windows, address widths, and coherency information being accurate.
3. SMMU/IOMMU translation and device-specific DMA behavior require validation on the selected board.
4. A successful allocation path in source does not establish that every controller driver uses the adapter and DMA contract correctly.

The `FALSE` and `NULL` returns in DMA helpers are generally validation, allocation-failure, overflow, address-window, or unsupported-range exits. They must not all be counted as unimplemented functions.

## 5. Interpreting FALSE and NULL returns

A mechanical search finds many `return FALSE;` and `return NULL;` statements. These are not equivalent to stubs.

The inspected code uses them for distinct, often intentional conditions:

- Invalid or missing function parameters.
- PCI configuration backend/root bridge not found or an access failure.
- MSI/ITS/GICv2m route unavailable, vector invalid, or allocation failure.
- MMIO mapping failure during runtime address-space conversion.
- Physical mapping request invalid or `MmMapIoSpace` failure.
- DMA address overflow, no matching DMA window, address-width violation, or allocation failure.
- Phase-0 processor-start preparation failure.

Examples:

- `HalpMapRuntimeMmioWindow` returns FALSE when an MMIO mapping fails; this is a real error path, not an empty implementation.
- `HalpArm64TranslateDmaAddress` returns FALSE for invalid ranges or when no DMA window covers the transfer.
- `HalGetMsiVectorRange` returns FALSE when no supported MSI mechanism is available.
- `HalpMapPhysicalMemory64` returns NULL for invalid input or failed mapping.

A separate call-site review is required to determine whether each failure is safely propagated or can lead to a boot failure, device-start failure, or bugcheck.

## 6. Prioritized remediation backlog

### P0 — Platform enumeration and device-start correctness

- [ ] Map the complete ARM64 PCI bus-handler/PnP call graph.
- [ ] Compare the ARM64 PCI path with the x86 implementation and identify reusable contracts without copying x86 assumptions blindly.
- [ ] Design ARM64 bus-handler registration and resource translation around ACPI root bridges, ECAM/_CBA, bus ranges, IO/memory windows, and interrupt routing.
- [ ] Implement and test resource assignment only after the bus-handler contract is understood.
- [ ] Test PCI device enumeration and driver start on the chosen board/VM.

### P1 — Resource and DMA contract completion

- [ ] Trace all callers of `HalAdjustResourceList` and define correct ARM64 resource semantics.
- [ ] Trace all callers of `HalAssignSlotResources` and `HalpAssignSlotResources`.
- [ ] Define crash-dump DMA requirements and implement `HalAllocateCrashDumpRegisters` consistently with the ARM64 DMA adapter design.
- [ ] Test DMA windows, address-width limits, non-coherent cache maintenance, SMMU/IOMMU, and common-buffer lifecycle.

### P2 — Debug and interface completeness

- [ ] Determine whether `HalpRegisterPciDebuggingDeviceInfo` is required for the selected debug/recovery strategy.
- [ ] Inventory every supported and unsupported `HAL_SET_INFORMATION_CLASS` value in `HaliSetSystemInformation`.
- [ ] Add tests for unsupported classes and ensure callers handle the returned status.

### P3 — Failure-path verification

- [ ] Audit all FALSE/NULL exits in this file against their callers.
- [ ] Add diagnostics and tests for invalid parameters, absent hardware, failed mappings, DMA range violations, and missing MSI facilities.
- [ ] Confirm fatal initialization failures are surfaced with actionable logs and do not leave partially initialized hardware state.

## 7. Recommended implementation order

1. Build a call graph for the PCI root bridge → bus handler → resource assignment → device enumeration/start sequence.
2. Compare ARM64 PCI contracts with existing x86 and RISC-V implementations at the same upstream revision.
3. Specify the ARM64 resource model and tests before implementing missing stubs.
4. Complete PCI bus-handler/PnP integration.
5. Complete resource assignment and adjustment entry points.
6. Complete crash-dump DMA behavior.
7. Validate on a reproducible ARM64 VM and one real board.

Do not start by replacing every `return FALSE` or `return NULL` with success. That would hide real hardware and validation failures.

## 8. Source references

All source links below are pinned to the audited upstream revision:

- [ARM64 HAL source](https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/ntos/hal/armcommon/halarm64.c)
- [ARM64 HAL header](https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/ntos/hal/armcommon/include/hal.h)
- [x86 DMA reference implementation](https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/hal/halx86/generic/dma.c)
- [ARM64 HAL ACPI integration](https://github.com/LiberNT/LiberNT/blob/20ac4b4b9b4bc9253088959a6cd1fe5a9956bec9/ntos/hal/armcommon/acpi/halacpi.c)

## 9. Audit boundary

This document records source-level evidence only. It does not claim that the functions were built, executed, or tested on physical hardware. The audit covers `halarm64.c`; additional ARM64 HAL files, architecture-specific ACPI modules, GIC code, kernel callers, and driver dependencies remain to be audited.
