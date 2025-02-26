# macOS 12: Monterey

**Reminder that Dortania and any tools mentioned in this guide are neither responsible for any corruption, data loss, or other ill effects that may arise from this guide, including ones caused by typos. You, the end user, must understand this is beta software on unsupported machines so do not pester developers for fixes. Dortania will not be accepting issues regarding this mini-guide except for typos and/or errors.**
**This guide expects you to have a basic understanding of hackintoshing. If you are not familiar with it, we highly recommend you to wait until there is an easier and more straight-forward solution available.**
**And note that macOS 12, Monterey will require macOS already installed on some machine to create the installer. Windows and Linux users will need to use a macOS VM to create the installer.**

## Table of Contents

[[toc]]

## Prerequisites

### Supported SMBIOS

SMBIOS dropped in Monterey:

* iMac15,x and older
* Macmini6,x and older
* MacBook8,1 and older
* MacBookAir6,x and older
* MacBookPro11,3 and older
  * MacBookPro11,4 and 11,4 are still supported

If your SMBIOS was supported in Big Sur and is not included above, you're good to go!

::: details Supported SMBIOS

* iMac16,1 and newer
* MacPro6,1 and newer
* iMacPro1,1 and newer
* Macmini7,1 and newer
* MacBook9,1 and newer
* MacBookAir7,1 and newer
* MacBookPro11,4 and newer

[Click here](./smbios-support.md) for a full list of supported SMBIOS.

:::

For those on Haswell or Ivy Bridge, here are some simple conversions:

* Ivy Bridge desktops with dGPU should use MacPro6,1
* Haswell desktops with dGPU should use iMac17,1
* Haswell desktops with only an iGPU should use iMac16,2
* Haswell laptops should use MacBookPro11,4 or MacBookPro11,5

### Supported hardware

Dropped GPU Hardware:

* Ivy Bridge (HD 4000 and HD 2500)
* Nvidia Kepler (GTX 6xx/7xx Cards)
* You can use [OpenCore-Legacy-Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher/) to add back support
  * No support is provided for Hackintoshes using OCLP!
  * You will lose access to non-full updates (Small 1-3GB updates)
  * Requires SIP, Apple Secure Boot, and AMFI disabled.

Haswell iGPUs are still supported in Monterey

* Macmini7,1 uses these drivers

### Bluetooth

::: warning

Note that all cards have not been fixed yet, and that bluetooth support is being worked on still.

Do not be suprised if your card does not work, and please be patient!

:::

With Monterey, Apple has completely rewritten the bluetooth stack. As of writing, many bluetooth devices do not work (legacy Broadcom and Intel). With the rewrite, injector kexts break bluetooth support in Monterey, though firmware uploader kexts are still needed. Make sure that you:

* Disable injector kexts
  * IntelBluetoothInjector.kext for Intel cards
  * BrcmBluetoothInjector.kext for Broadcom cards
  * If you still boot Big Sur or older, you can instead set the `MaxKernel` field to `20.99.9` for your injector kext in your config.plist.
* Keep Firmware uploader kexts
  * IntelBluetoothFirmware.kext for Intel
  * BrcmPatchRAM2/3.kext + BrcmFirmwareData.kext for Broadcom
* Add [BlueToolFixup](https://github.com/acidanthera/BrcmPatchRAM/releases)
  * Needed for all non-native Bluetooth devices (Including Intel)
  * If you still boot Big Sur or older, you can set the `MinKernel` field to `21.00.0` to prevent BlueToolFixup loading on older OSes.

See the below issues for more details:

* [BlueToolFixup PR](https://github.com/acidanthera/BrcmPatchRAM/pull/12)
* [Monterey Beta 5+ issues](https://github.com/acidanthera/bugtracker/issues/1821)

### OTA Updates

Starting with Monterey, updates are not delivered to T2 Macs which don't have Secure Boot enabled, and updates do not install properly if your SecureBootModel does not match your machine (ie. non-T2 SMBIOS using j137 or iMacPro1,1 using j160). Hackintoshes which use a T2 SMBIOS **MUST** have OpenCore 0.7.4+ with SecureBootModel set to `Default`. If your SMBIOS does not have a T2 chip, then either `Default` or `Disabled` is ok. More information is available on the [Apple Secure Boot page](https://dortania.github.io/OpenCore-Post-Install/universal/security/applesecureboot.html).

::: tip T2 SMBIOS List

| SMBIOS                                              | Minimum macOS Version |
| :---                                                | :---                  |
| iMacPro1,1 (December 2017)                          | 10.13.2 (17C2111)     |
| MacBookPro15,1 (July 2018)                          | 10.13.6 (17G2112)     |
| MacBookPro15,2 (July 2018)                          | 10.13.6 (17G2112)     |
| Macmini8,1 (October 2018)                           | 10.14 (18A2063)       |
| MacBookAir8,1 (October 2018)                        | 10.14.1 (18B2084)     |
| MacBookPro15,3 (May 2019)                           | 10.14.5 (18F132)      |
| MacBookPro15,4 (July 2019)                          | 10.14.5 (18F2058)     |
| MacBookAir8,2 (July 2019)                           | 10.14.5 (18F2058)     |
| MacBookPro16,1 (November 2019)                      | 10.15.1 (19B2093)     |
| MacPro7,1 (December 2019)                           | 10.15.1 (19B88)       |
| MacBookAir9,1 (March 2020)                          | 10.15.3 (19D2064)     |
| MacBookPro16,2 (May 2020)                           | 10.15.4 (19E2269)     |
| MacBookPro16,3 (May 2020)                           | 10.15.4 (19E2265)     |
| MacBookPro16,4 (June 2020)                          | 10.15.5 (19F96)       |
| iMac20,1 (August 2020)                              | 10.15.6 (19G2005)     |
| iMac20,2 (August 2020)                              | 10.15.6 (19G2005)     |

:::

Note: You do not need the `-revsbvmm` boot argument from RestrictEvents. Use OpenCore 0.7.4 or later.

### Troubleshooting

#### No Updates

Make sure that SIP is enabled. Two bits in SIP specifically cause issues:

* CSR_ALLOW_APPLE_INTERNAL (Bit 4 = 0x10)
  * Prevents updates appearing at all
* CSR_ALLOW_UNAUTHENTICATED_ROOT (Bit 11 = 0x800)
  * Prevents incremental OTA updates

If you want to still have SIP disabled, use either:

* `csrutil disable --no-internal` in Recovery
* A SIP value which does not include the two flags above

To enable SIP:

* Set `csr-active-config` to `<00 00 00 00>` in your config.plist
* Use `csrutil clear` in Recovery
  * Can instead add `csr-active-config` to NVRAM->Delete or reset NVRAM
