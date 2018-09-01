# xps-9570-arch-docs

This is a repository where I document all kinds of stuff about my Dell xps 9570 setup.\
#### Tree:
 - [Arch config](https://github.com/davidschlegel/xps-9570-arch-docs/blob/master/Arch_config.txt)\
    Here, I add stuff that I change to my system, so that I can look up what I have done.
    So it basically consists of shell commands with some comments.
    So, be aware, that it is not all well documented, always look things up at the archwiki first.

 - [Arch installation](https://github.com/davidschlegel/xps-9570-arch-docs/blob/master/Arch_installation_procedure.txt) \
    My installation proceedure. Most of it was taken by [grobgl](https://github.com/grobgl/arch-linux-setup)

 - [pacman package list](https://github.com/davidschlegel/xps-9570-arch-docs/blob/master/pacman_packages_explicit.txt)
    list of pacman packages sorted by date, generated with the command:\
  `expac --timefmt='%Y-%m-%d %T' -HM '%l\t\t%-30n\t%10d' $(pacman -Qqent) | sort > pacman_packages_explicit.txt`

- [aur / foreign package list](https://github.com/davidschlegel/xps-9570-arch-docs/blob/master/aur_packages_explicit.txt)\
list of aur/foreign packages, i.e. packages not installed by pacman,  sorted by date, generated with the command: \
`expac --timefmt='%Y-%m-%d %T' -HM '%l\t\t%-30n\t%10d' $(pacman -Qm) | sort > aur_packages_explicit.txt`

