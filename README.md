# sclipscomb.github.io
Sam's Installation Guide for Arch Linux (Fall 2022)

Howdy. Welcome to my Arch Linux install documentation. Just like the nerds who created Linux, I decided to make this document as unintuitive as possible (forgive me, it was not my intention). Please follow the steps below and try not to strain your eyes looking at this awful formatting. I originally wrote this in Word and exported it to GitHub pages. Did not turn out as well as I hoped.

1.	First, I downloaded VMware Workstation and activated it for personal use, I then downloaded the Arch Linux ISO file and tried to verify its GnuPG signature         using my pre-existing Windows Linux subsystem.

      a.	I had trouble getting GnuPG to work properly when trying to verify the signature, so I decided to just wing it and setup the ISO image anyway. This was             objectively terrible idea, and you should never do this when downloading a file from an HTTP mirror.
      
2.	Once my ISO file was acquired, I changed the boot mode from BIOS to UEFI in the .vmx config file. All I had to do was add a line at the bottom of the file           (firmware = “efi”) and save.
      
      a.	Note: You MUST make this change before initially booting your virtual machine, or else the BIOs environment variables take over and ruin your life.
      
3.	After restarting the entire install process from the beginning with a fresh .iso file, I was finally able to boot into Arch Linux successfully. I think I was       having trouble booting initially due to a spelling error I made in the config file. As you can see, I should have been held back a few grades.

4.	I then checked to make sure I had set UEFI correctly, by using the command ls /sys/firmware/efi/efivars and checking the UEFI config variables.

5.	I wanted to make sure the network interface was up and running, so I typed in the command ip link and hit enter, then followed up with ping eelslap.com to           verify I had a stable connection. I was stuck there for a bit until a friend told me to hit ctrl+c to break out of the ping reports. Fortunately, it works.

6.	I then synchronized my current virtual machine’s date and time with an online server, using the command timedatectl set-ntp true. I ensured my date and time         were properly updated with the command timedatectl status.

7.	Next step is to partition the virtual machine’s disk, so that I can install applications and a functioning GUI for the VM’s desktop. I will detail each step         in order down below:
      
      a.	Use lsblk to find existing hardware devices.
        
        i.	It listed loop0, sda, and sr0. I believe loop0 is the .iso file, sda is the 20 gigabytes of allocated memory, and sr0 is the boot manager itself.
        ii.	For the purposes of this project, we’re only going to be messing with sda going forward.
      
      b.	I was told by a little birdie that Arch recommends the following three partitions: EFI partition, root directory, and swap file. The same little birdie             also told me partition my disk using gdisk, since it organizes tables better than fdisk.
        
        i.	This process was highly involved, so I wrote notes for each step in shorthand below:
          
          1.	Create new table with “o” command
          2.	Create EFI partition with command “n”, then assign partition ID “1”, allow Arch to pick default first sector, set last sector to “+300" (recommended is 260 MiB), then assign GUID code “EF00” to make EFI System Partition
          3.	Create swap partition using “n” command, assign partition ID “2”, allow Arch to pick default first sector, set last sector to “+4G” (recommended is 4GiB), then assign GUID code 8200 to make Linux Swap Partition
          4.	Create root partition using command “n”, assign partition ID “3”, allow Arch to pick default first sector, also allow Arch to pick default last sector (uses remaining disk space), then assign GUID code 8300 to make Linux Filesystem Partition
          5.	Finally, write created partition table to disk using “w” command
      
      c.	Next step of the partitioning process was to format the newly created partitions.
        
        i.	First, I formatted partition sda3 into the ext4 file system using the command mkfs.ext4 /dev/sda3
        ii.	Second, I setup the swap partition with the command mkswap /dev/sda2 
        iii. Thirdly, I formatted the EFI partition using the command mkfs.fat -F32 /dev/sda1, which formatted it into FAT32
      
      d.	Final step of the partitioning process is mounting the partitions themselves. The process for this is detailed below:
        
        i.	Mounted the root volume with mount /dev/sda3 /mnt then created the mountpoint with mkdir /mnt/efi and mounted the EFI partition with mount /dev/sda1                 /mnt/efi
        ii.	I then enabled the swap partition with swapon /dev/sda2

8.	Now that partitions have been partitioned and mountpoints mounted, we can move onto installing packages. 
      
      a.	It is imperative we first get all of the necessary Linux packages. To do this, I had to find a proper mirror to download from using cat /etc/pacman.d               /mirrorlist 
      
      b.	I then installed some essential Linux packages with pacstrap /mnt base linux linux-firmware nano openssh sudo
        
            i.	At this point, the command failed due to corrupt/invalid downloaded packages, and I realized that I was using the wrong VMWare version, since I                     couldn’t create a snapshot.
            ii.	After my realization, I used the command sudo pacman -Sy archlinux-keyring to update my keys and attempted to install my downloaded packages                         again. This time it worked.
      
      c.	After running pacstrap again to download all my basic Linux files, I used the command genfstab -U /mnt >> /mnt/etc/fstab to generate a new file system               table, and then entered the new file system using arch-chroot /mnt. Finally, I was in my root directory!

9.	First thing I did upon entering the root directory was change the time zone to Central Time using ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime         and set the system clock with hwclock --systohc

10.	I then used the nano command to edit /etc/locale.gen and uncomment the “en_US.UTF-8” entry. I then ran locale-gen to generate locale files. After doing this,       I added “LANG=en_US.UTF-8” to /etc/locale.conf
      
      a.	I also changed to VirtualBox at this point and repeated the process up to this point, so I could actually take snapshots. Luckily, I was able to follow             the steps without issue back to where I was and make multiple snapshots along the way.

11.	After making the snapshot, I generated the ramdisk with the command mkinitcpio -P and set a new password using passwd.

12.	Now, I wanted to setup the bootloader so that I wouldn’t be locked out of my Arch Linux image every time I shutdown. I first installed the GRUB bootloader           using pacman -S grub efibootmgr and then used the command grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB to install GRUB into           /mnt/efi
      
      a.	I should note, GRUB apparently is the easiest bootloader to use for Arch so I went ahead and snagged it. Very easy setup.

13.	After installing GRUB I created a config file using grub-mkconfig -o /boot/grub/grub.cfg. Hopefully now I would be able to boot back into my Arch image             without any issue.

14.	Now, I exited the mount partition using exit and unmounted the partition using umount -R /mnt. Then, I rebooted using reboot and signed into “Arch Linux” as         root. And it worked!

15.	First thing I did upon rebooting into the system was create a user account for myself and give it sudo privileges. I did this with useradd -m sam followed up       by passwd sam to set my password. Next, to give sam sudo privileges, I went into visudo file with the command EDITOR=nano visudo and added the line “sam             ALL=(ALL)     ALL” under User Privilege Specification.
            
       a.	For some unknown reason, this broke sudo when I was using the user account, so I reverted to prior snapshot and tried to give sam sudo privileges                   by adding him to the sudo group with command sudo usermod -aG sudo sam. This also didn’t work, so I tried the first method again, placing the line                   at the bottom of the file, and it worked this time.

16.	Next, I installed some additional packages I was still missing with pacman -S man-db. This did not work, as my network seemed to be non-existent.
      
      a.	At this point, I realized I skipped a few crucial network configuration steps and had to go back and redo those. My steps to fix this issue are listed               below:
            
            i.	I reverted back to the snapshot closest to my first reboot and downloaded netctl and dhcpcd with pacman
            ii.	I then created a hostname “vm1” (cause I’m basic like that) and added the following lines to /etc/hosts:
                  127.0.0.1	localhost
                  ::1		localhost
                  127.0.1.1	vm1
            iii.	Following the setup of the network hosts, I attempted to setup my network card. It seems like “enp0s3” is the assigned network interface name for VirtualBox network adapters, which also has the ethernet prefix, so I set it up as such.
            iv.	I ran cp /etc/netctl/examples/ethernet-static /etc/netctl/enp0s3 to generate the network card, then edited the profile and changed the needed line to “Interface=enp0s3”
            v.	I then enabled the network card with systemctl enable dhcpcd and systemctl start dhcpcd
            vi.	After enabling and starting the network card, I rebooted back into Arch and signed in as root, then attempted to download the packages again. This time it worked!
      
      b.	I also needed to re-add my user since I reverted to an old snapshot, so I went ahead and created new user “sam”, set his password, then uncommented                 %wheel in the visudo and added “sam” to the “wheel” group to give him sudo privileges using sudo usermod -aG wheel sam

17.	 Now that all my packages were in order and my network configured, it was time to install and setup a GUI desktop. I went with LXDE since it was apparently          very lightweight and recommended in Codi’s slides.

18.	First, I downloaded all of the necessary LXDE packages with pacman -S lxde lxdm and enabled LXDM with systemctl enable lxdm 

19.	Next, to set up LXDE as the default desktop environment, I nanoed into ~/.xinitrc and added the line exec startlxde so I would automatically boot into the           desktop environment whenever I rebooted Arch.

20.	Finally, I rebooted and I was in the LXDE desktop! Hooray!
      
      a.	I’ll be honest, LXDE is hard to look at, but it gets the job done. Besides, I’ve always liked lightweight, minimalist desktops.

21.	I made a new user account for codi and assigned it sudo privileges, along with setting the password to the password specified in Harvey.

22.	After that I used ssh to access my class gateway using my given ip address and updated some apps running in the installation.

23.	I then set some aliases in the console (which I also changed to be fish by default). Some of the ones I made were alias c=’clear’, alias ports=’netstat             -tulanp’, and alias meminfo=’free -m -l -t’

24.	On top of that, I changed the desktop background to the infamous weekly event known as “Fiber Optic Friday.” If you are reading this in the future, ask Codi         about Fiber Optic Friday, it’s his favorite day of the week.

25.	Last thing to do was change the colors on my terminal. I decided on the Fallout-esque color combo of green on black, which I implemented by nanoing into             /bash.bashrc and adding the line “export PS1="\e[0;32m[\u@\h \W]\$ \e[m " at the bottom.

26.	That’s it! Arch Linux is fully setup! Never thought I’d see the day.
