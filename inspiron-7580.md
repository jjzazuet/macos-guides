Dell Inspiron 7568
==================

The following notes can guide you to configure a Dell Inspiron 7568 laptop to execute [macOS Sierra 10.12.3](http://www.apple.com/macos/) with the following hardware profile:

- Main platform: Inspiron 15-7568 (06FF).
- CPU: Intel(R) Core(TM) i7-6500U CPU @ 2.50GHz.
- Memory: 16GB DDR3L-1600Mhz.
- 4K Ultra HD Touchscreen.

## What's working:

- Graphics.
- Internal audio (partially working).
- Wifi module & bluetooth (needs hardware replacement, read below).
- Track pad.
- Battery indicator.
- Display brightness (partially working, see notes below).
- Integrated camera.
- Touch screen (partially working).
- Card reader.

## What's not working:

- HDMI audio (TODO I haven't confirmed with an HDMI TV :P).

## Disclaimer

> As with any OS and hardware mods, you do this at your own risk. I bear no liability whatsoever in case these instructions do not work on your particular hardware. They did with mine, so I'm just sharing with them with you. I'm also not responsible if you cause damage to your hardware by following these instructions.

Thanks.

## Installation

### Wifi hardware mod.

The laptop comes with an intel wifi card which is not natively supported by  macOS. If you wish to have internal wifi connectivity, you can replace it with a Broadcom BCM94352Z wireless module. In my case, the card that I received was not completely physically compatible with the laptop's mini PCIe slot. So I had to grab an exact-o and cut in a third empty space on the left side of the card so that it could fit inside the slot.

### Clover USB boot drive preparation.

Start by configuring Clover to install in UEFI boot mode. The following drivers are required inside the `drivers64UEFI` folder to reach the OS installer.

- `HFSPlus.efi`
- `OsxAptioFixDrv-64.efi`

In addition, the following kernel extensions are required inside the `kexts/other` folder to reach the OS installer, and add support for secondary peripheral devices:

- `FakeSMC.kext` (included with Clover).
- `SATA-100-series-unsupported.kext` (included with Clover).
- `VoodooPS2Controller.kext` from [OS-X-Voodoo-PS2-Controller](https://github.com/RehabMan/OS-X-Voodoo-PS2-Controller). Enables track pad support.
- `IntelBacklight.kext` from [OS-X-Intel-Backlight](https://github.com/RehabMan/OS-X-Intel-Backlight). Enables built in display back light control. Requires Clover `config.plist` changes (read section below).
- `BrcmFirmwareData.kext`, `BrcmPatchRAM2.kext`, `FakePCIID_Broadcom_WiFi.kext` and `FakePCIID.kext` from [OS-X-BrcmPatchRAM](https://github.com/RehabMan/OS-X-BrcmPatchRAM) and [OS-X-Fake-PCI-ID](https://github.com/RehabMan/OS-X-Fake-PCI-ID). Enables wifi support for the Broadcom replacement card.
- `ACPIBatteryManager.kext` from [OS-X-ACPI-Battery-Driver](https://github.com/RehabMan/OS-X-ACPI-Battery-Driver). Enables battery status reporting to the OS.
- `AppleALC.kext` from [AppleALC](https://github.com/vit9696/AppleALC). enables internal audio. Requires Clover `config.plist` changes (read section below).
- `USBInjectAll.kext` from [OS-X-USB-Inject-All](https://github.com/RehabMan/OS-X-USB-Inject-All). Enables integrated webcam and partial touchscreen support.
- `CoreDisplayFixup.kext` from [CoreDisplayFixup](https://github.com/PMheart/CoreDisplayFixup) with `Lilu.kext` from [Lilu](https://github.com/vit9696/Lilu) as a dependency. Enabled 4k display support without patching `CoreDisplay` (pixel clock patch).

### Clover `config.plist` pre-installation changes.

The following changes enable specific device functionality for the laptop's peripherals. these will also be required once the installation is completed and you replicate your Clover boot loader configuration into the machine's internal hard drive. To create your own `config.plist` file for booting, please use the file at the end of this document as a reference to see the exact location of each configuration block.

This file also contains additional patches by user [michmill](https://www.tonymacx86.com/members/michmill.556698/).

> TODO document extra patches from legacy config file.

#### Wifi

Enable kernel patches for the Broadcom card.

```xml
<dict>
  <key>Comment</key>
  <string>Brcm4360 Sierra fvco init</string>
  <key>Disabled</key>
  <false/>
  <key>Find</key>
  <data>gflSqgAAdSk=</data>
  <key>Name</key>
  <string>AirPortBrcm4360</string>
  <key>Replace</key>
  <data>gflSqgAAZpA=</data>
</dict>
```

#### Audio

Enable `layout-id` #3 for internal sound card for [AppleALC](https://github.com/vit9696/AppleALC/wiki/Installation-and-usage).

```xml
<key>Devices</key>
<dict>
  <key>Audio</key>
  <dict>
    <key>Inject</key>
    <integer>3</integer>
  </dict>
</dict>
```

#### Video (pre-installation)

The correct `ig-platform-id` value for this machine is `0x19160000`. However, at installation time, you'll need to intentionally specify an invalid number in this section in order to boot the installer in a fall-back graphics mode for the Intel HD520 card. Otherwise you'll get stuck in the console as described here: [Stuck at DSMOS has arrived](https://www.tonymacx86.com/threads/solved-dell-inspiron-7568-stuck-at-dsmos-as-arrived.206159/).

> Note: you may also set this value at boot time via Clover's patching tools.

```xml
<key>ig-platform-id</key>
<string>0xCAFEBABE</string>
```

#### ACPI DSDT fixes

These enable the software brightness slider control for the built-in display:
```xml
<key>Fixes</key>
<dict>
  <key>AddPNLF_1000000</key>
  <true/>
  <key>FixDarwin_0002</key>
  <true/>
  <key>NewWay_80000000</key>
  <true/>
</dict>
```

This clover configuration should allow you to boot into the OS graphical installer and perform the initial OS setup.

As with most other Hackintosh installation methods, you'll need to:

- Use your USB driver, and boot into the newly installed OS.
- Install Clover one more time (now into your primary boot drive)
- Copy the `config.plist` file which is tailored for this hardware.
- Copy the contents of `drivers64UEFI` and `kexts/other` to the Clover EFI partition.

## Post installation tasks

- First, apply Apple's system updates before configuring the graphics card correctly. This should prevent having to execute the pixel clock patch twice.

- Rebuild network interface list in order to avoid ["Unable to verify device" error message in iTunes/App Store](https://www.tonymacx86.com/threads/your-device-or-computer-could-not-be-verified-contact-support-for-assistance.160158/).

> Go into System Preferences -> Network and remove all interfaces. Apply, then remove `/Library/Preferences/SystemConfiguration/NetworkInterfaces.plist`. Reboot, then add all your network interfaces back, starting with Ethernet.

### Clover `config.plist` post-install changes

#### Graphics card

To get full video acceleration on the HD520 graphics card, first revert back to a valid `ig-platform-id` value:
```xml
<key>ig-platform-id</key>
<string>0x19160000</string>
```

Finally, just rebuild your kernel extension cache, and the device should be ready to operate.

    sudo kextcache -system-prelinked-kernel
    sudo kextcache -system-caches

#### Display brighness

As of macOS 10.12.4, Rehabman's `IntelBacklight` kernel extension is partially working since the display settings under Syste settings does not allow changing display brightness values using the slider control.

https://github.com/RehabMan/OS-X-Intel-Backlight/issues/4

You can instead use this tool to set brightness values via the command line:

https://github.com/nriley/brightness

## Reference Clover `config.plist` system configuration file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ACPI</key>
	<dict>
		<key>DSDT</key>
		<dict>
			<key>Debug</key>
			<false/>
			<key>DropOEM_DSM</key>
			<false/>
			<key>Fixes</key>
			<dict>
				<key>AddPNLF_1000000</key>
				<true/>
				<key>FixDarwin_0002</key>
				<true/>
				<key>NewWay_80000000</key>
				<true/>
			</dict>
			<key>Patches</key>
			<array>
				<dict>
					<key>Comment</key>
					<string>change EHC1 to EH01</string>
					<key>Disabled</key>
					<false/>
					<key>Find</key>
					<data>RUhDMQ==</data>
					<key>Replace</key>
					<data>RUgwMQ==</data>
				</dict>
				<dict>
					<key>Comment</key>
					<string>change EHC2 to EH02</string>
					<key>Disabled</key>
					<false/>
					<key>Find</key>
					<data>RUhDMg==</data>
					<key>Replace</key>
					<data>RUgwMg==</data>
				</dict>
				<dict>
					<key>Comment</key>
					<string>change HDAS to HDEF</string>
					<key>Disabled</key>
					<false/>
					<key>Find</key>
					<data>SERBUw==</data>
					<key>Replace</key>
					<data>SERFRg==</data>
				</dict>
				<dict>
					<key>Comment</key>
					<string>change HECI to IMEI</string>
					<key>Disabled</key>
					<false/>
					<key>Find</key>
					<data>SEVDSQ==</data>
					<key>Replace</key>
					<data>SU1FSQ==</data>
				</dict>
				<dict>
					<key>Comment</key>
					<string>change MEI to IMEI</string>
					<key>Disabled</key>
					<false/>
					<key>Find</key>
					<data>TUVJXw==</data>
					<key>Replace</key>
					<data>SU1FSQ==</data>
				</dict>
			</array>
			<key>ReuseFFFF</key>
			<false/>
		</dict>
		<key>DropTables</key>
		<array>
			<dict>
				<key>Signature</key>
				<string>#MCFG</string>
			</dict>
			<dict>
				<key>Signature</key>
				<string>DMAR</string>
			</dict>
		</array>
		<key>SSDT</key>
		<dict>
			<key>DropOem</key>
			<false/>
			<key>Generate</key>
			<false/>
			<key>PluginType</key>
			<integer>1</integer>
		</dict>
		<key>SortedOrder</key>
		<array>
			<string>SSDT.aml</string>
			<string>SSDT-0.aml</string>
			<string>SSDT-1.aml</string>
			<string>SSDT-2.aml</string>
			<string>SSDT-3.aml</string>
			<string>SSDT-4.aml</string>
			<string>SSDT-5.aml</string>
			<string>SSDT-6.aml</string>
			<string>SSDT-7.aml</string>
			<string>SSDT-8.aml</string>
			<string>SSDT-9.aml</string>
			<string>SSDT-10.aml</string>
			<string>SSDT-11.aml</string>
			<string>SSDT-12.aml</string>
			<string>SSDT-13.aml</string>
			<string>SSDT-14.aml</string>
			<string>SSDT-15.aml</string>
			<string>SSDT-16.aml</string>
			<string>SSDT-17.aml</string>
			<string>SSDT-18.aml</string>
			<string>SSDT-19.aml</string>
			<string>SSDT-XOSI.aml</string>
			<string>SSDT-LPC.aml</string>
		</array>
	</dict>
	<key>Boot</key>
	<dict>
		<key>Arguments</key>
		<string>dart=0 nv_disable=1</string>
		<key>Debug</key>
		<false/>
		<key>Legacy</key>
		<string>LegacyBiosDefault</string>
		<key>NeverHibernate</key>
		<true/>
		<key>Secure</key>
		<false/>
		<key>Timeout</key>
		<integer>5</integer>
		<key>XMPDetection</key>
		<string>Yes</string>
	</dict>
	<key>CPU</key>
	<dict>
		<key>UseARTFrequency</key>
		<false/>
	</dict>
	<key>Devices</key>
	<dict>
		<key>USB</key>
		<dict>
			<key>AddClockID</key>
			<true/>
			<key>FixOwnership</key>
			<true/>
			<key>Inject</key>
			<true/>
		</dict>
		<key>Audio</key>
		<dict>
			<key>Inject</key>
			<integer>3</integer>
		</dict>
	</dict>
	<key>DisableDrivers</key>
	<array>
		<string>VBoxHfs</string>
	</array>
	<key>GUI</key>
	<dict>
		<key>Custom</key>
		<dict>
			<key>Entries</key>
			<array>
				<dict>
					<key>Disabled</key>
					<false/>
					<key>FullTitle</key>
					<string>UEFI internal</string>
					<key>Hidden</key>
					<string>Always</string>
					<key>Ignore</key>
					<false/>
					<key>NoCaches</key>
					<false/>
					<key>Type</key>
					<string>Other</string>
				</dict>
			</array>
		</dict>
		<key>Mouse</key>
		<dict>
			<key>DoubleClick</key>
			<integer>500</integer>
			<key>Enabled</key>
			<false/>
			<key>Mirror</key>
			<false/>
			<key>Speed</key>
			<integer>8</integer>
		</dict>
		<key>Scan</key>
		<dict>
			<key>Entries</key>
			<true/>
			<key>Legacy</key>
			<true/>
			<key>Linux</key>
			<false/>
			<key>Tool</key>
			<true/>
		</dict>
		<key>Theme</key>
		<string>embedded</string>
	</dict>
	<key>Graphics</key>
	<dict>
		<key>Inject</key>
		<dict>
			<key>ATI</key>
			<false/>
			<key>Intel</key>
			<true/>
			<key>NVidia</key>
			<false/>
		</dict>
		<key>NvidiaSingle</key>
		<false/>
		<key>ig-platform-id</key>
		<string>0x19160000</string>
	</dict>
	<key>KernelAndKextPatches</key>
	<dict>
		<key>AppleRTC</key>
		<true/>
		<key>AsusAICPUPM</key>
		<true/>
		<key>Debug</key>
		<false/>
		<key>ForceKextsToLoad</key>
		<array>
			<string>\System\Library\Extensions\IONetworkingFamily.kext</string>
		</array>
		<key>KernelCpu</key>
		<false/>
		<key>KernelHaswellE</key>
		<false/>
		<key>KernelLapic</key>
		<true/>
		<key>KernelPm</key>
		<true/>
		<key>KextsToPatch</key>
		<array>
			<dict>
				<key>Comment</key>
				<string>Brcm4360 Sierra fvco init</string>
				<key>Disabled</key>
				<false/>
				<key>Find</key>
				<data>gflSqgAAdSk=</data>
				<key>Name</key>
				<string>AirPortBrcm4360</string>
				<key>Replace</key>
				<data>gflSqgAAZpA=</data>
			</dict>
		</array>
	</dict>
	<key>RtVariables</key>
	<dict>
		<key>BooterConfig</key>
		<string>0x28</string>
		<key>CsrActiveConfig</key>
		<string>0x67</string>
	</dict>
	<key>SMBIOS</key>
	<dict>
		<key>ProductName</key>
		<string>MacBookPro12,2</string>
		<key>Trust</key>
		<true/>
	</dict>
	<key>SystemParameters</key>
	<dict>
		<key>InjectKexts</key>
		<string>Detect</string>
	</dict>
</dict>
</plist>
```