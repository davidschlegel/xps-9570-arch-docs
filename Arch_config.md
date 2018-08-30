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
export PATH=<Path-to-your-executable>:$PATH
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
#and reboot. Testing with lscpi as above should now show the nvidia driver loaded.
