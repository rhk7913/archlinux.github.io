# Arch Linux Installation Documentation

This page outlines my process of installing Arch Linux on VMware Workstation. 

I used the following resources to assist me:
1. Arch Linux wiki: https://wiki.archlinux.org/title/Installation_guide
2. Checking file hash: https://portal.nutanix.com/page/documents/kbs/details?targetId=kA07V000000LWYqSAO
3. Partitioning/installing the DE: https://itsfoss.com/install-arch-linux/
4. Network manager: https://linuxhint.com/arch_linux_network_manager/

***Always use the default option when running any commands, unless specified to use a different option!***

## Pre-Installation

1. On the Arch Linux downloads page, https://archlinux.org/download/Download, download a HTTP direct download file (with the extension .iso) based on your country. I used this link from RIT: http://mirrors.rit.edu/archlinux/iso/2022.10.01/, where the file was named archlinux-2022.10.01-x86_64.iso.
2. After downloading your specified .iso file, use the following PowerShell command to ensure the file has a SHA256 hash. 

        cd downloads
        certutil -hashfile yourfile.iso SHA256
        
3. For instance, these were my specific PowerShell commands:

        cd downloads
        certutil -hashfile archlinux-2022.10.01-x86_64.iso SHA256
        
4. After running the PowerShell command, return to the Arch Linux downloads website and locate the checksums section under HTTP Direct Downloads. Compare the resulting hash from PowerShell to the checksum on the website. 
5. If the hashes match, the integrity of the file has been verified and you are able to proceed to the installation process. If they don't match, the .iso file may be corrupted and you may need to install another .iso file to install. 

#### Installing a desktop environment (DE)

**The website that I used for installing a DE provided steps for installing the Gnome DE.**

The following commands will install the Gnome DE: 

	pacman -S xorg
	pacman -S gnome
  
#### Enable display manager and network manager

The following commands will enable both the display manager and the network manager:

	systemctl enable gdm.service
	systemctl enable NetworkManager.service
	systemctl enable wpa_supplicant.service	
        
#### Finish

Exit out of chroot and shutdown the VM: 
        
    exit 
    shutdown

When the VM shutdowns, turn it back on and select the option to enter the GUI. There should be a option to boot into the GUI with a start-up menu to similar that of the beginning of the installation progress. 

####

**Overall thoughts:** While a generally a challenging task to accomplish, installing Arch Linux allowed me to greatly improve my understanding of the command line interface. In addition, this installation improved my ability to search for external resources. 
