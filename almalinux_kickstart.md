# 📀 Custom AlmaLinux ISO image creation guide (Bare metal)

This guide walks through the process of creating a custom AlmaLinux 9.x ISO image with an embedded Kickstart configuration (`ks.cfg`) for automated installation.


## Prepare Bbse ISO file system

```sh
mkdir almalinux_custom
cd almalinux_custom

wget https://repo.almalinux.org/almalinux/9/isos/x86_64/AlmaLinux-9-latest-x86_64-minimal.iso

mkdir almalinux_base mnt

# Mount the ISO and copy contents
sudo mount -o loop AlmaLinux-9-latest-x86_64-minimal.iso mnt
cp -r mnt/* almalinux_base/
sudo umount mnt/
```


## Modify GRUB config

Edit the file located at `almalinux_base/efi/EFI/BOOT/grub.cfg`

Replace the contents with something similar to:
> The version might be different for latest

```
set default="0"

function load_video {
  insmod efi_gop
  insmod efi_uga
  insmod video_bochs
  insmod video_cirrus
  insmod all_video
}

load_video
set gfxpayload=keep
insmod gzio
insmod part_gpt
insmod ext2

set timeout=0
### END /etc/grub.d/00_header ###

search --no-floppy --set=root -l 'AlmaLinux-9-5-x86_64-dvd'

### BEGIN /etc/grub.d/10_linux ###
menuentry 'Install AlmaLinux 9.5' --class fedora --class gnu-linux --class gnu --class os {
	linuxefi /images/pxeboot/vmlinuz inst.text inst.ks=hd:LABEL=AlmaLinux-9-5-x86_64-dvd:/ks.cfg inst.stage2=hd:LABEL=AlmaLinux-9-5-x86_64-dvd
	initrdefi /images/pxeboot/initrd.img
}
```


## Modify ISOLINUX config

Edit the file `almalinux_base/isolinux/isolinux.cfg`

Update it as follows:

```conf
label linux
  menu label ^Install AlmaLinux 9.5
  kernel vmlinuz
  append initrd=initrd.img inst.text inst.ks=hd:LABEL=AlmaLinux-9-5-x86_64-dvd:/ks.cfg inst.stage2=hd:LABEL=AlmaLinux-9-5-x86_64-dvd quiet

label check
  menu label Test this ^media & install AlmaLinux 9.5
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.ks=hd:LABEL=AlmaLinux-9-5-x86_64-dvd:/ks.cfg inst.stage2=hd:LABEL=AlmaLinux-9-5-x86_64-dvd rd.live.check quiet
```


## Add Kickstart file

Create a file named `ks.cfg` at the root of `almalinux_base/`. Example content:

```cfg
# Keyboard layout
keyboard 'us'

# Timezone
timezone Asia/Kolkata --utc

# Language
lang en_US.UTF-8

# Firewall
firewall --enabled --service=mdns

# Repositories
url --url=https://atl.mirrors.knownhost.com/almalinux/9/BaseOS/$basearch/os/
repo --name="appstream" --baseurl=https://atl.mirrors.knownhost.com/almalinux/9/AppStream/$basearch/os/
repo --name="extras" --baseurl=https://atl.mirrors.knownhost.com/almalinux/9/extras/$basearch/os/
repo --name="crb" --baseurl=https://atl.mirrors.knownhost.com/almalinux/9/CRB/$basearch/os/
repo --name="epel" --baseurl=https://dl.fedoraproject.org/pub/epel/9/Everything/$basearch/

# Network
network --activate --bootproto=dhcp --device=link --onboot=on

# SELinux
selinux --enforcing

# Bootloader
bootloader

# Disk partitioning
clearpart --all --initlabel
autopart --type=lvm

# Root password
rootpw admin

# User creation
user --name=blue --password='admin' --iscrypted --gecos="blue" --groups="wheel"

# Packages
%packages
@^minimal-environment
git
htop
NetworkManager-wifi
%end
```

## Build the custom ISO

Ensure you’re inside the `almalinux_base/` directory before running the following:

> Label here should match with what was provided in the grub config

```bash
mkisofs -o ~/iso-image/release.iso \
  -b isolinux/isolinux.bin \
  -J -R -l \
  -c isolinux/boot.cat \
  -no-emul-boot -boot-load-size 4 -boot-info-table \
  -eltorito-alt-boot -e images/efiboot.img -no-emul-boot \
  -graft-points -V "AlmaLinux-9-5-x86_64-dvd" .

# Make ISO hybrid for UEFI boot
isohybrid --uefi ~/iso-image/release.iso

# Embed md5 checksum for verification
implantisomd5 ~/iso-image/release.iso
```