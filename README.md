# MicroSD Setup

# EMMC Setup

# Post-Install Commands

# Other Notes
Right now, I have the following setup in my /etc/pacman.conf:
     # Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
     IgnorePkg   = generic-post-install pinebookpro-post-install
Rhis is to ignore the two above packages from attempting to update through core, extra, community, alarm and aur repos, which don't contain it and breaks any attempts at a system update right now for *some reason* yet to be found. Full pacman.conf that is working for me to this day is in [files](https://github.com/infinitechris/PineBookProArchSetup/blob/2d1a0ec5b527f502705739a9c55fa2c184ca0e95/files/pacman.conf)
