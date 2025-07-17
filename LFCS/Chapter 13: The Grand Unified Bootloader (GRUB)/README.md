# Chapter 13: The Grand Unified Bootloader (GRUB)
Boot process was covered in Chapter 7 <br>
Grub has two major versions <br>
V1 - Legacy <br>
V2 - Current Version <br>

Note: Behaviour of grub is dependent on distribution (on boot you can choose multiple kernels)
## Grub Config Files
The main config file that we might be concerned about is in ``/etc/default/grub`` <br>
```
# GRUB boot loader configuration

GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="Arch"
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
GRUB_CMDLINE_LINUX="rootfstype=ext4"

# Preload both GPT and MBR modules so that they are not missed
GRUB_PRELOAD_MODULES="part_gpt part_msdos"

# Uncomment to enable booting from LUKS encrypted devices
#GRUB_ENABLE_CRYPTODISK=y

# Set to 'countdown' or 'hidden' to change timeout behavior,
# press ESC key to display menu.
GRUB_TIMEOUT_STYLE=menu
...
```
In there we can configure:
- Kernel parameters
- Timeouts
- Background image
- Default kernel to boot
- Many more

Just editing this file will not do anything you have to generate the config (on arch linux it's the command below, for ubuntu it's ``update-grub``) <br>
```
[root@server grub.d]# grub-mkconfig -o /boot/grub/grub.cfg 
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot:  initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

This will generate the ``grub.cfg`` file which is located on boot partition <br>
```
[root@server grub.d]# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                         2:0    1    4K  0 disk 
sda                         8:0    0  200G  0 disk 
├─sda1                      8:1    0    1G  0 part /boot
└─sda2                      8:2    0  199G  0 part /
```

When the boot process is performed, grub doesn't look for ``/etc/default/grub`` but for ``/boot/grub/grub.cfg``

Note: ``/boot/grub/grub.cfg`` is also generated using ``/etc/grub.d/`` files <br>

``/boot/grub/grub.cfg`` is pretty low level <br>
```
menuentry 'Arch Linux' --class arch --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-d01da9df-5477-4557-97b0-a5da5e4c70dc' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_gpt
        insmod fat
        set root='hd0,gpt1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt1 --hint-efi=hd0,gpt1 --hint-baremetal=ahci0,gpt1  3A74-510F
        else
          search --no-floppy --fs-uuid --set=root 3A74-510F
        fi
        echo    'Loading Linux linux ...'
        linux   /vmlinuz-linux root=UUID=d01da9df-5477-4557-97b0-a5da5e4c70dc rw rootfstype=ext4 loglevel=3 quiet
        echo    'Loading initial ramdisk ...'
        initrd  /initramfs-linux.img
}
```
Which is what we would expect from bootloader it performs tasks before even the linux kernel is loaded.