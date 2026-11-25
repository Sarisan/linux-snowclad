## Description

Highly hardened and tailored Linux Kernel for Intel laptops

This kernel also includes fixes for vswing/preemph tables  
Support for Goodix GXFP5130 eSPI fingerprint reader  
Burst-Oriented Response Enhancer CPU Scheduler  
Intel Xe2 Graphics driver default for all devices  

## Support

Currently supported laptops

* HUAWEI MateBook D 14 2023 (MDF-XX)
* HUAWEI MateBook D 14 2024 (MDG-XX)

## External

List of external hardware support and features enabled

* IA32 Emulation
* Virtualization
* Kernel-based Virtual Machine (KVM) support
* Clang ThinLTO
* Default TCP congestion control BBR
* RFCOMM protocol support
* NT synchronization primitive emulation
* Universal TUN/TAP device driver support
* Realtek RTL8152/RTL8153 Based USB Ethernet Adapters
* CDC NCM support
* Huawei NCM embedded AT channel support
* Host for RNDIS and ActiveSync devices
* Allow to specify an EDID data set instead of probing for it
* Dummy BT SCO codec driver
* Wacom Intuos/Graphire tablet support (USB)
* USB Mass Storage support
* USB Attached SCSI
* USB Winchiphead CH341 Single Port Serial Driver
* Android Binder IPC Driver
* Android Binderfs filesystem
* UDF file system support
* exFAT filesystem support
* NTFS Read-Write file system support
* SquashFS 4.0 - Squashed file system support

External hardware support is being added on demand, I don't enable all possible hardware blindly  
If you need some external hardware support, you should open an issue

## Secure Boot

THIS IS FOR LINUX ONLY SETUP  
IF YOU DON'T UNDERSTAND ANYTHING BELOW DO NOT PROCEED TO DO ANYTHING

I highly recommend switching to systemd-boot + ukify with own secure boot key  
Here's the short guide

First install all the mandatory packages
```
apt install python3-cryptography systemd-boot systemd-repart systemd-ukify
```

`/etc/kernel/cmdline`: Put your cmdline here, this is an example from my setup
```
drm.edid_firmware=eDP-1:edid/cmn36b3.bin quiet rd.luks.name=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX=root root=/dev/mapper/root rw splash vt.global_cursor_default=0
```

`/etc/kernel/install.conf`:
```
initrd_generator=dracut
uki_generator=ukify
layout=uki
```

`/etc/kernel/uki.conf`:
```
[UKI]
SecureBootSigningTool=systemd-sbsign
SecureBootPrivateKey=/etc/kernel/secure-boot-private-key.pem
SecureBootCertificate=/etc/kernel/secure-boot-certificate.pem
SignKernel=true
```

Nuke your ESP wherever it's mounted, make sure /boot is not an ESP, e.g. for me it's /efi, usually /boot/efi
```
rm -fr /efi/*
```

Generate secure boot key
```
ukify genkey --secureboot-private-key=/etc/kernel/secure-boot-private-key.pem --secureboot-certificate=/etc/kernel/secure-boot-certificate.pem
```

Sign systemd-boot with your secure boot key
```
/usr/lib/systemd/systemd-sbsign --certificate /etc/kernel/secure-boot-certificate.pem --private-key /etc/kernel/secure-boot-private-key.pem --output /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed sign /usr/lib/systemd/boot/efi/systemd-bootx64.efi
```

Install systemd-boot
```
bootctl install --secure-boot-auto-enroll=yes --certificate=/etc/kernel/secure-boot-certificate.pem --private-key=/etc/kernel/secure-boot-private-key.pem
```

Configure systemd-boot  
`ESP/loader/loader.conf`: ESP is the path to your ESP
```
timeout 0
console-mode keep
secure-boot-enroll force
```

Generate at least one boot entry
```
dpkg-reconfigure linux-image-$(uname -r)
```

Go to UEFI and change Secure Boot to Disabled, this will put it to Setup Mode  
Next time you boot systemd-boot will automatically enroll your secure boot key  
You won't have to repeat any of these unless you're upgrading Debian to next major verison

## Display Overclocking

You can safely overclock display up to 70 Hz  
`cmn36b3.bin` is an EDID data containing additional DTD with 70 Hz

First verify this EDID matches your display  
The quickest way to do it using fastfetch  
If it says CMN36B3 in display section you can proceed further

Place `cmn36b3.bin` to `/usr/lib/firmware/edid`

Configure dracut to include and use this EDID  
`/etc/dracut.conf.d/edid.conf`:
```
kernel_cmdline+=" drm.edid_firmware=eDP-1:edid/cmn36b3.bin "
install_items+=" /lib/firmware/edid/cmn36b3.bin "
```

If you use ukify omit `kernel_cmdline` and append its content to `/etc/kernel/cmdline` instead  
Similarly to secure boot example above

Regenerate current UKI/initrd
```
dpkg-reconfigure linux-image-$(uname -r)
```

After restart you can go to display settings and select 70 Hz

## Building

Clone this repository somewhere and run this command inside it to download Linux Kernel source  
You must have `devscripts` installed

```
uscan --download-current-version
```

You will see a directory named `linux-<version>` in the upper directory  
Go to it and simply run this command to build the packages

```
debuild -us -uc -ui -b
```

When the building is done you will see `.deb` files in the upper directory

## Why

All modern distributions provide generic kernel which prefers legacy and common drivers over modern alternatives and platform specific options, therefore I made this kernel to target specific platform utilizing modern drivers and platform specific options

What benefits will you get from this? In case with this kernel you will notice significantly better performance, smoother experience, faster system boot and significantly better power saving

Also unlike official Debian kernel you will get a seamless BGRT/Plymouth transition using modern EFI DRM instead of legacy EFI Framebuffer
