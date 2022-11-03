# Installing Arch Linux

This page outlines my process of installing Arch Linux on VMware Workstation. 

# Pre-Installation

Hashing reference: https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000LWYqSAO

On the Arch Linux downloads page, https://archlinux.org/download, download a HTTP direct download file (with a .iso extension) based on your country. 

I used this link from RIT: http://mirrors.rit.edu/archlinux/iso/2022.10.01/, where the file was named archlinux-2022.10.01-x86_64.iso.

After downloading your specified .iso file, use the following PowerShell commands to ensure the file has a SHA256 hash. 

```
cd downloads
certutil -hashfile yourfile.iso SHA256
```
        
For instance, these were my specific PowerShell commands:

```
cd downloads
certutil -hashfile archlinux-2022.10.01-x86_64.iso SHA256
```
        
After running the PowerShell commands, return to the Arch Linux downloads page and locate the checksums section under HTTP Direct Downloads. Compare the resulting hash from PowerShell to the checksum on the website. 

If the hashes match, the integrity of the file has been verified and you are able to proceed to the installation process. If they don't match, the .iso file may be corrupted and you may need to install another .iso file to install. 

# Installation

***Always use the default option when running any commands, unless specified to use a different option!***

## Setup the virtual machine (VM) in VMware
Open up VMware Workstation and locate "File" in the upper left hand corner. 

Select "New Virtual Machine.

Select the "Typical (recommended)" option for the type of configuration.

Select the "Installer disc image file (iso):" option for the guest operating system installation step

Select "Linux" for the guest operating system and set the version to "Other Linux 5.x and later kernel 64-bit"

Name your virtual machine, preferably something that references Arch Linux. 

Set the maximum disk size to 20 GB and select "Split virtual disk as a single file" for the specified disk capacity step.

Select "Customize Hardware..." and set the memory to 2 GB.

Hit finish, but do not power on the VM yet! You will need to alter a portion of one of the VM's file to ensure that the correct firmware is being used on the VM.

Locate the folder that contains your VM files. Find the file with the virtual machine configuration (.vmx) extension. You may need to use a different platform to edit the file instead of the default notetaking app on your computer. I chose to use Notepad++. 

Insert the following lines to lines 2 and 3 in the .vmx file and save your changes.

```
firmware="efi"
bios.bootdelay=5000 
```
		
**Notes:** 
1. The first command changes the default BIOS firmware on my VM to UEFI firware. While it is possible to just keep the BIOS firmware, I found that a majority of my resources ended up using UEFI, so I decided to change the firmware to make my installation process smoother.
2. The second command extended my time to choose the correct mode when booting up the VM, since I kept missing it the first few tmes I tried to select it. 

You can now power on the VM! 
	
## Power on the VM 
After powering on the VM, locate and select the "Arch Linux install medium (x86_64, UEFI)" option.

## Verify that the VM is in UEFI mode
After entering the "Arch Linux install medium (x86_64, UEFI)" option, it may take a while for the command line interface to appear. 

When the root@archiso does eventually show up, you should have the ability to run commands. If no errors appear, you are good to continue with the process.

The following command will ensure that the UEFI mode is being used:

```
ls /sys/firmware/efi/efivars	
```
## Update the system clock
The following command will check which time is currently being used:

```
timedatectl
```

The following command will show you the available time zones you can choose from (press q to exit the list):

```
timedatectl list-timezones
````
 
The following command will select a applicable time zone:

```
timedatectl set-timezone yourtimezone
```
  
For instance, I ran the following command to select the America/Chicago timezone:

```
timedatectl set-timezone America/Chicago
```

Run the following command to ensure that the time zone has been updated: 

```
timedatectl
```

## Partitioning

Reference: https://itsfoss.com/install-arch-linux/

Partitioning is the process of dividing the hard drive into multiple selections. 

**Note:** For the EFI mode, two partitions must be made. The first one is the /dev/sda partition, which involves the hard drive of the computer currently being used (EFI system partition). The second one is the root partition, which is associated with the VM software.

The following command will display the available hard drives:

```
fdisk -l
```
  
The following commands will create a EFI system partition: 

```
fdisk /dev/sda
Command (m for help): n
Partition type: p
Partition number: 1
First sector: default option (2048)
Last sector: +500M
Command (m for help): t
Hex code or alias (type L to list all): EF
```
 
The following commands will create a root partition:

```
Command (m for help): n
Partition type: p
Partition number: 2
First sector: default option
Last sector: default option
```

The following command will save the newly created partitions and exit the partitioning platform:
```
Command (m for help): w
```

## Format partitions

After the partitions have been created, each partition must be formatted with the appropriate file system. 

The following commands will format the partitions:
```
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```
  
## Mount the file system

Reference for explanation of mounting's purpose: https://man.archlinux.org/man/mount.8.en

Mounting the .iso file allows access as if the file was on a physical device. 

The following command will mount the .iso file:
```
mount /dev/sda2 /mnt
```
	
## Install essential packages

The Arch Linux Installation guide states that the base, linux, and linux-firmware packages must be installed, as they are crucial for the proper running of the system.

I ended up also installing the man, sudo, vim, and zsh packages to use in the later parts of the installation process. 

**Note:** I chose to install vim over nano, since I wanted to gain familarity with editing in vim. However, if you have not edited with nano before, I would recommend using nano over using vim, since vim is slightly more complicated to use as a editor. 

The following command will install the base, linux, linux-firmware, man, sudo, vim, and zsh packages:
```
pacstrap /mnt base linux linux-firmware man sudo vim zsh
```
	
## Configure the system 

**Create a fstab file**

A fstab file is used to define how disk partitions, block devices, or remote file systems should be mounted to the file system. 

The following command will create the fstab file:
```
genfstab -U /mnt >> /mnt/etc/fstab
```
    
The following command will check if the fstab file has been created correctly:
```
cat /mnt/etc/ftsab
```
	 
**Change root (chroot)**

Chroot changes the root directory for the current running process and their children. Running a program in this modified environment means that any files or commands outside of the environment cannot be accessed. In this instance of chroot, you would enter the mounted disk as root. 

The following command will change root into the new system:
```
arch-chroot /mnt
```
  
**Time zone**

The following command will set and change the time zone:
```
ln -sf /usr/share/zoneinfo/yourtimezone /etc/localtime
hwclock --systohc
```
  
For instance, I ran the following command to set America/Chicago as my time zone:
```
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```
 
**Localization**

Localization sets your system's language, numbering, date, and currency formats. 

The following commands will allow you to edit the /etc/locale.gen file and uncomment your specific localization. Mine is the en_US.UTF-8 UTF-8 localization.
```
vim /etc/locale.gen
Press "Insert"
Remove the # in front of en_US.UTF-8 UTF-8
Press "ESC" and enter :wq to save the file and exit vim
```

The following command will create the /etc/locale.conf file and set the LANG variable:
```
echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
```
	
The following command will check that the locale.conf file was created correctly:
```
cat /etc/locale.conf
```

**Note:** If you uncomment the wrong localization, you may run into issues opening your terminal when you get using your GUI. I accidentally uncommented the localization right above en_US.UTF-8 UTF-8 and I kept having trouble opening my terminal the first few times I used the GUI. I ended up having to edit /etc/locale.gen again and the terminal functioned correctly again!

## Network Configuration

Reference: https://linuxhint.com/arch_linux_network_manager/

**Npte:** "myarch" was my chosen name for my network host

The following command will write the selected host name to the /etc/hostname file:
```
echo myarch > /etc/hostname
```
  
The following command will verify that the previous command worked: 
```
cat /etc/hostname
```
  
You will then need to edit the /etc/hosts file.
 
The following command will open the /etc/hosts file in vim: 
```
vim /etc/hosts
```

After entering vim, do the following: 

Enter the following three lines to the file: 

```
127.0.0.1		localhost
::1			localhost
127.0.1.1		yourhostname
```

For instance, I inserted the following three lines to the file:
```
127.0.0.1		localhost
::1			localhost
127.0.1.1		myarch
```
        
Press "ESC" and type :wq to save the changes to the /etc/hosts file and to exit vim. 

## Setup network manager

**Note:** Do not use the apt command to install any package or manager in Arch! pacman is the correct command for installation related commands in this distribution. I accidentally used apt in my first attempt to install the network manager and it would not install. I then realized it was the command used for Ubuntu, Debian, and other related distributions, not Arch.

The following commands will install a network manager:
```
pacman -Syu
pacman -S wpa_supplicant wireless_tools networkmanager
```

## Change the root password

The following command will guide you through changing your root password. You will need the root account and password to log into the GUI for the first time.
```
passwd
```

## Installing a desktop environment (DE)

**Note:** The website that I used for installing a DE provided steps for installing the Gnome DE.

The following commands will install the Gnome DE: 
```
pacman -S xorg
pacman -S gnome
```

## Enable display manager and network manager

The following commands will enable both the display manager and the network manager:
```
systemctl enable gdm.service
systemctl enable NetworkManager.service
systemctl enable wpa_supplicant.service	
```   

## Finish
Exit out of chroot and shutdown the VM:  
```
exit 
shutdown
```

When the VM shutdowns, turn it back on and select the option to enter the GUI. 

Power on the VM again and select the "Arch Linux" option when the start-up menu appears. You now have your Arch Linux running as a GUI! 

# Further modifications

**Note:** I ran most of my commands with a sudo user, so I omitted the sudo parts of the commands. If you are running these commands with a non-sudo user, you will need to add sudo to the beginning of your commands.

## Adding additional accounts
The following command will add a user to the system:
```
useradd name
```

For instance, I ran the following commands to add the users codi and rhea:
```
useradd codi
useradd rhea
```

## Adding aliases

Reference: https://www.tecmint.com/create-alias-in-linux/

Aliases are customizable shortcuts that allow you to access your commonly used commands without having to always use the full form of the command. 

The following command will show a list of your current aliases:
```
alias
```
There are two types of aliases: temporary and permanent. 

**Note:** Temporary aliases are only active in your current shell. If you exit out of your current shell, these aliases will no longer exist. On the other hand, permanent aliases will remain on the shell. 

The following syntax will create a temporary alias:
```
alias name='commandname'
```

For instance, creating an alias for the ls -la command would look like:
```
alias ll='ls -la'
```

To create a permanent alias, you will need to enter the ~/.bashrc file and append your alias to the file, using the above syntax:
```
vim ~/.bashrc
alias name='commandname'
Save and close the file
```

The following command will activate your permanent alias:
```
source ~/.bash_aliases
```
 
## Installing a different shell

**Note:** You should have the zsh shell already installed from the installation process, however it is a good idea to still check and see if the shell was properly installed.

The following command will install the zsh shell:
```
pacman -S zsh
```

## Installing ssh and enter the class gateway

In order to enter the class gateway, you will need to install the ssh package.

The following command will install the ssh package:
```
pacman -S ssh
```

The following command will connect you to the class gateway:
```
ssh -p yourportnumber yourusername@server-address
```

For instance, I ran the following command to enter the class gateway:
```
ssh -p 22 sysadmin@10.10.1.118
```

## Color-coding the terminal

Reference: https://averagelinuxuser.com/linux-terminal-color/

There are two ways to color-code bash: either on a user specific basis or on a system-wide level. It is recommended to only edit on a user specific basis.

The following process will enable color-coding on a user specific basis:
```
Backup the setting file: cp .bashrc .bashrc.backup
Edit your bash.bashrc file with your wanted color scheme
Move the bash.bashrc file to your home directory: mv .bashrc ~/.bashrc
Restart the terminal and the color scheme should appear
```

# Other requirements relating to the video

## Show VM's IP address

The following command will show the VM's ip address:
```
ip addr
```

## Showing all users

Reference: https://linuxhint.com/list_all_users_linux_system/

The following command displays only the users:
```
compgen -u
```
However, the following command will display all the users and their additional info via the /etc/passwd file:
```
cat /etc/passwd
```

## Showing sudoers

Reference: https://linuxopsys.com/topics/add-user-to-sudoers-in-arch-linux

**Note:** There is no group named sudo in Arch! The group containing the sudo users is called wheel.

The following command adds a user to the wheel group:
```
usermod -aG wheel username
```

For instance, I ran the following commands to add the users codi and rhea to the wheel group:
```
usermod -aG wheel codi
usermod -aG wheel rhea
```

The following command displays the sudo users:
```
getent group wheel
```

## Installing a Arch User Repository (AUR) package

Reference: https://linuxhint.com/aur_arch_linux/

The following command will syncronize all your packages:
```
pacman -Syu
```

The following command will install the base-devel package, a required tool in compling software from the AUR:
```
pacman -S git base-devel
```

The following command will install your selected AUR package:
```
git clone https://aur.archlinux.org/<pkgName>.git
```

The following command will change into the AUR package's directory:
```
cd packagename
```
**Note:** You will be unable to run makepkg as a root user, so you will need to run the following step as a non-root user to continue the installation process.

The following command will finish the installation 
process of installing the AUR package:
```
makepkg
sudo pacman -U packagename.tar.xz
```
Your AUR package should be installed!

# Overall thoughts
1. While installing Arch was a challenging task, I felt that it allowed me to greatly improve not only of understanding of the Linux command line interface, but also my understanding of how and what is required to create and maintain a Linux distribution.  
2. In addition, I felt that my ability to troubleshoot for Linux related tasks improved. While I have had to use external resources for other related courses, I haven't use them much for anything related to Linux.
