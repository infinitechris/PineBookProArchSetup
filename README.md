I finally grew tired of having to randomly follow at least three guides (and possibly have to re-work my install at least two more times before it's ready) to setup Arch my normal way, so instead I figured I'd create this somewhat useful guide.

I'm not going into full detail of the steps performed, as they're listed on the guides I used.

After this guide is complete, Arch with KDE Plasma for a DE will be setup!

You may notice I opt for a cold start instead of rebooting. This is because of bugs within the Pinebook's wifi firmware. This is my preference, and `reboot` can be subbed in any place where `shutdown -h now` is noted.

**NOTE**: Following this guide WILL overwrite all MicroSD and EMMC contents.

YOU HAVE BEEN WARNED!

Moving on.

**Guides used**:
- [x] [Installing Arch Linux on a PineBook Pro (external storage)](https://www.lorenzobettini.it/2022/12/installing-arch-linux-on-a-pinebook-pro-external-storage/)
     - I follow this guide until I reach the point where a DE is being installed and then move to the itsfoss guide below
- [x] [How to Properly Install and Setup KDE Plasma on Arch Linux](https://itsfoss.com/install-kde-arch-linux/)
     - I prefer this DE for my use case

Neither of the two main guides (Pine64's own and the other guide here on GitHub) I followed worked out of the box. So I ended up following Lorenzo Bettini's guide which worked every time! (provided I didn't mess up a step)

Also, I have a [Raspberry Pi USB WiFi Dongle](https://www.raspberrypi.com/products/raspberry-pi-usb-wifi-dongle/) that I will be using until the PineBook Firmware is loaded, so your steps leading up to installing firmware will likely vary, YMMV.

## Prerequisites
 - An existing Arch or Manjaro install must be used on the PineBook Pro for setup. I will eventually cover installing Manjaro onto a MicroSD card to use for this section, but it is currently out of scope.
- I prefer `axel` to `wget`, and I will be referencing it instead of `wget` in scripts. 
 - a MicroSD card to install Arch onto (naturally) at least 8GB in size
 - Internet access or *somewhat* recent downloads of:
     - `Tow Boot`: I used the version released on 10/05/2021 [from here](https://github.com/Tow-Boot/Tow-Boot/releases/download/release-2021.10-005/pine64-pinebookPro-2021.10-005.tar.xz)
     - [ARM Arch Linux install image](http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz)
          - [relevant signiture](http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.sig)
 - I do recommend conenction via SSH after rebooting into Arch from MicroSD, as there are some lengthly commands to type out, some which contain signing keys and are a pain to type out manually.
# MicroSD Setup
 - All of the following commands must be run as `root` so you must become `root`

       sudo su -
 - I perform the following commands in `Downloads`

       cd Downloads
       axel -a https://github.com/Tow-Boot/Tow-Boot/releases/download/release-2021.10-005/pine64-pinebookPro-2021.10-005.tar.xz
       tar xvf pine64-pinebookPro-2021.10-005.tar.xz
 - use `lsblk` before running the following to figure out which device you will be flashing Tow Boot and change if needed
 
       dd if=pine64-pinebookPro-2021.10-005/shared.disk-image.img of=/dev/mmcblk2 bs=1M oflag=direct,sync status=progress
 - Create partitions on this device

       fdisk /dev/mmcblk2
 - `fdisk` commands to be added here as I follow back through this guide to make a backup EMMC
 - Format `root` and `boot` partitions

       mkfs.ext4 /dev/mmcblk2p2
       mkfs.ext4 /dev/mmcblk2p3
 - Mount `root` and `boot` partitions

       mount /dev/mmcblk2p3 /mnt
       mkdir /mnt/boot
       mount /dev/mmcblk2p2 /mnt/boot
 - Download the Arch ARM Installer Image and verify it via gpg

       axel -a http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
       axel -a http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz.sig
       gpg --keyserver keyserver.ubuntu.com --recv-keys 68B3537F39A313B3E574D06777193F152BDBE6A6
       gpg --verify ArchLinuxARM-aarch64-latest.tar.gz.sig
 - Expand the image to `root` partition

       bsdtar -xpvf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt
 - Copy `root` partition UUID to `/mnt/etc/fstab`

       blkid /dev/sda3 >> /mnt/etc/fstab
 - Adjust `/mnt/etc/fstab` accordingly

       nano /mnt/etc/fstab
 - copy `boot` partition UUID to `/mnt/etc/fstab`

       blkid /dev/sda2 >> /mnt/etc/fstab
 - Adjust `/mnt/etc/fstab` accordingly
       
       nano /mnt/etc/fstab
 - Create `extlinux` directory within `/mnt/boot`

       mkdir -p /mnt/boot/extlinux
 - Add the `root` partition UUID to `extlinux.conf` file
 - 
       nano /mnt/boot/extlinux/extlinux.conf
 - Add `pacstrap` install of `dialog` and anything else needed before shutting down and restarting here
 - Unmount the drives mounted to `/mnt`

       umount -R /mnt
 - Shutdown

       shutdown -h now
# EMMC Setup
Whew! That part is done. From here in you **will** need a live internet connection.

I **highly** recommend connecting via SSH for this section as there is a key signiture we will be verifying against. This will be covered in advance of the relevant step.

That said it's not 100% needed as I've completed this install manually typing in commands to be run quite a bit in the past with moderate levels of success!

## Updating Arch on MicroSD
You will need to select the MicroSD to boot from for this section.

When you're ready, power back up, then press `ESC` when the prompt on the bottom of your screen shows up.

Then select the SD card to boot from.

If the steps above were completed successffuly, it should boot to a login prompt.
 - Login as `root`, password `root`
 - I recommend changing `root` and `alarm` passwords at this tine

       passwd
       passwd alarm
- Initalize the `pacman` keyring and populate `archlinuxarm` 

       pacman-key --init
       pacman-key --populate archlinuxarm
 - Sync time and date from `ntp` servers to hardware (I'm on the east cost and use `Detroit`)

       timedatectl set-timezone America\Detroit
       timedatectl set-ntp on
       hwclock --systohc
 - Config your `locale` and generating

       nano /etc/locale.gen
 - Uncomment `#en_US.UTF-8` and save/ exit
 - Generate `locales`

       locale-gen
 - Edit `locale.conf` and set `LANG` vairalbe to match the above

       nano /etc/locale.conf
 - Delete `C` and replace with `en_US.UTF-8`, then save/ exit
 - I typically update out of date packages at this time that way the OS on the MicroSD is up to date

       pacman -Syu
 - As long as there were no errors, we will install `sudo`

       pacman -S sudo
 - Edit `sudoers` to allow members of the group `wheel` to access `sudo`

       nano /etc/sudoers
 - Uncomment the line `# %wheel ALL=(ALL:ALL) ALL` and then save/ exit
 - Shutdown

       shutdown -h now
   
You can continue following alog with this guide from this point.

This will give you a 100% usable MicroSD running pinebook firmware, or skip to [here](https://github.com/infinitechris/PineBookProArchSetup/tree/main#writing-os-to-emmc) as the next steps will all be repeated on the Arch install on the EMMC.

## Installing Pinebook Firmware on MicroSD
Power back up your system to verify that everything updated correctly, hitting `ESC` when the prompt on the bottom of your screen appears, and selecting the MicroSD to boot from.

- When the login prompt appears, login as `alarm` at this point, so that we can ssh back in
- run `sudo wifi-menu` and connect to your AP
- run `ifconfig` to find your IP address
- On another device connect to your pinebook's ip address using ssh to your Pinebook
- Become `root` again, after logging in, as all of the following commands need to be run as `root` and not `sudo COMMAND`

       sudo su
- Add the kiljan pgp keys for the Pinebook firmware

       pacman-key --keyserver hkps://keys.openpgp.org/ --recv-keys A1EC3C686EF7A9DD232D1563D4D12D6AA6A92769
       pacman-key --lsign-key A1EC3C686EF7A9DD232D1563D4D12D6AA6A92769

       cat << 'EOF' >> /etc/pacman.conf

       [archlinuxarm-pbp]
       SigLevel = Optional TrustedOnly
       Server = http://pacman.kiljan.org/$repo/os/$arch
       EOF
 - Sync the repos

       pacman -Sy
 - Install the packages for the Pinebook firmware

       pacman -S ap6256-firmware libdrm-pinebookpro pinebookpro-audio pinebookpro-post-install towboot-pinebookpro-bin
 - Shutdown

       shutdown -h now

## Writing OS to EMMC
A lot of the following commands in the following section are the same as [MicroSD Setup](https://github.com/infinitechris/PineBookProArchSetup/blob/main/README.md#microsd-setup), as the majority of the changes are for the destination for `dd` 

Power back up your system to verify that everything is installed correctly, hitting `ESC` when the prompt on the bottom of your screen appears, and selecting the MicroSD to boot from.

 - When the login prompt appears, login as `root`
 - Create the `Downloads` directory

       mkdir Downloads
 - Change to this directory, download `Tow Boot`, then extract it

       cd Downloads
       axel -a https://github.com/Tow-Boot/Tow-Boot/releases/download/release-2021.10-005/pine64-pinebookPro-2021.10-005.tar.xz
       tar xvf pine64-pinebookPro-2021.10-005.tar.xz
 - use `lsblk` before running the following to figure out which device you will be flashing Tow Boot and change if needed
 
       dd if=pine64-pinebookPro-2021.10-005/shared.disk-image.img of=/dev/mmcblk1 bs=1M oflag=direct,sync status=progress
 - Create partitions on this device

       fdisk /dev/mmcblk1
 - `fdisk` commands to be added here as I follow back through this guide to make a backup EMMC
 - Format `root` and `boot` partitions

       mkfs.ext4 /dev/mmcblk1p2
       mkfs.ext4 /dev/mmcblk1p3
 - Mount `root` and `boot` partitions

       mount /dev/mmcblk1p3 /mnt
       mkdir /mnt/boot
       mount /dev/mmcblk1p2 /mnt/boot
 - Download the Arch ARM Installer Image and verify it via gpg

       axel -a http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
       axel -a http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz.sig
       gpg --keyserver keyserver.ubuntu.com --recv-keys 68B3537F39A313B3E574D06777193F152BDBE6A6
       gpg --verify ArchLinuxARM-aarch64-latest.tar.gz.sig
 - Expand the image to `root` partition

       bsdtar -xpvf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt
 - Copy `root` partition UUID to `/mnt/etc/fstab`

       blkid /dev/mmcblk1p3 >> /mnt/etc/fstab
 - Adjust `/mnt/etc/fstab` accordingly

       nano /mnt/etc/fstab
 - copy `boot` partition UUID to `/mnt/etc/fstab`

       blkid /dev/mmcblk1p2 >> /mnt/etc/fstab
 - Adjust `/mnt/etc/fstab` accordingly
       
       nano /mnt/etc/fstab
 - Create `extlinux` directory within `/mnt/boot`

       mkdir -p /mnt/boot/extlinux
 - Add the `root` partition UUID to `extlinux.conf` file
 - 
       nano /mnt/boot/extlinux/extlinux.conf
 - Add `pacstrap` install of `dialog` and anything else needed before shutting down and restarting here
 - Unmount the drives mounted to `/mnt`

       umount -R /mnt
 - Shutdown

       shutdown -h now
From this point on, we will be booting from the EMMC. You can still hit `ESC` then select this device, but the system will boot to EMMC by default as long as there's a bootable system installed there
## Installing Pinebook firmware to EMMC
The exact same steps from [Installing Pinebook Firmware on MicroSD](https://github.com/infinitechris/PineBookProArchSetup/tree/main#installing-pinebook-firmware-on-microsd) will be followed to the letter. So instead of me duplicating them here, I suggest following that section. The only difference is that wewill be booting from EMMC and not MicroSD

# KDE Plasma Setup
Now that we have firmware added to the EMMC, the MicroSD can be removed along with and dongles/ adapters you were using to install said firmware.

It's time to get a DM installed and get into our operating envirnment!
- After booting up and logging in as `alarm` lets make our personal user name that we'll be using going forward

       useradd -m triconda
 - give yourself a password

       passwd triconda
 - give yourself group permissions (add these here)
 - We will now install the following KDE prereqs and applications
     - Xorg group
     - KDE Plasma Desktop Environment
     - Wayland session for KDE Plasma
     - KDE applications

              pacman -S xorg plasma plasma-wayland-session kde-applications
 - Once the above completes, `enable` the `Displace Manager service`

        systemctl enable sddm.service

- Shutdown

       shutdown -h now
We're now ready to play around in a GUI and perform some needed Post-Install tweaks!
# Post-Install Tweaks
These are all applications that I install (chromium, snapd store, etc) to make the laptop into a more usable workstation
## KDE Tweaks
 - Dark mode / theme, power saving, login and lock screen setup here
## Yay AUR Helper
 - Install documentation here
## Snapd Store
 - Install documentation here
## Chromium
 - Install documentation here
## KDE Connect config
 - Simple walkthrough of app and android app setup
## Telegram
 - Install documentation here

# Other Notes
Right now, I have the following lines in `/etc/pacman.conf`:

     # Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
     IgnorePkg   = generic-post-install pinebookpro-post-install
This is to ignore the two above packages from attempting to update through `core`, `extra`, `community`, `alarm`, and `aur` repos, which don't contain it and breaks any attempts at a system update right now for *some reason* yet to be found. Full `pacman.conf` that is working for me to this day is in [files](https://github.com/infinitechris/PineBookProArchSetup/blob/2d1a0ec5b527f502705739a9c55fa2c184ca0e95/files/pacman.conf)

I also have a mobile hotspot that I use to access the internet, and if I was previously on a *real* AP, I have found that I have to stop `NetworkManager` (if already running) or start (if not) and then stop it. Then `wifi-menu` itializes `wlan` fine
