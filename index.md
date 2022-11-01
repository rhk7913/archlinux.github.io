# Installing Arch Linux

This page outlines my process of installing Arch Linux on VMware Workstation. 

I used the following resources in this process:
1. Arch Linux wiki: https://wiki.archlinux.org/title/Installation_guide
2. Checking file hash: https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000LWYqSAO
3. Creating and formating partitioning: https://itsfoss.com/install-arch-linux/
4. Network manager: https://linuxhint.com/arch_linux_network_manager/

***Always use the default option when running any commands, unless specified to use a different option!***

# Pre-Installation

On the Arch Linux downloads page, https://archlinux.org/download/Download, download a HTTP direct download file (with the extension .iso) based on your country. 

I used this link from RIT: http://mirrors.rit.edu/archlinux/iso/2022.10.01/, where the file was named archlinux-2022.10.01-x86_64.iso.

After downloading your specified .iso file, use the following PowerShell command to ensure the file has a SHA256 hash. 

```
cd downloads
certutil -hashfile yourfile.iso SHA256
```
        
For instance, these were my specific PowerShell commands:

```
cd downloads
certutil -hashfile archlinux-2022.10.01-x86_64.iso SHA256
```
        
After running the PowerShell command, return to the Arch Linux downloads website and locate the checksums section under HTTP Direct Downloads. Compare the resulting hash from PowerShell to the checksum on the website. 

If the hashes match, the integrity of the file has been verified and you are able to proceed to the installation process. If they don't match, the .iso file may be corrupted and you may need to install another .iso file to install. 

# Installation

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

Locate the folder that contains your VM files. Find the file with the virtual machine configuration (.vmx) extension. You may need to use a different platform to edit the file. I chose to use Notepad++. 

Insert the following lines to lines 2 and 3 in the .vmx file and save your changes.

```
firmware="efi"
bios.bootdelay=5000 
```
		
**The first command changes the default BIOS firmware on my VM to UEFI firware. While it is possible to just keep the BIOS firmware, I found that a majority of my resources ended up using UEFI, so I decided to change the firmware to make my installation process smoother.**

**The second command extended my time to choose the correct mode when booting up the, since I would miss it the first few tmes I attempted to select it.** 

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
 
Select a applicable time zone. I used America/Chicago: 

```
timedatectl set-timezone America/Chicago
```
  
Run the following command to ensure that the time zone has been updated: 

```
timedatectl
```

## Partitioning

**Partitioning refers to the dividing of the hard drives** - change this

**For the EFI mode, two partitions must be made. The first one is the /dev/sda partition, which involves the hard drive of the computer currently being used (EFI system partition). The second one is the root partition, which is associated with the VM software.** 

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

The following command will save your newly created partitions and exit the partitioning platform:

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

Mounting allows the .iso file to access its contents as if it was on a physical medium and then inserted into the optical drive. 

The following command will mount the .iso file:

```
mount /dev/sda2 /mnt
```
	
## Install essential packages

The Arch Linux Installation guide states that the base, linux, and linux-firmware packages must be installed, as they are curcial for the proper running of the system.

I ended up also installing the man, sudo, vim, and zsh packages to use in the later parts of the installation process. 

The following command will install the base, linux, linux-firmware, man, sudo, vim, and zsh packages:

```
pacstrap /mnt base linux linux-firmware man sudo vim zsh base-devel
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

The following command will set and change the time zone. I used America/Chicago as my time zone:

```
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```
  
**Localization**

Localization sets your system's language, numbering, date, and currency formats. 

The following commands will allow you to edit the /etc/locale.gen file and uncomment your specific localization. Mine is the en_US.UTF-8 UTF-8 localization.

```
vim /etc/locale.gen
Press "Insert" and remove the # in front of en_US.UTF-8 UTF-8
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

## Network Configuration

**"myarch" was my chosen name for my network host**

Begin by writing the selected host name to the /etc/hostname file:

```
echo myarch > /etc/hostname
```
  
Verify that the previous command worked: 

```
cat /etc/hostname
```
  
You will then need to edit the /etc/hosts file
 
Use the following command to open the /etc/hosts file in vim: 
 
```
vim /etc/hosts
```

After entering vim, do the following: 

To begin typing, press "insert" and add these three lines to the file: 

```
127.0.0.1		localhost
::1			localhost
127.0.1.1		myarch
```
        
Press "ESC" and type :wq to save the changes to the /etc/hosts file and to exit vim. 

## Setup network manager

The following commands will install a network manager:

```
pacman -Syu
pacman -S wpa_supplicant wireless_tools networkmanager
```

## Change the root password

The following command will guide you through changing your root password:

```
passwd
```
## Installing a desktop environment (DE)

_**The website that I used for installing a DE provided steps for installing the Gnome DE.**_

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

There should be a option to boot into the GUI with a start-up menu to similar that of the beginning of the installation progress. 

**Overall thoughts:** While a generally a challenging task, installing Arch Linux allowed me to greatly improve my understanding of the command line interface. In addition, this installation improved my ability to search for external resources. 
