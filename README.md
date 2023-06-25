I finally grew tired of having to randomly follow at least three guides (and possibly have to re-work my install at least two more times before it's ready) to setup Arch my normal way, so instead I figured I'd create this somewhat useful guide!

I'm not going into full detail of the steps performed, as they're listed on the guides I used.

After this guide is complete, Arch with KDE Plasma for a DE will be setup!

Moving on.

**Guides used**:
- [ ] [Installing Arch Linux on a PineBook Pro (external storage)](https://www.lorenzobettini.it/2022/12/installing-arch-linux-on-a-pinebook-pro-external-storage/)
     - I follow this guide until I reach the point where a DE is being installed and then move to the itsfoss guide below
- [ ] [How to Install Arch Linux [Step by Step Guide]](https://itsfoss.com/install-arch-linux/)
     - I mostly follow this guide for the locale setups.
- [ ] [How to Properly Install and Setup KDE Plasma on Arch Linux](https://itsfoss.com/install-kde-arch-linux/)
     - I prefer this DE for my use case

Neither of the two main guides (Pine64's own and the other guide here on GitHub) I followed worked out of the box. So I ended up following Lorenzo Bettini's guide which worked every time! (provided I didn't mess up a step)

Also, I have a [Raspberry Pi USB WiFi Dongle](https://www.raspberrypi.com/products/raspberry-pi-usb-wifi-dongle/) that I will be using until the PineBook Firmware is loaded, so your steps leading up to installing firmware will likely vary, YMMV.

## Prerequisites
 - An existing Arch or Manjaro install must be used on the PineBook Pro for this section. I will eventually cover installing Manjaro onto a MicroSD card to use for this section, but it is currently out of scope.
- I prefer `axel` to `wget`, and I will be referencing it instead of `wget` in scripts. 
 - a MicroSD card to install Arch onto (naturally) at least 8GB in size
 - Internet access or *somewhat* recent downloads of:
     - `Tow Boot`: I used the version released on 10/05/2021 [from here](https://github.com/Tow-Boot/Tow-Boot/releases/download/release-2021.10-005/pine64-pinebookPro-2021.10-005.tar.xz)
     - [ARM Arch Linux install image](http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz)
          - [relevant signiture](http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.sig)
 - I do recommend conenction via SSH after rebooting into Arch from MicroSD, as there are some lengthly commands to type out, some which contain signing keys and are a pain to type out manually.
# MicroSD Setup
 - All of the following commands must be run as `root` so you must become `root`
 - 
       sudo su -
 - I perform the following commands in `Downloads`
 - 
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
Whew! That part is done. From here in you **will** need a live internet connection. I'll document how I handle this with my dongle, but your solution likely varies.

You will need to select the MicroSD to boot from for this section.

When you're ready, power back up, then press `ESC` when the prompt on the bottom of your screen shows up.

Then select the SD card to boot from.

If the steps above were completed successffuly, it should boot to a login prompt.
 - Login as `root`, password `root`
 - I recommend changing `root` and `alarm` passwords at this tine

       passwd
       passwd alarm

# KDE Plasma Setup

# Post-Install Commands

# Other Notes
Right now, I have the following lines in my /etc/pacman.conf:

     # Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
     IgnorePkg   = generic-post-install pinebookpro-post-install
This is to ignore the two above packages from attempting to update through core, extra, community, alarm and aur repos, which don't contain it and breaks any attempts at a system update right now for *some reason* yet to be found. Full pacman.conf that is working for me to this day is in [files](https://github.com/infinitechris/PineBookProArchSetup/blob/2d1a0ec5b527f502705739a9c55fa2c184ca0e95/files/pacman.conf)

I also have a mobile hotspot that I use to access the internet, and if I was previously on a *real* AP, I have found that I have to stop NetworkManager (if already running) or start (if not) and then stop it. Then wifi-menu runs fine
