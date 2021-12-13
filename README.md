# LMAE
Create your own Linux Mint Arch Edition.

> # How to create your own "LMAE" - Linux Mint Arch Edition
> 
>> ## Step 1 - Installing Arch Linux:
>>
>>> ### 1.1 - Download a Arch ISO
>>>
>>> You can download the ISO from the offical ArchLinux-website:
>>> https://archlinux.org/download/
>>
>>> ### 1.2 - Flash the ISO with the flash tool of your choice
>>> - balenaEtcher
>>> - Win32 Disk Imager
>>> - Rufus
>>
>>> ### 1.3 - Boot your Computer with your flashed USB/CD
>>
>>> ### 1.4 - Set up your keyboard layout
>>> You can list available keymaps with:\
>>> `ls /usr/share/kbd/keymaps/**/*.map.gz`\
>>> Apply the keymap of your choice with:\
>>> `loadkeys de-latin1` | Example for the german keymap
>>
>>> ### 1.5 - Check if your internet connection is working
>>> Ensure your network interface is listed and enabled. You can check this with:\
>>> `ip link`
>>>
>>> Check, if you can establish a connection:\
>>> `ping 1.1.1.1`
>>
>>> ### 1.6 - Update the system clock
>>> Ensure the system clock is accurate:\
>>> `timedatectl set-ntp true`
>>
>>> ### 1.7 - Verify the boot mode
>>> `ls /sys/firmware/efi/efivars`\
>>> If the result is "No such file or directory", that means you need to
>>> install Arch in BIOS mode. If you get a list of the efivars, you should
>>> install Arch in UEFI mode.
>>
>>> ### 1.8 - Set up your disks
>>>> #### 1.8.1 - Locate your disk
>>>> List your disks and find the disk on which you want to install LMAE:\
>>>> `fdisk -l`\
>>>> What we need, is the path of the disk, which you want to use.\
>>>> The path should look like `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`
>>>
>>>> #### 1.8.2 - Partition your disk
>>>> When partitioning I use the GPT partition table. If you want to use another one you have to know the partitioning yourself
>>>>
>>>> Example layouts:
>>>> <center>
>>>> UEFI / GPT
>>>>
>>>> | Mount point | Partition type   | Suggested size             |
>>>> |-------------|------------------|----------------------------|
>>>> | `/mnt/boot` | EFI System       | 500MB or more              |
>>>> | -           | Linux swap       | More than 512 MiB          |
>>>> | `/mnt`      | Linux filesystem | the rest of the disk space |
>>>>
>>>>  BIOS / GPT
>>>>
>>>> | Mount point | Partition type   | Suggested size             |
>>>> |-------------|------------------|----------------------------|
>>>> | -           | BIOS boot        | 500MB or more              |
>>>> | -           | Linux swap       | More than 512 MiB          |
>>>> | `/mnt`      | Linux filesystem | the rest of the disk space |
>>>> </center>
>>>>
>>>> Now open the disk you want to use in "cfdisk". This should look like this:
>>>> `cfdisk /dev/sda`\
>>>> Now choose "gpt" and press "Enter".\
>>>> Then create the partitions depending on whether you have UEFI or BIOS. After that choose the partition type, write the changes and quit "cfdisk".
>>>
>>>> #### 1.8.3 - Format your disk
>>>> Now the created partitions have to be formatted. Here we again differentiate between UEFI and BIOS.
>>>>
>>>> Example for UEFI:
>>>> - `mkswap /dev/sda2` | Format as "Linux swap (swap)"
>>>> - `mkfs.ext4 /dev/sda3` | Format as "Linux filesystem (ext4)"
>>>> - `mkfs.fat -F 32 /dev/sda1` | Format as "FAT32"
>>>>
>>>> Example for BIOS:
>>>> - `mkswap /dev/sda2` | Format as "Linux swap (swap)"
>>>> - `mkfs.ext4 /dev/sda3` | Format as "Linux filesystem (ext4)"
>>>
>>>> #### 1.8.4 - Mount your disk
>>>> First we need to mount our "Linux filesystem" to "/mnt":\
>>>> `mount /dev/sda3 /mnt`
>>>>
>>>> Second we need to mount the "Linux swap":\
>>>> `swapon /dev/sda2`
>>>> 
>>>> If you are using UEFI, we need to create the mount point "/mnt/boot" now and then mount our "EFI System" partition:
>>>> - `mkdir /mnt/boot`
>>>> - `mount /dev/sda1 /mnt/boot`
>>
>>> ### 1.9 - Installation of the base system
>>> Now we need to install all the necessary packages to get our system up and running:\
>>> `pacstrap /mnt base linux linux-firmware networkmanager grub vim`
>>
>>> ### 1.10 - Configure the system
>>>> #### 1.10.1 - Generate an fstab
>>>>
>>>> We need to generate an fstab file, which mounts our disks automatically when booting:\
>>>> `genfstab -U /mnt >> /mnt/etc/fstab`
>>>
>>>> #### 1.10.2 - Change the root to the new system
>>>> After that we should change our root to "/mnt":\
>>>> `arch-chroot /mnt`
>>>
>>>> #### 1.10.3 - Set up the system time
>>>> To change the timezone, you need to change "Region", with the region and "City" with the of your timezone:\
>>>> `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
>>>>
>>>> After that we need to sync the system time with the hardware clock
>>>> `hwclock --systohc`
>>>
>>>> #### 1.10.4 - Create the localization
>>>> - Edit "/etc/locale.gen" with your favourite editor and uncomment the locals which you want to use. You should also uncomment "en_US.UTF-8".
>>>> - To generate the locales simply run `locale-gen`
>>>> - Create the file "/etc/locale.conf" and define your preferred locale in the file. Example: `LANG=en_US.UTF-8`
>>>> - The last step is to create the file "/etc/vconsole.conf". In this file we need to set our keyboard layout. We do this by adding the line: `KEYMAP=de-latin1` instead of "de-latin1" you have to insert your preferred keyboard layout.
>>>
>>>> #### 1.10.5 - Network configuration
>>>> - Create and edit the file "/etc/hostname". Choose a computer name and insert it.
>>>> - Now create the file "/etc/hosts" and insert the following:\
>>>> "computername" should be the name you set before in the "/etc/hostname" file
>>>>
>>>>       127.0.0.1      localhost
>>>>       ::1            localhost
>>>>       127.0.1.1      computername
>>>
>>>> #### 1.10.6 - Set the root password
>>>> To change the root password, you simply type:\
>>>> `passwd`\
>>>> Now just set a new password.
>>
>>> ### 1.11 - Configuring the GRUB boot loader
>>> Again, if you want to install a different bootloader, you have to do this yourself, as I will only explain GRUB.
>>>> #### 1.11.1 - Set up GRUB with BIOS
>>>> - grub-install --target=i386-pc /dev/sda
>>>
>>>> #### 1.11.2 - Set up GRUB with UEFI
>>>> - grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB
>>>
>>>> #### 1.11.3 - Enable microcode updates
>>>> Intel-CPU: `pacman -S intel-ucode`\
>>>> AMD-CPU: `pacman -S amd-ucode`
>>>
>>>> #### 1.11.4 - Generate the grub configuraton
>>>> - grub-mkconfig -o /boot/grub/grub.cfg
>>
>>> ### 1.12 - Loading the new System
>>> - Exit the chroot environment by typing: `exit`
>>> - Unmount the partitions: `umount -R /mnt`
>>> - Boot into the new installed System: `reboot now`\
>>> Make sure your installation medium is disconnected before booting.
>>> - Log in to the new system using username "root" and your password.
>>
>>> ### 1.13 - Important changes to the new system
>>> - Enable "NetworkManager":
>>>
>>>       systemctl enable NetworkManager
>>>       systemctl start NetworkManager
>
>> ## Step 2 - Setting up the desktop environment:
>>> ### 2.1 - Adding a system user
>>> We need to create a system user for cinnamon. You can create one and set his password by executing the following:\
>>> `useradd -m username` | Replace "username" with the name you want to use. \
>>> `passwd username` | Replace "username" with the name you used before.
>>
>>> ### 2.2 - Installing the desktop environment
>>> - First, we need to download the necessary packages:\
>>> `pacman -S xorg lightdm lightdm-gtk-greeter cinnamon gnome-terminal`
>>> - Second, we need start "lightdm", to open our desktop:
>>>
>>>       systemctl enable lightdm
>>>       systemctl start lightdm
>>> - The last step is to login to our desktop using the password of the user we created before.
>>
>>> ### 2.3 - Changing the keyboard layout
>>> - Navigate to "cinnamon menu -> keyboard -> layouts"
>>> - Set the keyboard layout of your choice by adding it at the bottem left (+) and delete the default (-)
>>
>>> ### 2.4 - Set up sudo
>>> - Install sudo using: `pacman -S sudo`
>>> - Open the Terminal and login as root using: `su`
>>> - Type `EDITOR=vim visudo`, to open the "sudoers"-file.
>>> - Under "User privilege specification" we should add our user. This should look like:
>>>
>>>       ##
>>>       ## User privilege specification
>>>       ##
>>>       root ALL=(ALL) ALL
>>>       username ALL=(ALL) ALL
>>>
>>> - Save and close the file.
>>> - Now change back to the user account with: `su username`
>>> - Sudo is now set up and ready to use.
>>
>>> ### 2.5 - Activating the AUR
>>> To use the AUR, we need to install yay:
>>> - `sudo pacman -S git base-devel` | Install necessary packages.
>>> - `cd ~` | Change directory to the users home.
>>> - `git clone https://aur.archlinux.org/yay.git` | Download yay.
>>> - `cd yay` | Change directory to the folder we have downloaded.
>>> - `makepkg -si` | Build package and install yay.
>>> - `cd ..`| Go back to the home directory.
>>> - `rm -rf ./yay/` | Delete the yay-folder.
>>> - `yay -Syy` | Make sure yay is working and sync the package database.
>>
>>> ### 2.6 - Making cinnamon look like on Linux Mint
>>>> #### 2.6.1 - Installing the Fonts
>>>> - `yay -S noto-fonts noto-fonts-emoji`
>>>> - Navigate to "cinnamon menu -> Font Selection" and change it to the following:
>>>>
>>>> <center>
>>>>
>>>> |                   | Font              | Size |
>>>> |-------------------|-------------------|------|
>>>> | Default font      | Ubuntu Regular    | 10   |
>>>> | Desktop font      | Ubuntu Regular    | 10   |
>>>> | Document font     | Sans Regular      | 10   |
>>>> | Monospace font    | Monospace Regular | 10   |
>>>> | Window title font | Ubuntu Medium     | 10   |
>>>>
>>>> </center>
>>>>
>>>
>>>> #### 2.6.2 - Installing the Mint-Themes and icons
>>>> - `yay -S mint-themes mint-y-icons mint-x-icons`
>>>> - Navigate to "cinnamon mennu -> Themes" and choose the mint themes, which you want to use."
>>>> - If you want to use the "Linux Mint backgrounds", you can install them by executing: `yay -S mint-backgrounds`\
>>>> Then just choose your favourite at "cinnamon menu -> Backgrounds"
>>
>>> ### 2.7 - Adding printer support
>>> - `yay -S cups system-config-printer`
>>> - `sudo systemctl enable cups`
>>> - `sudo systemctl start cups`
>>
>>> ### 2.8 - Installing the default Linux Mint programs
>>>> #### 2.8.1 - Installing programs from category "Accessories"
>>>> - `yay -S file-roller yelp warpinator mintstick xed gnome-screenshot redshift seahorse onboard sticky xviewer gnome-font-viewer bulky xreader gnome-disk-utility gucharmap gnome-calculator`
>>>
>>>> #### 2.8.1 - Installing programs from category "Graphics"
>>>> - `yay -S simple-scan pix drawing`
>>>
>>>> #### 2.8.2 - Installing programs from category "Internet"
>>>> - `yay -S firefox webapp-manager hexchat thunderbird transmission-gtk`
>>>
>>>> #### 2.8.3 - Installing programs from category "Office"
>>>> - `yay -S gnome-calendar libreoffice-fresh`
>>>
>>>> #### 2.8.4 - Installing programs from category "Programming"
>>>> - `yay -S python`
>>>
>>>> #### 2.8.5 - Installing programs from category "Sound & Video"
>>>> - `yay -S celluloid hypnotix rhythmbox`
>>>
>>>> #### 2.8.6 - Installing programs from category "Administration"
>>>> - `yay -S baobab gnome-logs timeshift`
>>>
>>>> #### 2.8.7 - Installing programs from category "Preferences"
>>>> - `yay -S gufw blueberry mintlocale`
