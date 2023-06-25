I finally grew tired of having to randomly follow at least three guides (and possibly have to re-work my install at least two more times before it's ready) to setup Arch my normal way, so instead I figured I'd create this somewhat useful guide!

After this guide is complete, Arch with KDE Plasma for a DE will be setup!

Guides to be integrated:
- [ ] [Installing Arch Linux on a PineBook Pro (external storage)](https://www.lorenzobettini.it/2022/12/installing-arch-linux-on-a-pinebook-pro-external-storage/)
     - I follow this guide until I reach the point where a DE is being installed and then move to the itsfoss guide below
- [ ] [How to Install Arch Linux [Step by Step Guide]](https://itsfoss.com/install-arch-linux/)
     - I mostly follow this guide for the locale setups.
- [ ] [How to Properly Install and Setup KDE Plasma on Arch Linux](https://itsfoss.com/install-kde-arch-linux/)
     - I prefer this DE for my use case

# MicroSD Setup
Neither of the two main guides (Pine64's own and the other guide here on GitHub) I followed worked out of the box. So I ended up following Lorenzo Bettini's guide which worked every time! (provided I didn't mess up a step)

Also, I have a [Raspberry Pi USB WiFi Dongle](https://www.raspberrypi.com/products/raspberry-pi-usb-wifi-dongle/) that I will be using until the PineBook Firmware is loaded, so your steps leading up to installing firmware will likely vary, YMMV.

## Prerequisites
 - An existing Arch or Manjaro install must be used for this section. I will eventually cover installing Manjaro onto a MicroSD card to use for this section, but it is currently out of scope.
 - a MicroSD card to install Arch onto (naturally)
 - Internet access or a * somewhat* recent download of Tow Boot. I used the version released on 10/05/2021 [from here](https://github.com/Tow-Boot/Tow-Boot/releases/download/release-2021.10-005/pine64-pinebookPro-2021.10-005.tar.xz), which is also located in [files]()

tar xf pine64-pinebookPro-2021.10-005.tar.xz

# EMMC Setup

# KDE Plasma Setup

# Post-Install Commands

# Other Notes
Right now, I have the following lines in my /etc/pacman.conf:
     # Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
     IgnorePkg   = generic-post-install pinebookpro-post-install
This is to ignore the two above packages from attempting to update through core, extra, community, alarm and aur repos, which don't contain it and breaks any attempts at a system update right now for *some reason* yet to be found. Full pacman.conf that is working for me to this day is in [files](https://github.com/infinitechris/PineBookProArchSetup/blob/2d1a0ec5b527f502705739a9c55fa2c184ca0e95/files/pacman.conf)

I also have a mobile hotspot that I use to access the internet, and if I was previously on a *real* AP, I have found that I have to stop NetworkManager (if already running) or start (if not) and then stop it. Then wifi-menu runs fine
