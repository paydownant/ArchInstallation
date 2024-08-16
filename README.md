## Arch installation

#### 1. Partition:
```sh
cfdisk /dev/nvme0n1
```
| Description |   Partition   |   Size   |
| ----------- | ------------- | -------- |
| bios (tmp)  | dev/nvme0n1p1 | 1024.0KB |
| efi         | dev/nvme0n1p2 | 550.0MB  |
| root        | dev/nvme0n1p3 | Rest     |

BIOS partition is temporary and will never be used
#### 2. File system creation
```sh
# BOOT
mkfs.btrfs -L root -n 32k /dev/nvme0n1p3
# EFI
mkfs.fat -f 32 -L efi /dev/nvme0n1p1
```
#### 3. Mount the file system
```sh
# Mount to /mnt
mount /dev/nvme0n1p3 /mnt
# Create subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
# Unmount /mnt
umount /mnt

# Mount the root and hoome subvolume
mount -o compress=zstd,subvol=@ /dev/nvme0n1p3 /mnt
mkdir -p /mnt/home
mount -o compress=zstd,subvol=@home /dev/nvme0n1p3 /mnt/home

# Mount the efi partition
mkdir -p /mnt/efi
mount /dev/nvme0n1p2 /mnt/efi
```
#### 4. Install essential packages:
```sh
pacstrap -K /mnt base linux linux-firmware sof-firmware git btrfs-progs base-devel grub grub-btrfs inootify-tools timeshift reflector efibootmgr vim networkmanager
```
#### 5. Configure the system
```sh
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```
You can change contents of fstab for optimisation such as ``noatime`` options
#### 6. Chroot
```sh
arch-chroot /mnt
```
#### 7. Timezone
```sh
ln -sf /usr/share/zoneinfo/Australia/Melbourne /etc/localtime
date
hwclock --systohc
```
#### 8. Localization
```sh
vim /etc/locale.gen
```
go to en_US.UTF-8 UTF-8 and uncomment
``locale-gen``
```sh
vim /etc/locale.conf
```
write ``LANG=en_US.UTF-8`` and save
```sh
vim /etc/vconsole.conf
```
write ``KEYMAP=us``
#### 9. Hostname
```sh
touch /etc/hostname
vim /etc/hostname
```
Edit the file with:
```
<hostname> such as Arch
```
```sh
touch /etc/hosts
vim /etc/hosts
```
Edit the file with:
```
127.0.0.1 localhost
::1 localhost
127.0.1.1 <your host name> such as Arch
```

#### 10. Set root password
```sh
passwd
```
#### 11. Add user
```sh
useradd -m -G wheel -s /bin/bash "username"
passwd "username"
EDITOR=vim visudo
```
Near the end of the file uncomment for wheel group users
#### 12. Update
```sh
su "username"
sudo pacman -Syu
exit
```
#### 13. Enabling daemons
```sh
systemctl enable NetworkManager
```
#### 14. Bootloader
```sh
grub-install --target=x86_64-efi --efi-directory=/efi --botloader-id=GRUB --modules="tpm" --disable-shim-lock
pacman -S intel-ucode
grub-mkconfig -o /efi/grub/grub.cfg
```
#### 15. Restrict /efi permissions
```sh
chmod 700 /efi
```
#### 16. Exit
```sh
exit
umount -a
reboot
```


## Gnome Installation
### 1. Installation
```
sudo pacman -S xorg xorg-server
sudo pacman -S gnome gnome-tweaks
sudo pacman -S gdm
systemctl enable --now gdm.service
```


## Create Bootable Drive for UEFI Shell
#### 1. Format the drive for fat32
```
mkfs.vfat -F32 /dev/"drive"
```
#### 2. Mount
```
mkdir /media/usb
mount /dev/"drive" /media/usb
cd /media/usb
```
#### 3. Make directory
```
mkdir -p efi/boot/
cd efi/boot/
```
#### 4. Download shell
```
sudo wget -q -O BOOTX64.efi https://github.com/tianocore/edk2/raw/edk2-stable201903/ShellBinPkg/UefiShell/X64/Shell.efi
```