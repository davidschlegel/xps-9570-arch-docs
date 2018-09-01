#Install reflector to get up-to-date fast mirrors for downloads
sudo pacman -Syu reflector
#run it by country and get the fastest 200  https mirrors
sudo reflector --verbose --country Germany -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist
#Also possible: Updating mirrorlist at every startup using systemd-service (see wiki)

#Install elinks for command line browser experience 
sudo pacman -Syu elinks

#Install yay into your favourite directory
git clone https://github.com/Jguer/yay
#install go, make, gcc to compile it (It's all in the base-devel, but I didn't want to have so mach bloatware)
sudo pacman -Syu go make gcc
#export the path to make it executable globally
echo 'path+=<Path-to-your-executable>' >> ~/.zshrc
#and make it directly happen...
source ~/.zshrc

#Install nvidia driver
#check your graphics and installed drivers by
lspci -k | grep -A 2 -E "(VGA|3D)"
#install nvidia driver
sudo pacman -Syu nvidia
#use libglvnd as a libgl provider since it is just a dispatch library (bridge) for mesa-libgl and nvidia-libgl
#also install nvidia-utils
sudo pacman -Syu nvidia-utils
#...and reboot. Testing with lscpi as above should now show the nvidia driver loaded.

#Install network manager to autoconnect.
sudo pacman -Syu networkmanager
#connect to wifi:
#enable systemd service for Networkmanager
systemctl enable NetworkManager.service
#... and start it
systemctl start NetworkManager.service
#... and connect
nmcli device wifi connect <SSID> password <PASSWORD>

#install firmware updater fwupd
sudo pacman -Syu fwupd
#get available devices
fwupdmgr get-devices
#refresh metadata
fwupdmgr refresh
#check for updates
fwupdmgr get-updates
#install updates (requires root usually)
sudo fwupdmgr update
#wait for the update to finish; it reboots automatically.


#track dotfiles
#Use yadm for this!
yay yadm-git
#add all neccessary files in your home directory
yadm init
yadm add <dotfiles-to-keep-track-of>
#everything works just like git
#to keep track up new dotfiles, be sure you add them.
#To have a better overview, I suggest to add a .gitignore file to list all files you don't want to track.


#Install i3-gaps
yay i3-gaps dmenu
#Along with it, you probably want a nice lock screen
sudo pacman -Syu i3lock imagemagicks
git clone https://github.com/guimeira/i3lock-fancy-multimonitor
#Then, you might want to install polybar as a nice bar:
yay polybar-git


#Install X and X related stuff
yay xorg-xrandr xf86-video-intel xorg-server xorg-xinit xorg-xev xorg-xbacklight
#Make an Xorg configuration file via
Xorg -configure
#and copy it the generated file to /etc/X11/xorg.conf
#you can modify ~/.Xresources and ~/.xinitrc as you whish
#There is a lot to customize here.
#On this machine, it is also good to put
acpi_rev_override=1
#to your kernel options

#Install scrot to take screenshots
yay scrot

#Install caffeine-ng to prevent system from going to sleep
yay caffeine-ng

#Install redshift (with additional packages for system tray)
sudo pacman -Syu redshift gtk3 python-gobject python-xdg
#Add the following to /etc/geoclue/geoclue.conf to let redshift acces automatic location 
[redshift]
allowed=true
system=false
users=
#In i3, it is important to enable the geoclue agent on startup, so put
exec --no-startup-id /usr/lib/geoclue-2.0/demos/agent
#to your i3 config.
#You can enable a systemd user unit with
systemctl --user enable redshift-gtk.service

#Install Bluetooth
sudo pacman -Syu bluez bluez-utils blueman
#Enable systemd service
systemctl enable bluetooth.service
#Right now ther is an error with authorization (???)

#Install pasystray, a tray for managing audio
yay pasystray
#Also install playerctl to control keybindinigs
yay playerctl

#Install network-manager-applet
sudo pacman -Syu network-manager-applet



#Power down discrete GPU
#The GPU consumes a lot of power, so it is a very good idea
#to power down the gpu permanently (unless you do graphics intensitive stuff)
#The method presented here uses acpi_calls.
#A perhaps better method is to use bumblebee and to start an application with
#the nvidia gpu on demand by running it with primusrun/optirun. Every other applications
#use the integrated intel gpu.
#
#First install acpi_call
yay -Syu acpi_call
#once acpi_call is installed, load the kernel module:
sudo modprobe acpi_call
#the basic command to power down the discrete gpu is:
sudo /usr/share/acpi_call/examples/turn_off_gpu.sh | grep works
#this should reveal on single entry
#using powertop, the power drop should be noticable
#
#edit this file to add the kernel module to the array of modules to load at boot:
sudo vim /etc/modules-load.d/acpi_call.conf
#containing just
acpi_call
#To turn off the GPU at boot use systemd-tmpfile:
sudo vim /etc/tmpfiles.d/acpi_call.conf
#It should contain this:
w /proc/acpi/call - - - - \\_SB.PCI0.PEG0.PEGP._OFF
#
#One problem that remains is that after suspension/hibernation the gpu is powered on
# and has to be turned off manually


#Undervolt CPU, CPU Cache, and GPU
#install intel-undervolt
yay intel-undervolt
intel-undervolt read
#edit the confit file to desired undervolt
sudo vim /etc/intel-undervolt.conf
#apply
sudo intel-undervolt apply
#...and read again to check
sudo intel-undervolt read
#start systemd service
systemctl start intel-undervolt.service
#and enable it
systemctl enable intel-undervolt.service




#Swap file
#
#Install systemd-swap
pacman -Syu systemd-swap
#edit
sudo vim /etc/systemd/swap.conf
#and set:
swapfc_enabled=1
swapfc_force_preallocated=1
#Then start and enable systemd-swap
systemctl start systemd-swap.service
systemctl enable systemd-swap.service
#It is a very good idea to decrease swappiness, since you sould have enough ram for every day purposes
#So, edit 
sudo vim /sys/fs/cgroup/memory/memory.swappiness 
#to a value of (default is 60)
10
#
#Hibernation into swap file
#
#set kernel parameter
resume=<Your/Partition> 
#This is in my case the normal root LVM Partition
#
#Additional kernel parameter required for swap file:
resume_offset=<swap_file_offset>
#where the value of <swap_file_offset> can be obtained running
filefrag -v <swap_file>
#or -- to be absolutely sure -- by using
swap-offset
#from the uswusp-git in the AUR.
#
#configure initramfs if neccessary (it is NOT when you have already the systemd hook)
sudo vim /etc/mkinitcpio.conf
#and edit
HOOKS=(... resume ...)
#resume hook should hook after lvm2, if you use LVM partitions
#... and regenerade initramfs
sudo mkinitcpio -p linux
#
#...and you sould be good to go after a reboot.
