# Arch-Btrfs-Installation
วิธีการติดตั้ง Arch Linux บน Btrfs Root File System
loadkeys us
setfont LatGrkCyr-12x22
ip link
dhcpcd enp0s20u1
timedatectl set-ntp true
lsblk -f
```
NAME   FSTYPE   LABEL       UUID                                 MOUNTPOINT
loop0  squashfs                                                  /run/archiso/sfs/airootfs
sda                                                              
sdb    iso9660  ARCH_201708 2017-08-01-13-16-38-00               
├─sdb1 iso9660  ARCH_201708 2017-08-01-13-16-38-00               /run/archiso/bootmnt
└─sdb2 vfat     ARCHISO_EFI EE46-F2B4                            
```
free -h
cfdisk /dev/sda
```
Device:/dev/sda1 Size:512M Type:EFI System
Device:/dev/sda2 Size:100%-Mem_Size Type:Linux filesystem
Device:/dev/sda3 Size:Mem_Size Type:Linux swap
```
mkfs.vfat -F32 -n "EFI" /dev/sda1
mkfs.btrfs -L "ROOT" /dev/sda2
mkswap -L "SWAP" /dev/sda3
lsblk -f
```
NAME   FSTYPE   LABEL       UUID                                 MOUNTPOINT
loop0  squashfs                                                  /run/archiso/sfs/airootfs
sda                                                              
├─sda1 vfat     EFI         0646-5696                            
├─sda2 btrfs    ROOT        5403eee0-5738-4fc0-9ceb-3c826ba80787 
└─sda3 swap     SWAP        30b18d69-b19c-4e6b-a9d7-32c5a62c0b59 
sdb    iso9660  ARCH_201708 2017-08-01-13-16-38-00               
├─sdb1 iso9660  ARCH_201708 2017-08-01-13-16-38-00               /run/archiso/bootmnt
└─sdb2 vfat     ARCHISO_EFI EE46-F2B4                            
```
mount -o noatime,ssd,space_cache,compress=lzo /dev/sda2 /mnt
mount -l | grep /mnt
```
/dev/sda2 on /mnt type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=5,subvol=/) [ROOT]
```
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
btrfs subvolume create @pkg
btrfs subvolume create @snapshots
ls
```
@ @home @pkg @snapshots
```
cd
umount /mnt
mount -o noatime,ssd,space_cache,compress=lzo,subvol=@ /dev/sda2 /mnt
mount -l | grep /mnt
```
/dev/sda2 on /mnt type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@) [ROOT]
```
mkdir /mnt/boot
mkdir /mnt/home
mkdir -p /mnt/var/cache/pacman/pkg
mkdir /mnt/.snapshots
mount -o noatime,ssd,space_cache,compress=lzo,subvol=@home /dev/sda2 /mnt/home
mount -o noatime,ssd,space_cache,compress=lzo,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg
mount -o noatime,ssd,space_cache,compress=lzo,subvol=@snapshots /dev/sda2 /mnt/.snapshots
mount /dev/sda1 /mnt/boot
swapon /dev/sda3
free -h
```
              total        used        free      shared  buff/cache   available
Mem:           7.7G        108M        7.3G        122M        329M        7.2G
Swap:          7.7G          0B        7.7G
```
mount -l | grep /mnt
```
/dev/sda2 on /mnt type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@) [ROOT]
/dev/sda2 on /mnt/home type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=258,subvol=/@home) [ROOT]
/dev/sda2 on /mnt/var/cache/pacman/pkg type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=259,subvol=/@pkg) [ROOT]
/dev/sda2 on /mnt/.snapshots type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=260,subvol=/@snapshots) [ROOT]
/dev/sda1 on /mnt/boot type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro) [EFI]
```
pacstrap /mnt base base-devel bash-completion btrfs-progs connman git snapper wpa_supplicant
genfstab -Lp /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```
# 
# /etc/fstab: static file system information
#
# <file system>	<dir>	<type>	<options>	<dump>	<pass>
# /dev/sda2 UUID=5403eee0-5738-4fc0-9ceb-3c826ba80787
LABEL=ROOT          	/         	btrfs     	rw,noatime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@,subvol=@	0 0

# /dev/sda2 UUID=5403eee0-5738-4fc0-9ceb-3c826ba80787
LABEL=ROOT          	/home     	btrfs     	rw,noatime,compress=lzo,ssd,space_cache,subvolid=258,subvol=/@home,subvol=@home	0 0

# /dev/sda2 UUID=5403eee0-5738-4fc0-9ceb-3c826ba80787
LABEL=ROOT          	/var/cache/pacman/pkg	btrfs     	rw,noatime,compress=lzo,ssd,space_cache,subvolid=259,subvol=/@pkg,subvol=@pkg	0 0

# /dev/sda2 UUID=5403eee0-5738-4fc0-9ceb-3c826ba80787
LABEL=ROOT          	/.snapshots	btrfs     	rw,noatime,compress=lzo,ssd,space_cache,subvolid=260,subvol=/@snapshots,subvol=@snapshots	0 0

# /dev/sda1 UUID=0646-5696
LABEL=EFI           	/boot     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro	0 2

# /dev/sda3 UUID=30b18d69-b19c-4e6b-a9d7-32c5a62c0b59
LABEL=SWAP          	none      	swap      	defaults  	0 0
```
arch-chroot /mnt
echo archbook > /etc/hostname
ln -sf /usr/share/zoneinfo/Asia/Bangkok /etc/localtime
hwclock --hctosys --utc
nano /etc/locale.gen
* Uncomment en_US.UTF-8
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo LANGUAGE=en_US >> /etc/locale.conf
echo KEYMAP=us > /etc/vconsole.conf
nano /etc/pacman.conf
* Uncomment 2 lines [multilib], Include = /etc/pacman.d/mirrorlist
pacman -Syu
pacman -S linux-headers broadcom-wl-dkms dkms intel-ucode
passwd
bootctl install
nano /boot/loader/loader.conf
```
default	arch
timeout 0
editor 0
```
nano /boot/loader/entries/arch.conf
```
title		Arch Linux Rolling
linux		/vmlinuz-linux
initrd		/intel-ucode.img
initrd		/initramfs-linux.img
options		root=LABEL=ROOT rootflags=subvol=@ rw
```
nano /etc/mkinitcpio.conf
* Add 'btrfs' in HOOK example: HOOK='..., btrfs'
mkinitcpio -p linux
exit
umount -R /mnt
reboot
