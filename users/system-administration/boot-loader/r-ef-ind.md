<!-- TITLE: rEFInd -->
<!-- SUBTITLE: rEFInd as AOSC OS Bootloader -->

---

[^_^]:
    Remove this only if:
		1 You are one of the maintainers of AOSC OS.
		2 You have checked this passage carefully and have already made all necessary changes.
		This passage is migration from ArchWiki by Bobby Rong <rjl931189261@126.com>.

**WARNING:** This passage is written by a user **which doesn't use rEFInd** and **it is NOT checked by the AOSC maintainers yet**. **DO NOT TRY TO TEST THE COMMANDS BELOW UNLESS YOU KNOW WHAT YOU ARE DOING**!
**警告：本文由一个从未使用过 rEFInd 的用户编写，也未经过 AOSC 成员的确认。如果你不知道运行下面的命令后会发生什么，请勿尝试运行这些命令！**

**Contributions are welcomed. 欢迎修改本文。**

---

rEFInd is a UEFI boot manager capable of launching EFISTUB kernels. It is a fork of the no-longer-maintained rEFIt and fixes many issues with respect to non-Mac UEFI booting. It is designed to be platform-neutral and to simplify booting multiple OSes.

> Note: In the entire article esp denotes the mountpoint of the EFI System Partition aka ESP.

# Installation

Follow the installation guide and do all the steps before the chapter [Bootloader Configuration](/users/installation/amd64#bootloader-configuration). Now install the `refind` package:

```
# apt install refind
```

rEFInd has read-only drivers for ReiserFS, Ext2, Ext4, Btrfs, ISO-9660, HFS+, and NTFS. Additionally rEFInd can use drivers from the UEFI firmware i.e. FAT (or HFS+ on Macs or ISO-9660 on some systems).

> Note: Your kernel and initramfs need to reside on a file system which rEFInd can read.

# Configuration

The rEFInd package includes the refind-install script to simplify the process of setting rEFInd as your default EFI boot entry. The script has several options for handling differing setups and UEFI implementations. See `refind-install(8)`. For many systems it should be sufficient to simply run:

```
# refind-install
```

This will attempt to find and mount your ESP, copy rEFInd files to `esp/EFI/refind/`, and use efibootmgr to make rEFInd the default EFI boot application.

Alternatively you can install rEFInd to the default/fallback boot path `esp/EFI/BOOT/bootx64.efi`. This is helpful for bootable USB flash drives or on systems that have issues with the NVRAM changes made by efibootmgr:

```
# refind-install --usedefault /dev/sdXY
```

Where /dev/sdXY is the partition of your ESP.

You can read the comments in the install script for explanations of the various installation options.

By default refind-install installs only the driver for the file system on which kernel resides. Additional file systems need to be installed manually or you can install all drivers with the `--alldrivers` option. This is useful for bootable USB flash drives e.g.:

```
# refind-install --usedefault /dev/sdXY --alldrivers
```

After installing rEFInd's files to the ESP, verify that rEFInd has created refind_linux.conf containing the required kernel parameters (e.g. `root=`) in the same directory as your kernel. If it has not created this file, you will need to set up passing kernel parameters manually or you will most likely get a kernel panic on your next boot.

By default, rEFInd will scan all of your drives (that it has drivers for) and add a boot entry for each EFI bootloader it finds, which should include your kernel (since Arch enables EFISTUB by default). So you may have a bootable system at this point.

> Tip: It is always a good idea to edit the default config `esp/EFI/refind/refind.conf` to ensure that the default options work for you.

>Warning: When refind-install is run in chroot (e.g. in live system when installing Arch Linux) `/boot/refind_linux.conf` is populated with kernel options from the live system not the one on which it is installed. You need to adjust kernel options in `/boot/refind_linux.conf` manually.

# Upgrading

If `refind-install` worked for your original installation of rEFInd, you can rerun it to copy the updated files. The new config file will be copied as `refind.conf-sample` so that you can integrate changes into your config file using a diff tool.