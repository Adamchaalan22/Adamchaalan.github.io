# Arch Linux VM Installation Documentation (macOS – Apple Silicon)

# Overview
This document summarizes my process of installing and configuring Arch Linux on macOS (Apple Silicon) using UTM. It is a concise reference for rebuilding the VM if needed and includes every major step, the issues I encountered, and how I resolved them.

This was by far one of the toughest installations I’ve ever done — I had multiple failed boots, GRUB installation errors, and network connection issues that took hours of troubleshooting.

---

## System Setup
**Host System:** macOS (M2 MacBook)  
**Virtualization Tool:** UTM  
**Guest OS:** Arch Linux (AArch64 ISO from [Archboot](https://release.archboot.com/aarch64/latest/iso/))

---

## Step 1 – Preparing the Installation Medium
1. Downloaded the **Arch Linux ARM ISO** and verified its checksum from the Arch Linux website.  
2. Created a new VM in UTM:  
   - Virtualize → Linux (ARM64)  
   - 4 CPU cores, 4 GB RAM, 30 GB storage  
   - Attached the downloaded ISO  
3. Disabled Secure Boot and selected the ISO as the boot source.

// I had to recreate the VM twice because my previous instances got corrupted when UTM froze during the ISO boot. If your VM stops responding, delete it and start from scratch.

---

# Step 2 – Booting the Live Environment
After booting into the Arch ISO, I was presented with a Zsh shell as `root`.

### Setting up keyboard and display
```bash
loadkeys us
setfont ter-132b


### Verifying UEFI boot
```bash
cat /sys/firmware/efi/fw_platform_size
```
Output: `64` confirms a 64-bit UEFI environment.

---

# Step 3 – Connecting to the Internet (Major Challenge)
This was the hardest step. Every time I ran:
```bash
ip link
```
I got: `no devices found`.  

// The official [Arch Wiki guide on Wi-Fi](https://wiki.archlinux.org/title/Wireless_network_configuration) didn’t help much in my case because the Mac UTM network drivers are emulated differently and don’t expose a usable wireless interface directly to Arch.

After hours of searching on the Arch Wiki and DuckDuckGo, I realized that the default UTM network setting (Emulated NIC) was not compatible with the ARM ISO. I fixed it by:
1. Opening UTM > Settings > Network > changing interface to Shared Network (Emulated).
2. Restarted the VM.
3. Used:
```bash
iwctl
station wlan0 connect "MyWiFi"
```
Finally, I was able to get internet connectivity and verify with:
```bash
ping archlinux.org
```

## I strongly recommend ignoring the Arch Wiki’s Wi-Fi instructions for this scenario if using macOS/UTM — they won’t work as expected without physical hardware like ethernet.

---

# Step 4 – Syncing the System Clock
```bash
timedatectl set-ntp true
timedatectl status
```

---

# Step 5 – Partitioning the Disk
Used `fdisk` to partition the disk:
```bash
fdisk /dev/vda
```
Partitions created:
- `/dev/vda1` → EFI (1 GB)
- `/dev/vda2` → swap (4 GB)
- `/dev/vda3` → root (25 GB)

Formatted the partitions:
```bash
mkfs.fat -F 32 /dev/vda1
mkswap /dev/vda2
mkfs.ext4 /dev/vda3
```
Mounted them:
```bash
mount /dev/vda3 /mnt
mount --mkdir /dev/vda1 /mnt/boot
swapon /dev/vda2
```

---

## Step 6 – Installing the Base System
```bash
pacstrap -K /mnt base linux linux-firmware nano
```
// My first two installs failed due to connection timeouts. I fixed it by using faster mirrors with `reflector` before running `pacstrap`.

---

## Step 7 – Configure the System
Generated fstab:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Entered chroot:
```bash
arch-chroot /mnt
```
Set timezone:
```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```
Edited locales:
```bash
nano /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=us" > /etc/vconsole.conf
echo "archmac" > /etc/hostname
```

---

## Step 8 – Networking Configuration
Installed NetworkManager:
```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

---

## Step 9 – Set Root Password
```bash
passwd
```

---

## Step 10 – Boot Loader (Another Major Issue)
Installing GRUB was extremely frustrating. I kept getting errors like:
```
failed to install grub: cannot find EFI directory
```
After hours of searching, I realized I mounted `/boot` instead of `/boot/EFI`. I fixed it by:
```bash
mkdir -p /boot/EFI
mount /dev/vda1 /boot/EFI
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
Once that was done, GRUB installed successfully.

---

## Step 11 – Reboot
Exited and unmounted:
```bash
exit
umount -R /mnt
reboot
```

**Problem:** My VM refused to boot the first time — it would show a blank screen. I realized I forgot to remove the ISO in UTM, so it kept booting into the installer. After ejecting the ISO, the system booted fine.

---

## Step 12 – Post Installation Setup
Created users:
```bash
useradd -m -G wheel adam
passwd adam
useradd -m -G wheel codi
passwd codi
EDITOR=nano visudo
# Enabled %wheel ALL=(ALL:ALL) ALL
```

Installed GUI:
```bash
pacman -S xorg sddm lxqt
systemctl enable sddm
```

Changed default shell:
```bash
pacman -S zsh
chsh -s /bin/zsh adam
```

Installed SSH:
```bash
pacman -S openssh
systemctl enable sshd --now
```

Added aliases in `.zshrc`:
```bash
alias ll='ls -la'
alias update='sudo pacman -Syu'
alias cls='clear'
```

---

## Errors and Recovery Summary
| Issue | Cause | Fix |

| No internet connection | UTM network type not supported | Switched to Shared (Emulated) Network |
| Arch Wi-Fi instructions failed | UTM doesn’t expose usable Wi-Fi NICs | Used shared network instead and avoided Arch Wiki method |
| GRUB install failed | Wrong EFI mount path | Corrected EFI directory mount point |
| VM corruption | UTM crashed mid-install | Rebuilt VM and used smaller disk image |
| Blank screen after reboot | Forgot to remove ISO | Removed ISO from UTM settings |
| Pacstrap timeout | Slow mirrors | Used Reflector to sort fastest mirrors |
| Arch-chroot refused to start | Missing mount | Rechecked `/mnt` structure and remounted partitions correctly |
| GUI wouldn't launch | `sddm` not enabled | Ran `systemctl enable sddm` and rebooted |
| "Permission denied" on sudo | User not added to wheel group | Re-ran `usermod -aG wheel <username>` and rechecked visudo |
| Locale errors on boot | `locale.conf` was missing | Recreated and re-ran `locale-gen` |

---

## Reflection
This installation was extremely challenging. I had to restart the process multiple times, and I learned how unforgiving Arch can be when you skip even one small step. Between network issues, bootloader failures, corrupted installs, chroot problems, GUI launch failures, and misleading documentation, I spent hours troubleshooting using Arch Wiki pages, forums, and DuckDuckGo searches.

Despite the frustration, I gained a deeper understanding of how Linux systems boot, how partitions and file systems work, and how networking is configured from scratch. In the end, it was rewarding to finally see the GUI load successfully.

---

## Final System Summary
- OS: Arch Linux ARM (AArch64)
- DE: LXQt (via SDDM)
- Shell: Zsh
- Network: Configured with NetworkManager
- SSH: Installed and active
- Users: `adam`, `codi` (sudo enabled)
- Boot: UEFI with GRUB

---

## Key Takeaways
1. Always double-check EFI and mount points before installing GRUB.
2. UTM network settings can make or break connectivity.
3. Arch Wiki Wi-Fi instructions may not apply on UTM/macOS.
4. GUI won’t start if you forget to enable the display manager.
5. Document every command; it saves hours later.
6. Don’t skip any step from the Arch Wiki — every line matters.

**End of Installation — Adam Chaalan**


