Arch installation

1. Partition
iso>> cfdisk /dev/"disk"
efi: "boot partition" 512MB
swap: "swap partition" 16GB
root: "root partition" rest

2. File system creation
iso>> mkfs.btrfs -L "mylabel" -n 32k /dev/"root partition"
iso>> mkfs.fat -f 32 -L "mylabel" /dev/"boot partition"
iso>> mkswap /dev/"swap partition"

3. Mount the file system
iso>> mount /dev/"root partition" /mnt
iso>> mount /dev/"boot partition" /mnt/boot
iso>> swapon /dev/"swap partition"

4. Install essential packages
iso>> pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr vim networkmanager

5. Configure the system
iso>> genfstab -U /mnt >> /mnt/etc/fstab
iso>> cat /mnt/etc/fstab

6. Chroot
iso>> arch-chroot /mnt

7. Timezone
root>> ln -sf /usr/share/zoneinfo/Australia/Melbourne /etc/localtime
root>> date
root>> hwclock --systohc

8. Localization
root>> vim /etc/locale.gen
go to en_US.UTF-8 UTF-8 and uncomment
root>> locale-gen
root>> vim /etc/locale.conf
write "LANG=en_US.UTF-8" and save
root>> vim /etc/vconsole.conf
write "KEYMAP=us"

9. Hostname
root>> vim /etc/hostname
Write "Surface"

10. Set root password
root>> passwd

11. Add user
root>> useradd -m -G wheel -s /bin/bash "username"
root>> passwd "username"
root>> EDITOR=vim visudo
Near the end of the file uncomment for wheel group users

12. Update
root>> su "username"
user>> sudo pacman -Syu
user>> exit

13. Enabling daemons
root>> systemctl enable NetworkManager

14. Bootloader
root>> grub-install --target=x86_64-efi --efi-directory=/boot --botloader-id=GRUB --modules="tpm" --disable-shim-lock
root>> grub-mkconfig -o /boot/grub/grub.cfg

15. Exit
iso>> exit
iso>> umount -a
iso>> reboot


Create Bootable Drive for UEFI Shell
1. Format the drive for fat32
>> mkfs.vfat -F32 /dev/"drive"
2. Mount
>> mkdir /media/usb
>> mount /dev/"drive" /media/usb
>> cd /media/usb
3. Make directory
>> mkdir -p efi/boot/
>> cd efi/boot/
4. Download shell
>> sudo wget -q -O BOOTX64.efi https://github.com/tianocore/edk2/raw/edk2-stable201903/ShellBinPkg/UefiShell/X64/Shell.efi
