## Arch installation

#### 1. Partition:
```sh
cfdisk /dev/"disk"
```
efi: "boot partition" 512MB<br>
swap: "swap partition" 16GB<br>
root: "root partition" rest<br>
#### 2. File system creation
```sh
mkfs.btrfs -L "mylabel" -n 32k /dev/"root partition"
mkfs.fat -f 32 -L "mylabel" /dev/"boot partition"
mkswap /dev/"swap partition"
```
#### 3. Mount the file system
```sh
mount /dev/"root partition" /mnt
mount /dev/"boot partition" /mnt/boot
swapon /dev/"swap partition"
```
#### 4. Install essential packages:
```sh
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr vim networkmanager
```
#### 5. Configure the system
```sh
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```
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
vim /etc/hostname
```
Write ``Surface``
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
grub-install --target=x86_64-efi --efi-directory=/boot --botloader-id=GRUB --modules="tpm" --disable-shim-lock
grub-mkconfig -o /boot/grub/grub.cfg
```
#### 15. Exit
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