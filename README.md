# Lenovo Thinkpad L450 - OpenCore Setup

```
OS: macOS Monterey
Host: Hackintosh (SMBIOS: MacBookPro12,1)
CPU: Intel i5-5300U (4) @ 2.30GHz
GPU: Intel HD Graphics 5500
Memory: 16GB DDR3 1600MHz
```

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

## To Use

0. Clone this repo:

```shell
$ git clone https://github.com/Lacedaemon/Thinkpad-L450-OpenCore
```

1. Per the `[Repo]/EFI/OC/config.plist`, download OpenCore 0.9.6 or later from [OpenCore](https://github.com/acidanthera/OpenCorePkg/releases).
 * *If the version used in this repo is newer than the latest prebuilt release, OpenCore must be compiled from source:*

 ```shell
$ git clone https://github.com/acidanthera/OpenCorePkg
$ cd [path to cloned repo]
$ ./macbuild.tool
 ```

2. From the downloaded OpenCore package, extract the following files to their respective places in the cloned repo:
 * `[OpenCore]/EFI/BOOT/BOOTx64.efi` -> `[Repo]/EFI/BOOT/BOOTx64.efi`
 * `[OpenCore]/EFI/OC/OpenCore.efi` -> `[Repo]/EFI/OC/OpenCore.efi`
 * `[OpenCore]/EFI/OC/Drivers/OpenRuntime.efi` -> `[Repo]/EFI/OC/Drivers/OpenRuntime.efi`
 * `[OpenCore]/EFI/OC/Drivers/OpenCanopy.efi` -> `[Repo]/EFI/OC/Drivers/OpenCanopy.efi`
3. Download and extract the latest compatible `ApfsDriverLoader.efi` from [AppleSupportPkg](https://github.com/acidanthera/AppleSupportPkg/releases) to `[Repo]/EFI/OC/Drivers`.
4. Download and copy the latest compatible `HfsPlus.efi` from [OcBinaryData](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) to `[Repo]/EFI/OC/Drivers`.
5. Follow the README's in the `[Repo]/EFI/OC/ACPI` and `[Repo]/EFI/OC/Kexts` folders.
6. Generate new SMBIOS data with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) using MacBookPro12,1.
7. Copy the finished `[Repo]/EFI` folder to a FAT32 partition (e.g. a USB drive or EFI System Partition), then boot from it.

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

## Contact

For general questions regarding this build, I can be reached at:

* Discord: `@HackinDoge#9460`
* Reddit: [/u/HackinDoge](https://reddit.com/u/HackinDoge)
* OSx86.net: [HackinDoge](https://www.osx86.net/profile/345012-hackindoge/)

Please ensure that you've read through all the linked materials before reaching out.

If there is a critical misconfiguration, error, or other issue with any of the files in this build, or if there is a file(s) that need to be added, please open an Issue here on GitHub so I can get that resolved.
