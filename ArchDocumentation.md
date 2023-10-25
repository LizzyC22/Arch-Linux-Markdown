# Arch Documentation
<font color="gold"> Elizabeth Christensen</font>
<font color="white">  
Starting off with obtaining the VMWorkstation Pro license and Arch ISO, I then installed VMWorkstation Pro 17.  
Creating the Arch Linux Distro inside of VMWorkstation  
Select Create a New Virtual Machine > Setup Arch Linux VM  
edit the vmx in notepad or vsCode  
boot up Arch in UEFI  
 <font color="red"> 
 # First Error Encountered  </font>  

While it was booting up in UEFI, it ran into a connection error. Gave it some time and it corrected itself.  
# Back to Arch Pre Installation
Rule of thumb, you cannot paste anything into the vm.  
<font color="royalblue"> 1.6 Verify the boot mode</font>  
Verify boot mode > ls /sys/firmware/efi/efivars   
<font color="royalblue"> 1.7 Connect to the internet</font>  
Verify you have a internet connection > ip link  
Ping something to verify. > ping minecraft.net   
<font color="royalblue"> 1.8 Update the system clock </font>  
Update the system clock > timedatectl status  
<font color ="royalblue"> 1.9 Partition the disks </font>  
To start this process > fdisk -l  
modify partition tables > fdisk /dev/sda  
A boot partition is needed.  
Add first new partition > n  
Make it primary > p  
Leave the first sector alone  
Last sector > +500M  
Add a second partition > n  
Make it primary > p  
Leave the first and last sector alone  
Save and exit > w  
<font color="royalblue"> 1.10 Format the partitions </font>  
Since this is a EFI system partition, it needs a FAT32 file system. This means one needs to use > mkfs.fat -F32 /dev/sda1  
Need to create a new file system of type ext4 as well > mkfs.ext4 /dev/sda2  
<font color="royalblue"> 1.11 Mount the File Systems </font>  
Typically, now we need to mount the iso file, which in this case is > mount /dev/sda2 /mnt  

# Now onto the actual Arch Installation  
<font color="royalblue"> 2.1 Select the mirrors </font>  
We need to download our packages from something called a 'mirror server'. We can check and edit > nano /etc/pacman.d/mirrorlist for options.  
  
I navigated to archlinux.org/mirrorlist/ and generated a pacman mirrorlist based on my geography and needs with this installation. I selected United States and kept the default selections of protocols http and https, and IP version of IPv4.  
  
From this list, I selected my top 5 choices and typed them into the pacman.d/mirrorlist file.

<font color="royalblue"> 2.2 Install essential packages</font>  
To install the base package, Linux kernel and firmware for common hardware > pacstrap -K /mnt base linux linux-firmware  
Please be aware this will take some time, between 5 to 20 minutes.  
  
Additional packages can be installed, but thats the base package.  
# Configure the system  

<font color="royalblue"> 3.1 Fstab</font>  
To start the configuration, we need to generate a fstab file > genfstab -U -p /mnt >> /mnt/etc/fstab  
We can check the fstab file content with > cat /mnt/etc/fstab  

<font color="royalblue"> 3.2 Chroot</font>  
We now need to chroot into /mnt > arch-chroot /mnt  
  
<font color="royalblue"> 3.3 Time zone</font>  
Next up is setting the timezone with > ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime  
Following this, we should set the hardware clock to use UTC > hwclock --systohc --utc  
<font color="red">
 # Second Error Encountered </font>  
  I accidentally typed --system instead of --systohc, but checked my previous command and fixed what I typed.  
    
<font color="royalblue">3.4 Localization </font>  
Make sure en_US.UTF-8 UTF-8 is uncommented inside of the /etc/locale.gen file > nano /etc/locale.gen  
  
Then we generate locales by running > locale-gen  
Create locale.conf(5) > touch /etc/locale.conf  
Inside of this file > nano /etc/locale.conf  
type LANG=en_US.UTF-8 on the first line. In some cases, that might autogenerate inside of the file. In that case, simply make sure it is uncommented.  
  
<font color="royalblue"> 3.5 Network configuration </font>  
Starting off here we need to create the hostname file > touch /etc/hostname  
Then name the host! > nano /etc/hostname > put your hostname on the first line inside the file  
(I chose to keep it as archiso.)  

We need to install a network manager, which can be done with > pacman -Syu  
Then > pacman -S wpa_supplicant wireless_tools networkmanager  

<font color="royalblue"> 3.7 Root password </font>  
Run > sudo passwd root  
<font color="red"> 
# Everything Died </font>  
When I ran passwd it said it encountered a Bug and freaked out. So I had a momentary lapse in judgement, thought rebooting it would fix the issue. It was 2 am. Everything got eviscerated. So we began from scratch.   
  

<font color="royalblue"> Side Requirements </font>  
Also realized a myriad of commands needed to be installed, so ran pacman -S for things like sudo, nano, vim, etc.    
Adding a user account for myself and Codi with user permissions:  
useradd -mg users -G wheel,storage,power -s /bin/bash gamer  
changed gamers account password > passwd gamer  
added gamer to the sudo group > usermod -aG wheel gamer  
enabled sudo perms for the wheel group > visudo  
rinse and repeat with the codi account  
  
I chose to install Zsh with the following command > sudo pacman -S zsh  
To change it to the default shell, run > chsh  
  
For SSH, I installed OpenSSH like the arch linux wiki recommended > pacman -S openssh  
  
For color coding, archlinux wiki says there are five prompt strings that can be customized. PS0, PS1, PS2, PS3, and PS4. Each string is associated with a different type of prompt, primary, secondary, etc.  

As an example, I chose to change the color of PS1 and the name. So followed along with the wiki and made 2 variables > GREEN="\[$tput setaf 2)\]" and RESET="\[$(tput sgr0)\]"  then I set the PS1 variable to > PS1="$\{GREEN}\gamer${RESET}> "  
  
(please note the backslashes around GREEN is to keep it from italicizing)  
  
To add a alias to a command, run > alias newname=oldcommandname  
Before I ran the alias command, I needed to check if the nicknames I was going to give to commands were a command themself. 
Example of a alias I created > back='cd ..'  
  
Because Im a coward I chose GNOME as my desktop environment :)  
pacman -S xorg  
pacman -S gnome  

<font color="royalblue"> 3.8 Boot loader</font>  
The bootloader used for this is GRUB with the following commands >  
pacman -S grub efibootmgr  
mkdir /boot/EFI  
mount /dev/sda1 /boot/EFI  
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/EFI  
grub-mkconfig -o /boot/grub/grub.cfg  

# Reboot  
Before reboot, we need to enable the display manager and network manager >  
systemctl enable gdm.service  
systemctl enable NetworkManager.service  
systemctl enable wpa_supplicant.service  
  
Now for the moment of truth >  
exit  
shutdown now  
# Post-installation  
It worked! YIPPEE!


# Resources Used  
All hail tecmint  
https://www.linuxfordevices.com/tutorials/linux/make-arch-terminal-awesome  

https://www.tecmint.com/arch-linux-installation-and-configuration-guide/  

https://www.tecmint.com/fix-passwd-authentication-token-manipulation-error-in-linux/  

https://linuxhint.com/arch_linux_network_manager/  

https://stackoverflow.com/questions/12736351/exit-save-edit-to-sudoers-file-putty-ssh  

https://www.linuxcapable.com/sudo-privileges-in-arch-linux-add-delete-and-manage-users/  
  
https://bookdown.org/yihui/rmarkdown-cookbook/special-chars.html  
  
https://www.cyberciti.biz/tips/bash-aliases-mac-centos-linux-unix.html
</font>