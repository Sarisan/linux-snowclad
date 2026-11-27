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

Regenerate current UKI/initrd
```
dpkg-reconfigure linux-image-$(uname -r)
```

After restart you can go to display settings and select 70 Hz
