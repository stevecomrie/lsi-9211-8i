# Tutorial: Updating IBM M1015/LSI 9211-8i Firmware on UEFI Systems

This is an archive of a [2016 post by ZzBloopzZ](https://forums.servethehome.com/index.php?threads/tutorial-updating-ibm-m1015-lsi-9211-8i-firmware-on-uefi-systems.11462/) 
 on the servethehome.com forums. 

I've seen enough tutorials & zip file download links go 404 in my day to risk not having a back-up.

## Purpose
A proper MODERN tutorial on updating the firmware on the IBM M1015 on a UEFI system, in my case my IBM M1015's were already crossflashed to LSI 8211-8i long ago with the older P16 firmware which I wanted to update to the latest firmware. It is the same process either way. This tutorial is for IT-Mode.  
  
Also, I have 2x of these cards in my machine and this tutorial will allow you to grab the SAS address and also flash each individual card one at a time WITHOUT having to physically take out the cards. This saves time and hassles.  
  
I had to look at 3-4 tutorials and help documents to get this process to work smoothly. Spent several hours mind trying to figure this out so don't want others to waste as much time as I did. :c)  
  
## Part 1 - Downloading Necessary Files and Preparing USB Flash Drive  

### Download the following:

[a) LSI-9211-8i.zip](https://www.mediafire.com/download/6mtie10d9ud6675/LSI-9211-8i.zip) (Custom Flashing Tools) 

[b) Rufus](https://rufus.akeo.ie/) (Simple Free Bootable USB Creation Tool)  

c) [LINK](http://www.avagotech.com/products/server-storage/host-bus-adapters/sas-9211-8i#downloads) (Click Firmware. Download the following:  
Installer_P##_for_UEFI (## stands for latest current firmware)  
9211-8i_Package_P##_IR_IT_Firmware_BIOS_for_MSDOS_Windows (## stands for latest current firmware)  

### Prepare USB Flash Drive   

Try to use a USB 2.0 or under flash drive, and one without any important data on it as it will be WIPED!

a) Open Rufus. Create bootable flash drive with the following settings: Volume Label: SAS2008, FAT32, MS-DOS, Deselect “Create extended label and icon files"  

b) Open LSI-9211-8i.zip and extract JUST THE FILES to the root of the flash drive. IGNORE all of the folders as they are unnecessary.  

c) Open 'Installer_P##_for_UEFI.zip' and locate the "sas2flash.efi" file and extract just that file to the root of the flash drive. Overwrite when asked.  

d) Open '9211-8i_Package_P##_IR_IT_Firmware_BIOS_for_MSDOS_Windows.zip' and the navigate to the 'Firmware' folder. Locate the '2118it.bin' file and extract just that file to the root of the flash drive. Overwrite when asked. Then navigate to the 'sasbios_rel' folder and extract both 'mptsas2.rom' & 'mptbios.txt' to the root of the flash drive. Overwrite when asked.  

e) Safely eject the USB Flash Drive.  

f) Your flash drive is now prepared!  
  
## Part 2: Flashing The HBA Cards
  
1.  Connect USB flash drive to the server in a USB 2/1.1/1 port (Avoid USB 3.0 port). As a precaution, I disconnected all other USB flash drives.
2.  Boot from the flash drive via UEFI (F11 on Super Micro boards, then select “UEFI: Built-in EFI Shell”)  
    
3.  Type "fs0:" to access USB drive. (Note: May have to type "map -b" first to determine device name if you have several USB drives.)  
    
4.  Type "sas2flash.efi -listall" (Lists all SAS cards installed, for me 0 was first card and 1 was the second)  
    
5.  Type "sas2flash.efi -c 0 -list" (To get SAS Address of the first card, write it down or take picture with phone)  
    
6.  Type "sas2flash.efi -c 1 -list" (To get SAS Address of the second card, write it down or take picture with phone)
7.  Reboot the system but this time boot to the USB Flash Drive via DOS (normal/non UEFI).
8.  Type: “megarec -writesbr 0 sbrempty.bin” (Writes an empty sbr to for card #1)  
    
9.  Type: “megarec -cleanflash 0” (Erases controller flash for card #1)  
    
10.  Type: “megarec -writesbr 1 sbrempty.bin” (Writes an empty sbr to for card #2)  
    
11.  Type: “megarec -cleanflash 1” (Erases controller flash for card #2)  
    
12.  Reboot the system and again boot from the flash drive via UEFI . (F11 on Super Micro boards, then select “UEFI: Built-in EFI Shell”)
13.  Type "fs0:" to access USB drive. (Note: May have to type "map -b" first to determine device name if you have several USB drives.)
14.  Type: “sas2flash.efi -c 0 -o -f 2118it.bin” (Flash new firmware with IT-Mode for card #1)
15.  Optional: If you want the HBA Boot BIOS then type: "sas2flash.efi -c 0 -o -b mptsas2.rom" (For card #1) [I skipped this since I do not need it for my use.]  
    
16.  Type: “sas2flash.efi -c 0 -o -sasadd 500605B#########” (Program SAS Address for card #1, replace #'s with what your wrote down for the first card. Put numbers only, skip the dashes/hyphons.)
17.  Optional: If you want the HBA Boot BIOS then type: "sas2flash.efi -c 1 -o -b mptsas2.rom" (For card #2) [I skipped this since I do not need it for my use.]  
    
18.  Type: “sas2flash.efi -c 1 -o -f 2118it.bin” (Flash new firmware with IT-Mode for card #2)  
    
19.  Type: “sas2flash.efi -c 1 -o -sasadd 500605B#########” (Program SAS Address for card #2, replace #'s with what your wrote down for the second card. Put numbers only, skip the dashes/hyphons.)  
    
20.  Verify Flash: "sas2flash.efi -listall"
21.  Type: “exit”
22.  Enjoy!

  
## Notes
  
1. I did it in this exact order/method because I heard some people having odd issues or even bricking their cards by not erasing cards or via the NOT recommended "sas2flash.efi -o -e 6" before updating the firmware. This could be FUD but I saw few people have issues doing it this way.  
2. Also heard of problems with flashing the .bin and BIOS .rom in the same step with the .efi version of the sas2flash tool and that is why I put them in separate independent steps.  
3. It is highly suggested to use MATCHING versions of the flash tool (sas2flash), .bin and .rom. For example, if it is P20 use the P20 flash tool, P20 .bin and P20 .rom. The only EXCEPTION is that if for some reason you cannot downgrade or it is not updating then try using the version P5 of the flash tool.  
4. Better to get SAS Address via sas2flash.efi because megacli.exe sometimes reads the cards in different order or even not at all compared with sas2flash. Since we are programming with sas2flash mind as well use same tool to get the SAS Address just to be safe.  
  
  
### P20 Myth
  
There are many revisions of the P20 firmware. The first few had lots of bugs so people suggest avoiding it. However, someone pointed out that all versions that are 20.00.04 and after are fine. I using the current latest version 20.00.07 and have had no problems. Did a full scrub, bonnie benchmark, copied large files and played them back with no errors.  
  
  
## Additional Troubleshooting
  
[matt's Tutorial](https://techmattr.wordpress.com/2016/04/11/updated-sas-hba-crossflashing-or-flashing-to-it-mode-dell-perc-h200-and-h310/)  
[STH Tutorial](https://www.servethehome.com/ibm-serveraid-m1015-part-4/)  
  
Special thanks to mobilenvidia, mattr, linus, STH community for their tutorials and help.  

### Misc Forum Posts
https://forums.servethehome.com/index.php?threads/tutorial-updating-ibm-m1015-lsi-9211-8i-firmware-on-uefi-systems.11462/

http://codefromabove.com/2017/03/crossflash-ibm-m1015-to-lsi-9220-8i-it-mode-for-freenas/

https://www.truenas.com/community/threads/help-reflashing-lsi-9220-8i-to-it-mode.59176/

https://www.truenas.com/community/threads/ibm-serveraid-m1015-and-no-lsi-sas-adapters-found.27445/page-2#post-301617

https://www.truenas.com/community/threads/sas2flash-easier-alternative.30789/

https://forums.laptopvideo2go.com/topic/29059-sas2008-lsi92409211-firmware-files/