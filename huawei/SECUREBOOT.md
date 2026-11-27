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
