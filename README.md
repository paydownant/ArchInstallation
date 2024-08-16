## Arch installation

### 1. Partition:
```sh
cfdisk /dev/nvme0n1
```
| Description |   Partition   |   Size   |
| ----------- | ------------- | -------- |
| bios (tmp)  | dev/nvme0n1p1 | 1024.0KB |
| efi         | dev/nvme0n1p2 | 550.0MB  |
| root        | dev/nvme0n1p3 | Rest     |

BIOS partition is temporary and will never be used
### 2. File system creation
```sh
# BOOT
mkfs.btrfs -L root -n 32k /dev/nvme0n1p3
# EFI
mkfs.fat -F 32 /dev/nvme0n1p2
```
### 3. Mount the file system
```sh
# Mount to /mnt
mount /dev/nvme0n1p3 /mnt
# Create subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
# Unmount /mnt
umount /mnt

# Mount the root and hoome subvolume
mount -o noatime,space_cache=v2,ssd,compress=zstd,discard=async,subvol=@ /dev/nvme0n1p3 /mnt
mkdir -p /mnt/home
mount -o noatime,space_chache=v2,ssd,compress=zstd,discard=async,subvol=@home /dev/nvme0n1p3 /mnt/home

# Mount the boot partition
mkdir -p /mnt/boot
mount /dev/nvme0n1p2 /mnt/boot
```
### 4. Install essential packages:
```sh
pacstrap -K /mnt base linux linux-firmware sof-firmware git btrfs-progs base-devel grub grub-btrfs inotify-tools timeshift reflector efibootmgr vim networkmanager zram-generator
```
### 5. Configure the file system
```sh
# Generate
genfstab -U /mnt >> /mnt/etc/fstab
# Check
cat /mnt/etc/fstab
```
You can change contents of fstab for optimisation such as ``noatime`` options
### 6. Chroot
```sh
arch-chroot /mnt
```
### 7. Configuring from installation
File System Table
```sh
vim /etc/mkinitcpio.conf
----------------------
# Set
MODULES=(btrfs)
----------------------
```
Update initramfs
```sh
mkinitcpio -p linux
```
Time Zone
```sh
ln -sf /usr/share/zoneinfo/Australia/Melbourne /etc/localtime
date
hwclock --systohc
```
### 8. Localization
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
### 9. Hostname
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

### 10. Set root password
```sh
passwd
```
### 11. Add user
```sh
useradd -m -G wheel -s /bin/bash "username"
passwd "username"
EDITOR=vim visudo
```
Near the end of the file uncomment for wheel group users
### 12. Update
```sh
su "username"
sudo pacman -Syu
exit
```
### 13. Configure ZRAM (Optional)
```sh
# Zram setup this is not necessary unless you want to change zram size
vim /etc/systemd/zram-generator.conf
```
You can edit the file:
```sh
[zram0]
zram-size = ram / 2
EOF
```
### 14. Enabling daemons
```sh
systemctl enable NetworkManager
pacman -S bluez
systemctl enable bluetooth
```
### 15. Bootloader
```sh
grub-install --target=x86_64-efi --efi-directory=/boot --botloader-id=GRUB --modules="tpm" --disable-shim-lock
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
```
### 16. Restrict /boot permissions
```sh
chmod 700 /boot
```
### 17. Exit
```sh
exit
umount -a
reboot
```
### 18. Configuring for SecureBoot
```sh
# Get prerequisite
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

yay -S shim-signed
sudo pacman -S sbsigntools
```
Go into root mode
```sh
# Copy to directory where your bootloader is located
cp /usr/share/shim-signed/shimx64.efi /boot/EFI/shimx64.efi
cp /usr/share/shim-signed/mmx64.efi /boot/EFI/

# Create NVRAM entry to boot shim
efibootmgr --unicode --disk /dev/nvme0n1 --part 2 --create --label "Shim" --loader /boot/EFI/shimx64.efi
```
You will need to create your own keys, sign the grub bootloader (not the shim) and the kernel, as well as enrol the key in MokManagement
You must have your own MOK.key, MOK.crt, and MOK.cer. Each time kernel or grub is updated, they need to be signed using the keypair (.key and .crt).
```sh
# Generate key pair
openssl req -newkey rsa:2048 -nodes -keyout MOK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Machine Owner Key/" -out MOK.crt

openssl x509 -outform DER -in MOK.crt -out MOK.cer

# Copy cer to EFI so that you can enrol (you can also enrol using mokutil)
cp MOK.cer /boot/EFI/

# Sign your boot loader
sbsign --key MOK.key --cert MOK.crt --output /efi/vmlinuz-linux /efi/vmlinuz-linux
# Sign your kernel
sbsign --key MOK.key --cert MOK.crt --output /efi/EFI/grubx64.efi /efi/EFI/grubx64.efi
```

Since having to sign each time they are updated is tedious, you can create executable to automate it:
```sh
/etc/initcpio/post/kernel-sbsign
--------------------------------------------
#!/usr/bin/env bash

kernel="$1"
[[ -n "$kernel" ]] || exit 0

# use already install kernel if it exists
[[ ! -f "$KERNELDESTINATION" ]] || kernel="$KERNELDESTINATION"

keypairs=(/path/to/MOK.key /path/to/MOK.crt)

for (( i=0; i<${#keypairs[@]}; i+=2 )) do
    key="${keypairs[$i]}" cert="${keypairs[(( i + 1 ))]}"
    if ! sbverify --cert "$cert" "$kernel" &> /dev/null; then
        sbsign --key "$key" --cert "$cert" --output "$kernel" "$kernel"
    fi
done
```

## Gnome Installation
### 1. Installation
```
sudo pacman -S xorg xorg-server gnome
sudo pacman -S gdm
systemctl enable --now gdm.service
```


## Create Bootable Drive for UEFI Shell
### 1. Format the drive for fat32
```
mkfs.vfat -F32 /dev/"drive"
```
### 2. Mount
```
mkdir /media/usb
mount /dev/"drive" /media/usb
cd /media/usb
```
### 3. Make directory
```
mkdir -p efi/boot/
cd efi/boot/
```
### 4. Download shell
```
sudo wget -q -O BOOTX64.efi https://github.com/tianocore/edk2/raw/edk2-stable201903/ShellBinPkg/UefiShell/X64/Shell.efi
```