# Arch Linux VM Installation Documentation
Built by hand in VMware by following the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide).

## Creating the VM

1. Downloaded the ISO from https://dfw.mirror.rackspace.com/archlinux/iso/2026.10.01/
2. Opened VMware and created a new VM, selecting the Arch ISO as the installer disc.
3. Set the guest OS to Other Linux 6.x kernel 64-bit.
4. Gave the VM 4096 MB of RAM, 2 processors with 2 cores, and a 30 GB disk.
5. Set the network adapter to NAT.
6. Started the VM and selected Arch Linux install medium from the boot menu.

## Verifying the Boot Mode, Network, and Clock

7. Checked whether the VM booted in UEFI mode:
   ```bash
   cat /sys/firmware/efi/fw_platform_size
   ```
   > **Issue:** It returned "No such file or directory," which means the VM was booting in BIOS mode, not UEFI.
   >
   > **Fix:** Powered off the VM, changed the firmware from **BIOS** to **UEFI** in the VM settings, and booted the ISO again.

8. Re-ran the same command. It now printed `64`, confirming UEFI mode.
9. Tested the network connection (packets sent fine):
   ```bash
   ping archlinux.org
   ```
10. Synced the system clock:
    ```bash
    timedatectl set-ntp true
    ```

## Partitioning, Formatting, and Mounting

11. Listed the disks to find the target disk (`/dev/sda`):
    ```bash
    lsblk
    ```
12. Opened the disk in `fdisk`:
    ```bash
    fdisk /dev/sda
    ```
    Inside `fdisk` (`m` shows the help menu):
    - `g`: created a new empty GPT partition table.
    - `n`: created partition 1 for EFI, size `+1G`.
    - `t`: changed the partition type by entering `1` (EFI System).
    - `n`: created partition 2 for root, using the remaining 29 GB.
    - `p`: printed the partition table to check it.
    - `w`: wrote the changes and exited.

13. Formatted the partitions:
    ```bash
    mkfs.fat -F 32 /dev/sda1
    mkfs.ext4 /dev/sda2
    ```
14. Mounted them:
    ```bash
    mount /dev/sda2 /mnt
    mount --mkdir /dev/sda1 /mnt/boot
    ```

## Installing the Base System

15. Installed the base system, plus `nano`, `networkmanager`, `sudo`, and `open-vm-tools`:
    ```bash
    pacstrap -K /mnt base linux linux-firmware nano networkmanager sudo open-vm-tools
    ```
16. Generated the fstab and entered the new system:
    ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    arch-chroot /mnt
    ```

## Configuring the System (inside chroot)

17. Set the time zone and synced the hardware clock:
    ```bash
    ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
    hwclock --systohc
    ```
18. Set the locale. Opened `nano /etc/locale.gen`, removed the `#` in front of `en_US.UTF-8 UTF-8`, then saved and exited. Then:
    ```bash
    locale-gen
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    ```
19. Set the hostname:
    ```bash
    echo "myarch" > /etc/hostname
    ```
20. Set the root password:
    ```bash
    passwd
    ```
21. Installed the GRUB bootloader:
    ```bash
    pacman -S grub efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot --removable
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
    I used `--removable` so GRUB is installed at the UEFI fallback path. VMware does not reliably keep UEFI boot entries between reboots.
22. Enabled networking and the VMware tools service:
    ```bash
    systemctl enable NetworkManager
    systemctl enable vmtoolsd
    ```
23. Left the chroot, unmounted everything, and rebooted:
    ```bash
    exit
    umount -R /mnt
    reboot
    ```
    Disconnected the ISO from the VM's CD/DVD drive so it booted from the disk instead of the installer.

## Users, Shell, and SSH

24. Logged in as `root` with the password I set earlier.
25. Installed zsh and ssh:
    ```bash
    pacman -S zsh openssh
    ```
26. Created my user with zsh as the login shell and set its password (`yourname` is a placeholder; I used `crw9423`):
    ```bash
    useradd -m -G wheel -s /usr/bin/zsh crw9423
    passwd crw9423
    ```
27. Created the `codi` user the same way:
    ```bash
    useradd -m -G wheel -s /usr/bin/zsh codi
    passwd codi
    ```
28. Gave the `wheel` group sudo access. Ran `EDITOR=nano visudo` and uncommented this line:
    ```
    %wheel ALL=(ALL:ALL) ALL
    ```
29. Enabled and checked the ssh server (status showed `active (running)`; pressed `q` to leave the status screen):
    ```bash
    systemctl enable --now sshd
    systemctl status sshd
    ```
30. Tested my new user and sudo:
    ```bash
    su - crw9423
    ```
    On the first zsh startup it showed a setup menu, and I pressed `q` to skip it. Then:
    ```bash
    sudo whoami
    ```
    It printed `root`, so sudo works. Then `exit` to return to root.

## Installing the Desktop Environment (LXQt)

31. Installed the display server, LXQt, and the SDDM login manager:
    ```bash
    pacman -S xorg-server lxqt sddm
    ```
32. Installed extra packages for VMware and the desktop:
    ```bash
    pacman -S xf86-video-vmware xf86-input-vmmouse gtkmm3 ttf-dejavu firefox
    ```
    > **Issue:** pacman returned `target not found` for `xf86-video-vmware` (and `xf86-input-vmmouse`), so the install failed.
    >
    > **Fix:** These legacy X.org drivers are no longer in the repositories, and they aren't needed because the kernel's `vmwgfx` driver and X's built-in `modesetting` driver handle the VMware display. Installed only the packages that exist:
    > ```bash
    > pacman -S gtkmm3 ttf-dejavu firefox
    > ```

33. Enabled the login manager so the system boots into the GUI, and enabled the VMware services:
    ```bash
    systemctl enable sddm
    systemctl enable vmtoolsd vmware-vmblock-fuse
    ```
34. Rebooted:
    ```bash
    reboot
    ```
    At the SDDM login screen I made sure the session was **LXQt Desktop**, then logged in as my user.

## Terminal Colors and Aliases

35. Opened **QTerminal** from the menu.
    > **Issue:** Pasting a command into the terminal only showed `^V`.
    >
    > **Fix:** Terminal apps don't paste with Ctrl+V. Used **Ctrl+Shift+V** (or right-click, then Paste) instead.

36. Wrote the zsh config with a colored prompt, colored command output, and aliases:
    ```bash
    cat > ~/.zshrc << 'EOF'
    autoload -U colors && colors
    PROMPT='%F{green}%n@%m%f %F{blue}%~%f %# '
    alias ls='ls --color=auto'
    alias grep='grep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip -color=auto'
    alias ll='ls -lah'
    alias ..='cd ..'
    alias ...='cd ../..'
    alias update='sudo pacman -Syu'
    alias install='sudo pacman -S'
    alias cls='clear'
    alias myip='ip -brief addr'
    alias ports='ss -tuln'
    alias df='df -h'
    alias free='free -h'
    EOF
    ```
37. Loaded the new config:
    ```bash
    source ~/.zshrc
    ```
38. Turned on colored pacman output. Ran `sudo nano /etc/pacman.conf`, uncommented `#Color`, and added `ILoveCandy` below it.
39. Copied the config to the `codi` user and fixed its ownership:
    ```bash
    sudo cp ~/.zshrc /home/codi/.zshrc
    sudo chown codi:codi /home/codi/.zshrc
    ```

## Final Checks

40. Verified each requirement:
    ```bash
    echo $SHELL                  # /usr/bin/zsh
    systemctl is-active sshd     # active
    systemctl get-default        # graphical.target
    groups; id codi              # both include wheel
    ```
