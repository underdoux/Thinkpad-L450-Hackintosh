# Lenovo Thinkpad L450 - OpenCore Setup

```
OS: macOS Monterey
Host: Hackintosh (SMBIOS: MacBookPro12,1)
CPU: Intel i5-5300U (4) @ 2.30GHz
GPU: Intel HD Graphics 5500
Memory: 16GB DDR3 1600MHz
OpenCore: 1.0.4
```

## Latest Update Changes
- Updated to OpenCore 1.0.4
- Updated required drivers:
  - OpenRuntime.efi
  - OpenCanopy.efi
  - OpenHfsPlus.efi (replaces HfsPlus.efi)
- Removed deprecated ApfsDriverLoader.efi (integrated into OpenCore)
- Added new security features for Monterey

## Functionality

If a function is not explicitly mentioned below, it is most likely working.

|Function|Status|
|--------|------|
|Volume/brightness keys after wake from sleep|Not working, WIP|
|Battery status after sleep|Laggy, WIP|
|Audio jack|Working with layout-id 17|
|mDP Output|Working with HDMI 2.0 patch|
|Card reader|Working with Sinetek-rtsx.kext|
|WiFi|Working with AirportItlwm.kext|
|Bluetooth|Working with genuine Apple BCM94360CS|

## Troubleshooting Roadmap

### 1. Volume/Brightness Keys After Sleep
Current Status: Not Working
Investigation Plan:
- [ ] Check SSDT-KBD.aml implementation
- [ ] Verify BrightnessKeys.kext is present and loading
- [ ] Test alternative ACPI patches for keyboard mapping
- [ ] Monitor IORegistry during sleep/wake cycle
Next Steps:
1. Update VoodooPS2Controller.kext to latest version
2. Implement SSDT-KEYS patch
3. Add KeySupport="Yes" in config.plist
4. Test with different keyboard layouts in macOS

### 2. Battery Status Lag
Current Status: Laggy after sleep
Investigation Plan:
- [ ] Review current SSDT-BATT.aml implementation
- [ ] Check SMCBatteryManager.kext version
- [ ] Monitor battery information reporting
- [ ] Test ECEnabler.kext implementation
Next Steps:
1. Update SMCBatteryManager.kext
2. Implement updated battery patches from RehabMan guide
3. Add ECEnabler.kext
4. Test different ACPI battery reporting methods

### 3. Sleep/Wake Optimization
Current Status: Affects multiple functions
Investigation Plan:
- [ ] Review DSDT sleep methods
- [ ] Check power management settings
- [ ] Monitor wake triggers
- [ ] Test different sleep modes
Next Steps:
1. Implement SSDT-SLEEP patch
2. Update power management kexts
3. Test hibernation fixes
4. Monitor sleep/wake logs

### 4. Graphics Performance
Current Status: Working but could be optimized
Investigation Plan:
- [ ] Review current framebuffer patches
- [ ] Check WhateverGreen.kext configuration
- [ ] Monitor external display handling
- [ ] Test different IGPU properties
Next Steps:
1. Update WhateverGreen.kext
2. Implement optimized framebuffer patches
3. Test different display configurations
4. Monitor graphics performance metrics

## Known Issues & Troubleshooting for macOS Monterey

### Common Issues:
1. **Sleep/Wake Issues**
   - Battery status may be laggy after wake
   - Volume/brightness keys may not respond after wake
   - Fix: Reset SMC and NVRAM if issues persist

2. **Graphics Issues**
   - External display not detected
   - Fix: Verify HDMI/mDP connection and refresh framebuffer with Cmd+Alt+Eject

3. **Audio Issues**
   - No sound after update/install
   - Fix: Reset audio layout with `alcid=17` in boot-args

4. **WiFi/Bluetooth Issues**
   - WiFi not working after update
   - Fix: Ensure AirportItlwm.kext matches your macOS version

### Error Codes:
- `OCB: LoadImage failed - Security Violation`: Enable `SecureBootModel` in config.plist
- `Kernel Panic - not syncing`: Verify Lilu.kext loads first
- `-v keepsyms=1 debug=0x100`: Add these boot-args for verbose mode debugging

## BIOS Settings

|Config|Security|Startup|Restart|
|------|--------|-------|-------|
|<ul><li>Network<ul><li>All [Disabled]</li></ul></li><li>USB<ul><li>USB UEFI BIOS Support [Enabled]</li><li>USB 3.0 Mode [Enabled]</li></ul></li><li>Keyboard/Mouse<ul><li>Trackpoint [Enabled]</li><li>Trackpad [Enabled]</li></ul></li><li>Display<ul><li>Total Graphics Memory [512MB]</li></ul></li><li>CPU<ul><li>Intel (R) Hyper-Threading Technology [Enabled]</li></ul></li></ul>|<ul><li>Security Chip<ul><li>Security Chip [Disabled]</li></ul></li><li>Memory Protection<ul><li>Execution Prevention [Enabled]</li></ul></li><li>Virtualization<ul><li>All [Enabled]<ul><li>VT-d is safe because of OpenCore's `DisableIOMapper`</li></ul></li></ul></li><li>Secure Boot<ul><li>Secure Boot [Disabled]</li></ul></li></ul>|<ul><li>Startup<ul><li>UEFI/Legacy Boot [UEFI Only]<ul><li>CSM Support [No]</li></ul></li></ul></li></ul>|<ul><li>Restart<ul><li>OS Optimized Defaults [Enabled]</li></ul></li></ul>|

## Installation Steps

1. Download OpenCore 1.0.4 from [OpenCore](https://github.com/acidanthera/OpenCorePkg/releases).

2. Required EFI drivers (already included in this repo):
   - OpenRuntime.efi
   - OpenCanopy.efi
   - OpenHfsPlus.efi

3. Generate new SMBIOS data with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) using MacBookPro12,1.

4. Copy the EFI folder to your EFI partition.

### Additional Steps for Monterey:

1. Update all kexts to their latest versions
2. Ensure SecureBootModel is set to "j137" in config.plist
3. Add `-lilubetaall` to boot-args if using beta kexts
4. Set MinKernel to "20.0.0" for Monterey compatibility
5. Enable required patches for Monterey in config.plist

## Credits

* [jsassu20](https://github.com/jsassu20/Lenovo-T450-Catalina-OpenCore),  [EchoSpirit](https://github.com/EchoEsprit/Hackintosh-Catalina-OpenCore-Lenovo-T450s-efi), and [Sniki](https://www.tonymacx86.com/threads/guide-lenovo-thinkpad-t440s-using-clover-uefi-hotpatch.279492/), whose configs were studied to put together this build
* acidanthera for OpenCorePkg et all
* khronokernel, for providing [the vanilla guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/) which this config.plist is based on
* RehabMan, for his guides on [patching ACPI for battery status](https://www.tonymacx86.com/threads/guide-how-to-patch-dsdt-for-working-battery-status.116102/) and [converting those patches for hotpatching](https://www.tonymacx86.com/threads/guide-using-clover-to-hotpatch-acpi.200137/)
* [CorpNewt](https://github.com/CorpNewt), for Lilu-and-Friends, ProperTree, GenSMBIOS, and SSDTTime