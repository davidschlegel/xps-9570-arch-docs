#Install reflector to get up-to-date fast mirrors for downloads
sudo pacman -Syu reflector
#run it by country and get the fastest 200  https mirrors
sudo reflector --verbose --country Germany -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist
