# openwrt-Ubiquity-Aircube-notes
Notes regarding a port of openwrt to the Aircube ISP

I think this may help you, from reading your notes here

https://github.com/c-mauderer/openwrt-Ubiquity-Aircube-notes/blob/master/notes/Notes.asciidoc

## 1

```
You can downgrade uboot once you get in with the serial console, `urescue -f -e`.
```

## 2

When trying to use a new version of AirOS or openwrt to reflash an old version of AirOS just use the old AirOS binary flash tool (THAT IS ALREADY INSTALLED ON THE AIROS FIRMWARE)

The issue is that they are releasing new firmwares with different flash layouts and basically the uboot tftp loaded as well as the web ui won't allow LEDE or OpenWRT images to be flashed.

As we know the devices are the same hardware, which means the same binaries will run.

So, regardless of the AirOS version we can scp the binLEDEary which is the updater and then flash our openwrt image.

```
binwalk -e XW.v5.6.15.30572.170328.1052.bin
scp _XW.v5.6.15.30572.170328.1052.bin.extracted/squashfs-root/bin/ubntbox ubnt@10.103.68.69:/tmp/
scp XW.v5.6.15.30572.170328.1052.bin ubnt@10.103.68.69:/tmp/fwupdate.bin
ssh ubnt@10.103.68.69
cd /tmp; /tmp/ubntbox fwupdate.real -m fwupdate.bin
```

This is important because the older AIROS versions didn't expect signing in uboot.

## 3

You can install `mtd` on AIROS by taking the executable from an older version of AIROS (dissect the binary) if `mtd` is not already on the version of AIROS currently on the device.  Make sure the AIROS build is for ath79 not ath79.

first ssh ubnt@192.168.1.20

check if mtd exists, if not, download the 3.7.58.6385 firmware that's here in mindmap so it can be downgraded to include mtd.

then

```
scp BZ.......bin ubnt@192.168.1.20:/tmp/
ssh ubnt@192.168.1.20
cd /tmp
fwupdate.real -m BZ.....bin
```

### once you have mtd in airos

```
BZ.v3.7.40# mtd write /tmp/openwrt-ar71xx-generic-ubnt-unifiac-lite-squashfs-sysupgrade.bin kernel0
Unlocking kernel0 ...
Erasing kernel0 ...
Writing from /tmp/openwrt-ar71xx-generic-ubnt-unifiac-lite-squashfs-sysupgrade.bin to kernel0 ...  [e/w]

BZ.v3.7.40# mtd erase kernel1
Unlocking kernel1 ...
Erasing kernel1 ...
```

Then write a nullbyte to the bs partition (check /proc/mtd for the correct path) to make it boot from kernel0, which is the partition that sysupgrade writes to.

```
BZ.v3.9.19# cat /proc/mtd 
dev:    size   erasesize  name
mtd0: 00060000 00010000 "u-boot"
mtd1: 00010000 00010000 "u-boot-env"
mtd2: 00790000 00010000 "kernel0"
mtd3: 00790000 00010000 "kernel1"
mtd4: 00020000 00010000 "bs"
mtd5: 00040000 00010000 "cfg"
mtd6: 00010000 00010000 "EEPROM"

BZ.v3.9.19# dd if=/dev/zero bs=1 count=1 of=/dev/mtd4
1+0 records in
1+0 records out
1 byte copied, 7.3437e-05 s, 13.6 kB/s
```

## 4

Beyond that, you can always compile static linked binaries in the openwrt buildroot then copy them to the AIROS image to flash using the MTD ioctls.
