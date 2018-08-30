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


