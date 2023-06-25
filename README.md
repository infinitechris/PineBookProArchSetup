I finally grew tired of having to randomly follow at least three guides (and possibly have to re-work my install at least two more times before it's ready) to setup Arch my normal way, so instead I figured I'd create this somewhat useful guide!

After this guide is complete, Arch and KDE Plasma will be setup!

Guides to be integrated:
- [ ] [Installing Arch Linux on a PineBook Pro (external storage)](https://www.lorenzobettini.it/2022/12/installing-arch-linux-on-a-pinebook-pro-external-storage/)
- [ ] [How to Properly Install and Setup KDE Plasma on Arch Linux](https://itsfoss.com/install-kde-arch-linux/)
- [ ] [How to Install Arch Linux [Step by Step Guide]](https://itsfoss.com/install-arch-linux/)

# MicroSD Setup

# EMMC Setup

# KDE Plasma Setup

# Post-Install Commands

# Other Notes
Right now, I have the following setup in my /etc/pacman.conf:
     # Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
     IgnorePkg   = generic-post-install pinebookpro-post-install
Rhis is to ignore the two above packages from attempting to update through core, extra, community, alarm and aur repos, which don't contain it and breaks any attempts at a system update right now for *some reason* yet to be found. Full pacman.conf that is working for me to this day is in [files](https://github.com/infinitechris/PineBookProArchSetup/blob/2d1a0ec5b527f502705739a9c55fa2c184ca0e95/files/pacman.conf)
