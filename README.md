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

## USB Installation Guide

### Prerequisites
1. A USB drive (minimum 16GB)
2. InstallAssist.pkg file
3. Access to a working Mac or existing Hackintosh
4. [OpenCore](https://github.com/acidanthera/OpenCorePkg/releases) 1.0.4
5. All required kexts (available in Requirement/Kext folder)

### Creating the USB Installer

1. Format USB Drive:
   ```bash
   # List all disks to find your USB drive
   diskutil list
   
   # Replace N with your USB drive number
   diskutil partitionDisk /dev/diskN GPT JHFS+ "USB" 100%
   ```

2. Create macOS Installer:
   ```bash
   # Create installer (replace 'USB' with your drive name if different)
   sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/USB
   ```

3. Mount EFI Partition:
   ```bash
   # Find the EFI partition
   diskutil list
   
   # Mount EFI (replace N with USB disk number)
   sudo diskutil mount /dev/diskNs1
   ```

4. Install OpenCore:
   - Copy the entire EFI folder from this repository to the USB's EFI partition
   - The structure should be: /Volumes/EFI/EFI/OC/

5. Generate SMBIOS:
   - Use GenSMBIOS tool from Requirement/Tools/GenSMBIOS
   - Run GenSMBIOS.command
   - Select option 1 to download MacSerial
   - Select option 2 to generate SMBIOS
   - Generate for MacBookPro12,1
   - Copy the generated values to config.plist:
     * Serial Number
     * Board Serial
     * SmUUID
     * Apple ROM

6. Verify Installation Files:
   - Check EFI/OC/Kexts contains:
     * Lilu.kext
     * VirtualSMC.kext
     * WhateverGreen.kext
     * VoodooPS2Controller.kext
     * Other required kexts
   - Verify EFI/OC/Drivers has:
     * OpenRuntime.efi
     * OpenCanopy.efi
     * OpenHfsPlus.efi

### BIOS Configuration
Before installation, configure BIOS settings as specified in the BIOS Settings section above.

### Installation Steps

1. Boot from USB:
   - Insert the USB drive
   - Power on the laptop
   - Press F12 for boot menu
   - Select your USB drive (UEFI option)

2. macOS Installation:
   - Select Disk Utility
   - Format your target drive:
     * Name: Monterey (or your preference)
     * Format: APFS
     * Scheme: GUID Partition Map
   - Exit Disk Utility
   - Select Install macOS
   - Follow the installation prompts

3. Post-Installation:
   - Boot into the installed system
   - Mount EFI partition of internal drive:
     ```bash
     # List disks
     diskutil list
     
     # Mount EFI (replace N with internal disk number)
     sudo diskutil mount /dev/diskNs1
     ```
   - Copy EFI folder from USB to internal drive's EFI partition

4. Final Steps:
   - Restart the system
   - Remove the USB drive
   - Verify all functions are working:
     * WiFi/Bluetooth
     * Audio
     * Graphics
     * Keyboard/Trackpad
     * Battery status
     * Sleep/Wake

### Troubleshooting Installation
- If installation fails, boot with -v flag for verbose mode
- If graphics issues occur, try adding igfxonln=1 to boot args
- For kernel panics, verify Lilu.kext loads first in config.plist
- Check error messages in OpenCore boot picker

### Additional Steps for Monterey:

1. Update all kexts to their latest versions
2. Ensure SecureBootModel is set to "j137" in config.plist
3. Add `-lilubetaall` to boot-args if using beta kexts
4. Set MinKernel to "20.0.0" for Monterey compatibility
5. Enable required patches for Monterey in config.plist

## Required Downloads

### Essential Kexts
Download the latest versions from Acidanthera (https://github.com/acidanthera):
- [VoodooPS2Controller.kext](https://github.com/acidanthera/VoodooPS2/releases) - For keyboard and trackpad
- [BrightnessKeys.kext](https://github.com/acidanthera/BrightnessKeys/releases) - For brightness control
- [ECEnabler.kext](https://github.com/1Revenger1/ECEnabler/releases) - For battery status
- [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC/releases) - Includes SMCBatteryManager.kext
- [Lilu.kext](https://github.com/acidanthera/Lilu/releases) - Required for many other kexts
- [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen/releases) - For graphics

### Required Tools
- [ProperTree](https://github.com/corpnewt/ProperTree) - For editing config.plist
- [MaciASL](https://github.com/acidanthera/MaciASL/releases) - For ACPI editing
- [IORegistryExplorer](https://github.com/khronokernel/IORegistryClone/blob/master/ioreg-302.zip) - For system debugging
- [Hackintool](https://github.com/headkaze/Hackintool/releases) - For system information and patching

### Optional but Recommended
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) - For generating SMBIOS data
- [USBMap](https://github.com/corpnewt/USBMap) - For USB port mapping
- [HibernationFixup](https://github.com/acidanthera/HibernationFixup/releases) - For sleep/wake issues

### Installation Steps
1. Download the latest release of each kext/tool
2. For kexts:
   - Extract the .zip files
   - Copy only the .kext files to EFI/OC/Kexts/
   - Update config.plist using ProperTree to add new kexts
3. For tools:
   - ProperTree and MaciASL: Extract and run directly
   - IORegistryExplorer: Copy to Applications folder
   - Hackintool: Install normally as a macOS application

### Important Notes
- Always download the latest stable versions
- Back up your current EFI folder before making changes
- Update config.plist after adding new kexts
- Reset NVRAM after major kext updates

## Credits
* [jsassu20](https://github.com/jsassu20/Lenovo-T450-Catalina-OpenCore),  [EchoSpirit](https://github.com/EchoEsprit/Hackintosh-Catalina-OpenCore-Lenovo-T450s-efi), and [Sniki](https://www.tonymacx86.com/threads/guide-lenovo-thinkpad-t440s-using-clover-uefi-hotpatch.279492/), whose configs were studied to put together this build
* acidanthera for OpenCorePkg et all
* khronokernel, for providing [the vanilla guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/) which this config.plist is based on
* RehabMan, for his guides on [patching ACPI for battery status](https://www.tonymacx86.com/threads/guide-how-to-patch-dsdt-for-working-battery-status.116102/) and [converting those patches for hotpatching](https://www.tonymacx86.com/threads/guide-using-clover-to-hotpatch-acpi.200137/)
* [CorpNewt](https://github.com/CorpNewt), for Lilu-and-Friends, ProperTree, GenSMBIOS, and SSDTTime