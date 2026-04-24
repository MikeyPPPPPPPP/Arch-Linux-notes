Slow WIFI

1. Type "lspci -k" to get the devices

Qualcomm Atheros QCA6174 802.11ac Wireless Network Adapter 

A. I need to install the firmware.


Firmware installation process:

1. I already have the firmware.

Looks like I need to remove then reactivate the firmware:

2. sudo modprobe -r ath10k_pci && sudo modprobe ath10k_pci


Slow due to hardware encryption problem. Looks like I need to remove it and force software encryption:

sudo modprobe ath10k_pci nohwcrypt=1

To enable persictance on boot I need to add a file to /etc/modprobe.d/ with the command inside. (they say to make a filename with a meaningful name)

I'll name the file ath10k
Now I'll restart, did it work?
I think, but time will tell.
----------------------------------------------------------

Format SSD

The SSD has stuff on it from the windows but its NFTS which is not a problem, I just need a tool like nfts-3g. There was nothing of interest on is, actually I thought I backed it up. Now I still don't think I'll back it up.


I want two partitions, a normal and an LUKS encrypted partition. I was thinking one for games and one for files. Its 1TB so a 400 / 500 ratio should due as I really only want Verdum and GTA V.


I'll make both partitions but for now only mount the 400GB partition gor GTA V for now. Here I go!

Tree:

sda
|_sda1  400GB   ext4 part /mnt/steam_games
|_sda2  531.5GB ext4 part /mnt/vault


1. lsblk to show that the SSD is sda, it does show 2 partitions.

Note: I wont install nfts-3g because it'll add bloat I don't want;)

Process:

1. I need to format it. 
sudo gdisk /dev/sda

command: x
Expert command: z
Whipe out GPT on /dev/sda: Y
Blank out MBR?: Y

2. I need to make the partitions.

sudo gdisk /dev/sda
command: n
partition number: 1
First sector: 
Last sector: 400GiB
command: w

do you want to proceed?: Y


3. Now I need to creat the filesystem and mount the partition and add precistance.

sudo mkfs.ext4 /dev/sda1
sudo mkdir /mnt/steam_games
sudo mount /dev/sda1 /mnt/steam_games
lsblk -f
sudo nano /etc/fstab
UUID=fb0d2f84-b34c-40b3-8325-7effe6621c83	/mnt/steam_games	ext4	defaults	0 2

reboot to check the precistance.
It worked!

---------------------------------------------------------------


Set Steam to use the new partition 

when I tryed to add it in settings, it wouldn't appear.

then I checked the directory permissions and it could only be used by root.

A. I'll watch my Youtube video and change the permissions of that file. I want them to be "michael users" instead of "root root".

sudo chown michael:users /mnt/steam_games

It worked!
---------------------------------------------------------------


# Intalling the graphics driver for my nvidia card.


1. find the card:
lspci -k -d ::03xx

2. run nvidia-smi to get card info

It says off net to the card :(

3. Now to turn it on. I'll start with nvidia-smi -h

The docs say to use -pm or --persistence-mode=   0/disabled 1/enabled
since I only have one nvidia GPU I'll do:

sudo nvidia-smi -pm 1

4. I need to install ttf-ms-fonts for the RockStar Launcher.

	First I needed to install git. 
	: sudo pacman -S git
	
	Now I can install ttf-ms-fonts with github because I dont have yay.
	: git clone https://aur.archlinux.org/ttf-ms-fonts.git

	-s syncdeps  -i install
	: sudo makepkg -s -i ttf-ms-fonts/PKGBUILD
	
5 Add game launcher option no ESYNC.
	
	PROTON_NO_ESYNC=1


---------------------------------------------------------------



I wanted to switch wifi networks aso Iused this command:

nmcli device wifi list
nmcli device wifi connect "proveitallnight"

---------------------------------------------------------------


I want to make it so it doesn't shutoff whem I close the lid, the arch wiki si telling me to run a shrt command
```
.zshrc
xset s off
```



SSH problem with starting the sshd.
---------------------------------------------------------------


So I tryed "sudo systemctl start sshd" and it gave me an error and also sead to check the journalctl to check the logs and gave me a command to run. Onec the journal logs were up I started reading that systemd was started to run the sshd but sshd.service was the problem. It also said "An ExecStart= process belonging to unit sshd.service has exited. the process exit code is exited and its status is 255." After asking Grok about ssh problems and mentinoing systemd and ExecStart he was able to narrow it down to the sshd user not being their and after checking /etc/passwrd I was able to confirm there was no sshd user.
```sudo useradd -r -U -d /var/empty/sshd -s /usr/bin/nologin -c "sshd prililege-seperation user" sshd```
I also needed to update the keyring and reupdate the laptop.

That Worked!!!!
# Bashrc

```
#
# ~/.bashrc
#
# If not running interactively, don't do anything
[[ $- != *i* ]] && return

alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias ll='ls -la'
alias cls='clear'
alias vault='/home/michael/Scripts/mount_vault.sh'
#PS1='[\u@\h \W]\$ '

PS1="$(tput setaf 2)\u$(tput setaf 6)\@$(tput setaf 1)@$(tput setaf 9)\h:$(tput setaf 3)\W$(tput sgr0)\n\$"
```

PS1:
```
michael12:25 PM@michaels-arch:~
$
```

For the PS1 and coloring, I'll compare the one on my Mac OS and make it similar, but I want to play with it.

Now I want to change it: 
PS1:
```
michael12:25 PM@michaels-arch:~
#
```
source ~/.bashrc

# Slow WiFi
Every time I boot up the computer it puts me on a wifi network I dont want to be on and when I try to change it forces itself back.

nmcli connection show --active
```
NAME          UUID                                  TYPE      DEVICE 

Frontier0394  3a9e2883-184b-422b-9da6-2063f837db04  wifi      wlp3s0 
[[lo]]            65eaf6e9-c4ac-4244-b9a3-1174e7b92e77  loopback  lo
```

nmcli device wifi list

lists wifi networks nearby:
```
IN-USE  BSSID              SSID                        MODE   CHAN  RATE         SIGNAL  BARS  SECURITY  

 
        F4:CF:A2:BD:D3:AD  PNR08IF3100E24E09B          Infra  1     135 Mbit/s   72      


```

Not over SSH.
```
nmcli device wifi connect "Your-Correct-SSID" password "yourpassword"
or
nmcli device wifi connect "Your-Correct-SSID" --type
```

I ended up forgetting the Frontier network, so there was one left.


## What worked
I went into the GUI and clicked forget the network 'Frontier0132'; this way, it would have no choice other than to connect to the other one.


# SSH 

in "SSH problem with starting the sshd." I setup a user nammed sshd with no password and it will be the low privlaged account I'll make. I checked etc/shadow to see if it was missing a '*'.
![[Screenshot 2025-12-08 at 1.22.37 AM.png]]


Ok, I set a password to a real one I found with a targeted password attack.


userdel sshd
sudo passwd sshd


sudo useradd -m -s /bin/bash sshd
sudo usermod -L sshd

in /etc/ssh/sshd_config

Match User sshd 
	PasswordAuthentication yes
	PubkeyAuthentication yes
	PermitUserRC no


![[Screenshot 2025-12-08 at 2.42.14 AM.png]]![[Screenshot 2025-12-08 at 2.43.03 AM.png]]

Ok, I didn't reboot. I did and I forgot to add nologin and no home to the user account


```
cat /etc/passwd
sshd:x:1001:1001::/home/sshd:/bin/bash



sudo usermod -s /sbin/nologin sshd
cat /etc/passwd
sshd:x:1001:1001::/home/sshd:/usr/sbin/nologin
```

I did ```sudo usermod -s /sbin/nologin sshd``` but I got the directory wrong so I changed it to: ```sudo usermod -s /usr/sbin/nologin sshd```. 

1. ```
    sudo rm -rf /home/sshd
    ```


To hide the sshd user account from the login screen, I needed to check my display manager.
systemctl status display-manager

It showed: sddm

From here I made a new folder and file.

mkdir /etc/sddm.conf
nvim /etc/sddm.conf/hide_sshd.conf

```
[Users]
HideUser=sshd
```

Works, and I can still ssh into that user.

Now I want to test it all, first I'll reboot my Alienware laptop to check the sshd user (only one user, michael . GOOD! ). Now I'll try to SSH as michael and enter the password to the account, "Permission denied, please try again." GOOD! Ok good. 

Now I need to put set up the .ssh folder in the new sshd home folder location (/var/empty/sshd) because I just deleted it and the /var/empty/sshd/. folder doesn't have my public key.

$sudo mkdir -p /var/empty/sshd/.ssh
$sudo touch /var/empty/sshd/.ssh/authorized_keys
$sudo chown -R sshd:sshd /var/empty/sshd/.ssh
$sudo chmod 700 /var/empty/sshd/.ssh
$sudo chmod 600 /var/empty/sshd/.ssh/authorized_keys


$sudo nvim /etc/ssh/sshd_config
```
Match User sshd 

	PasswordAuthentication yes 
	# optional but recommended 
	PubkeyAuthentication yes 
	AuthorizedKeysFile /var/empty/sshd/.ssh/authorized_keys 
	PermitUserRC no
```


I then needed to set up key login: ssh-copy-id sshd@192.168.87.30

This worked. Now when I log in to ssh as the low-privileged user sshd using the command: sshd@192.168.87.30, I'm then put into the directory: /var/empty/sshd and need to su michael to gain full privileges.

Then I'll add deny user michael so you need to ssh to the sshd user first.

sudo nvim /etc/ssh/sshd_config
```
DenyUser michael
```

To start hardening SSH i'll edit the /etc/ssh/sshd_config

I'll uncomment:
```
UsePAM no
PermitRootLogin no
PasswordAuthentication no for user sshd
```


```
nvim .ssh/config

Host arch
    HostName 192.168.254.40
    User sshd
```

### Result, login command:
```ssh arch```
![[Screenshot 2025-12-08 at 12.26.32 AM.png]]

# Encrypted Partition

The initial partition table looks like this:
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 931.5G  0 disk 
└─sda1   8:1    0   400G  0 part /mnt/steam_games
sdb      8:16   0 119.2G  0 disk 
├─sdb1   8:17   0     1G  0 part /boot
├─sdb2   8:18   0    16G  0 part [SWAP]
├─sdb3   8:19   0    32G  0 part /
└─sdb4   8:20   0  70.2G  0 part /home
```
The first thing I'll do is partition the rest of sda to sda2; the goal is to make an encrypted partition (vault) that is ext4.

First I'll make a partition with the rest of the data:
```
sudo fdisk /sda/sda2
n p enter enter w
```

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 931.5G  0 disk 
├─sda1   8:1    0   400G  0 part /mnt/steam_games
└─sda2   8:2    0 531.5G  0 part
```

now I setup the luks partition with the original Mac password:
```
sudo cryptsetup luksFormat /dev/sda2
```

Then I opend the partition on set the decrypted parition to ext4. Next I made a folder in /mnt/vault then mounted the decrypted partition to /mnt/vault 
```
sudo cryptsetup open /dev/sda2 encrypted_data
sudo mkfs.ext4 /dev/mapper/encrypted_data
sudo mkdir /mnt/vault
sudo mount /dev/mapper/encrypted_data /mnt/vault
sudo chown $USER /mnt/vault
```

```
NAME               MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                  8:0    0 931.5G  0 disk  
├─sda1               8:1    0   400G  0 part  /mnt/steam_games
└─sda2               8:2    0 531.5G  0 part  
  └─encrypted_data 253:0    0 531.5G  0 crypt /mnt/vault
```


To close it:
```
sudo umount /mnt/vault
sudo cryptsetup close encrypted_data
```

I want to intentinally not have it auto mount so the logged in user needs to enter in the password after just on case I don't need it open.

### mount and dismount script

 Script location: /home/michael/Scripts/mount_vault.sh

```
#!/bin/bash
#Check if the README.md in the /mnt/vault directory 

if [ ! -f /mnt/vault/README.md ]; then
	#File not found
	echo "Decrypting Data..."
	sudo cryptsetup open /dev/sda2 encrypted_data
	sudo mount /dev/mapper/encrypted_data /mnt/vault

else
	sudo umount /mnt/vault
	sudo cryptsetup close encrypted_data

fi
```
```chmod +x mount_vault.sh```


I set up an alias in the .bashrc file
```
alias vault='/home/michael/Scripts/mount_vault.sh'
source ~/.bashrc
```


## PoC

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

sda      8:0    0 931.5G  0 disk 
├─sda1   8:1    0   400G  0 part /mnt/steam_games
└─sda2   8:2    0 531.5G  0 part 

sdb      8:16   0 119.2G  0 disk 
├─sdb1   8:17   0     1G  0 part /boot
├─sdb2   8:18   0    16G  0 part [SWAP]
├─sdb3   8:19   0    32G  0 part /
└─sdb4   8:20   0  70.2G  0 part /home
```
![[Screenshot 2025-12-14 at 6.04.59 PM.png]]

Command "vualt" asks for a password to the partition.
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

sda      8:0    0 931.5G  0 disk 
├─sda1   8:1    0   400G  0 part /mnt/steam_games
└─sda2   8:2    0 531.5G  0 part 
  └─encrypted_data 253:0    0 531.5G  0 crypt /mnt/vault

sdb      8:16   0 119.2G  0 disk 
├─sdb1   8:17   0     1G  0 part /boot
├─sdb2   8:18   0    16G  0 part [SWAP]
├─sdb3   8:19   0    32G  0 part /
└─sdb4   8:20   0  70.2G  0 part /home
```


Here is the command in its entirty, I ssh to the box then elevat privileges, next run the aleas "vault" to decrypt/encrypt the partition on the SSD. 
```
michaelprovenzano|5:54PM@michaels-mbp:~
>ssh arch
[sshd@michaels-arch ~]$ su michael
Password: 
michael06:08 PM@michaels-arch:sshd
$cd ~
michael06:08 PM@michaels-arch:~
$vault
Decrypting Data...
[sudo] password for root: 
Enter passphrase for /dev/sda2: 
michael06:09 PM@michaels-arch:~
$lsblk

NAME               MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                  8:0    0 931.5G  0 disk  
├─sda1               8:1    0   400G  0 part  /mnt/steam_games
└─sda2               8:2    0 531.5G  0 part  
  └─encrypted_data 253:0    0 531.5G  0 crypt /mnt/vault
sdb                  8:16   0 119.2G  0 disk  
├─sdb1               8:17   0     1G  0 part  /boot
├─sdb2               8:18   0    16G  0 part  [SWAP]
├─sdb3               8:19   0    32G  0 part  /
└─sdb4               8:20   0  70.2G  0 part  /home
```
![[Screenshot 2025-12-14 at 6.02.21 PM.png]]


Works GREAT!!!!!!

# timeshift
I went on the GUI and took a screenshot on the system, its now on /mnt/steam_games partition
# rsync
To get this working, I needed to edit /etc/ssh/sshd_config and temporarily disable the restriction on the user michael. I decided to use rsync because I know SCP and wanted to learn a new tool. Next, I ran this command:
```
michaelprovenzano10:59PM@michaels-mbp:/Volumes/T7/H
```

# VPN 
Securing internet traffic with Arch and a VPN.  
  
I have a VPN on my phone and MacBook, but now I want to integrate my computer running Arch Linux. This was not as easy as one command; I needed to update, reinstall, and configure the application to work. There was a problem with corrupt software in libxml-legacy, then the linux-firmware, which I had to uninstall and reinstall, and clear the package cache. Then I needed a web browser to log in to the VPN, and because I don't intend on having a web browser, I decided the Tor browser was the best because I don't want to be tracked. I ended up logging in and using Inspect Element to see that it was trying to reach the Nordlynx protocol, which was exactly what the VPN CLI was looking for. This allowed me to type the callback URL exactly as intended using Tor. The official website suggested I install yay and AUR to download this, but turns out you only need GIT and pacman.

# TLP - Optimize Linux Laptop Battery Life[](https://linrunner.de/tlp/#tlp-optimize-linux-laptop-battery-life "Link to this heading")

install and enable TLP
```
sudo pacman -S tlp tlp-pd tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl start tlp
sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket

```

Adding optimizations: /etc/tlp.conf
```
CPU_SCALING_GOVERNOR_ON_BAT=schedutil # or ondemand — much snappier than powersave CPU_ENERGY_PERF_POLICY_ON_BAT=balance_performance CPU_BOOST_ON_BAT=1 # allow turbo on battery CPU_HWP_ON_BAT=balance_performance # if intel + HWP enabled

# Optional aggressive: PLATFORM_PROFILE_ON_BAT=balanced # instead of power-save / low-power
```

```tlp-stat -s``` I get this error:

```
**Error: After the next restart, you won't be able to switch power profiles on the desktop because tlp-pd.service is not enabled --> Invoke 'systemctl enable tlp-pd.service' to ensure the full functionality of TLP.**
```
To take advantage of power profiles, I'll enable this.
```
sudo systemctl enable tlp-pd.service
```
The error is gone:```tlp-stat -s```

# Librepods

So I want to install Docker on my Arch Linux computer first.
First, I updated my computer (of course, I read it all and look for confections)
```
sudo pacman -Syu
```
I intend on building docker containers and loading compose files so those packeges will be installed.
```
sudo pacman -S docker docker-compose docker-buildx
```

### I haven't done this, so you'll need root every time

**Allow non-root user access**: Add your user to the Docker` group to run Docker commands without `sudo`:
```
sudo usermod -aG docker ${USER}
```


Now I'll intsall librepods
```
git clone https://github.com/kavishdevar/librepods.git
```

Now I'll make a Dockerfile
```
cd librepods/linux
```
docker-compose.yaml
```
services: librepods: build: . image: librepods-test container_name: librepods-run network_mode: host restart: unless-stopped # optional: keeps it running if it crashes

devices: - /dev/snd - /dev/dri - /dev/bus/usb # for Bluetooth

group_add: - ${BLUETOOTH_GID:-118} # adjust if your bluetooth gid is different (check with `getent group bluetooth`) - ${AUDIO_GID:-998} # adjust if needed (check with `getent group audio`)

environment: - DISPLAY=${DISPLAY} - WAYLAND_DISPLAY=wayland-0 - XDG_RUNTIME_DIR=/run/user/1000 - QT_QPA_PLATFORM=wayland - QT_DEBUG_PLUGINS=1 # prints Qt plugin/tray loading info – very useful! - DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus # Optional debug/force fixes – uncomment one at a time to test # - XDG_CURRENT_DESKTOP= # sometimes KDE/Qt thinks SNI is unavailable # - KDE_FULL_SESSION= # - QT_QPA_PLATFORM=xcb # fallback to X11 backend (requires X11 mount below)

volumes: - /tmp/.X11-unix:/tmp/.X11-unix # X11 fallback (safe to keep) - /run/user/1000/bus:/run/user/1000/bus # dbus session – critical for tray on many DEs - /run/user/1000/pulse:/run/user/1000/pulse - /run/user/1000/wayland-0:/run/user/1000/wayland-0

cap_add: - NET_ADMIN # sometimes needed for Bluetooth tweaks

# Optional: run interactively for easier debugging # stdin_open: true # tty: true
```

Dockerfile
```
FROM ubuntu:24.04

RUN apt-get update -qq && \ apt-get install -y --no-install-recommends \ ca-certificates build-essential cmake git \ libssl-dev libpulse-dev libbluetooth-dev dbus \ qt6-base-dev qt6-connectivity-dev qt6-multimedia-dev \ qt6-declarative-dev qt6-svg-dev libqt6multimediaquick6 \ && rm -rf /var/lib/apt/lists/*

WORKDIR /app COPY . .

RUN mkdir build && cd build \ && cmake .. \ && make -j$(nproc)

CMD ["./build/librepods"]
```

# what worked
I followed the instructions on the Github page and it worked first try.


# fixing boot errors 


First red error says: 
```
journalctl -k -r | grep -i nvidia
```

```
probe with NVIDIA failed with error -1
```

people on the wiki say to add the nvidia-utils and nvidia-dkms package so I'll do research on the tool first.
```
pacman -Sy nvidia-utils nvidia-dkms
```


```
Jan 29 15:27:02 michaels-arch kernel: **NVRM: None of the NVIDIA devices were initialized.**

Jan 29 15:27:02 michaels-arch kernel: **NVRM: The NVIDIA probe routine failed for 1 device(s).**

Jan 29 15:27:02 michaels-arch kernel: **nvidia 0000:01:00.0: probe with driver nvidia failed with error -1**

Jan 29 15:27:02 michaels-arch kernel: **NVRM: The NVIDIA GPU 0000:01:00.0 (PCI ID: 10de:13d8)**

                                      **NVRM: installed in this system is not supported by open**

                                      **NVRM: nvidia.ko because it does not include the required GPU**

                                      **NVRM: System Processor (GSP).**

                                      **NVRM: Please see the 'Open Linux Kernel Modules' and 'GSP**

                                      **NVRM: Firmware' sections in the driver README, available on**

                                      **NVRM: the Linux graphics driver download page at**

                                      **NVRM: www.nvidia.com.**
```

I'm just going to install the newer nvidia driver. I'll get the model I have with another command since nvidia isn't working.
```
lspci | grep -i nvidia
```
GeForce GTX 960
Now I'll find the driver on the Nvidia website.![[Screenshot 2026-02-08 at 4.11.42 PM.png]]

I'll now copy the link to the terminal and download it.
```
mkdir driver; cd driver
curl https://us.download.nvidia.com/XFree86/Linux-x86_64/580.126.09/NVIDIA-Linux-x86_64-580.126.09.run -o NVIDIA-Linux-x86_64-580.126.09.run

chmod +x NVIDIA-Linux-x86_64-580.126.09.run
sudo ./NVIDIA-Linux-x86_64-580.126.09.run
```
Now nvidia works
![[Screenshot 2026-02-08 at 4.23.06 PM.png]]
# Fixed
# Next Error

This nex section is big but I highlighted the part that hase the error.
![[Screenshot 2026-02-08 at 4.43.08 PM.png]]
```lpc_ich: Resource conflict(s) found affecting gpio_ich```
To solve this I'll uninstall the part thats confilcting. First i'll blacklist it.
```
sudo nano /etc/modprobe.d/blacklist-ich.conf

blacklist gpio_ich
blacklist lpc_ich     # only add this if blacklisting gpio_ich alone doesn't silence everything
```

Regenerate initramfs
```
sudo mkinitcpio -P
sudo reboot
```

# Fixed
![[Screenshot 2026-02-08 at 5.17.39 PM.png]]

New error: ```ACPI Warning: \_SB.PCI0.PEG0.PEGP._DSM: Argument #4 type mismatch - Found [Buffer], ACPI requires [Package] (20250807/nsarguments-61)```

![[Screenshot 2026-02-08 at 5.24.47 PM.png]]
They say I need to make my own: ```/etc/default/grub```, because its not a arch linux default file.
Ok, the foled /etc/grub.d/ does exist on my laptop
```
sudo nvim /etc/default/grub

# /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="Arch"
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia_drm.modeset=1"
GRUB_CMDLINE_LINUX=""
```

Now I need the GRUB tools
```
sudo pacman -S grub
```

### Error
```
#ls /boot/grub/grub.cfg
ls: cannot access '/boot/grub/grub.cfg': No such file or directory
```
## solution
```
sudo mkdir -p /boot/grub
```

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```
# Done for now, no fix

# Next error
```at24 9-0050: supply vcc not found, using dummy regulator ```
![[Screenshot 2026-02-08 at 5.49.48 PM.png]]
For this I'll just blacklist it.
```
sudo nvim /etc/modprobe.d/at24.conf

options at24 ignore_dummy_regulator_warning=1
```


# New error
```
Feb 08 18:11:02 michaels-arch bootctl[687]:  **Mount point '/boot' which backs the random seed file is world accessible, which is a security hole!** 

Feb 08 18:11:02 michaels-arch bootctl[687]: **Random seed file '/boot/loader/random-seed' is world accessible, which is a security hole!**
```

![[Screenshot 2026-02-08 at 6.01.02 PM.png]]

sudo nvim /etc/fstab
```
UUID=4BE4-D0B4      /boot     vfat      rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro 0 2
```

```
fmask=0077,dmask=0077
```

remount the partition
```
sudo umount /boot
sudo mount -a
sudo systemctl daemon-reload

```

Now to check if it worked: ```sudo bootctl install```
# Fixed!!!!!
![[Screenshot 2026-02-08 at 6.17.49 PM.png]]


# New error to fix:
```gkr-pam: unable to locate daemon control file```![[Screenshot 2026-02-08 at 6.21.37 PM.png]]
- Edit /etc/pam.d/sddm (backup first: sudo cp /etc/pam.d/sddm /etc/pam.d/sddm.bak)
- Find lines like:
    
    text
    
    ```
    auth    optional        pam_gnome_keyring.so
    session optional        pam_gnome_keyring.so auto_start
    ```
    
- Add only_if=nonexistent or comment them out (e.g., # prefix).
## Fixed!!!!
![[Screenshot 2026-02-08 at 6.43.00 PM.png]]


# Back to error
```at24 9-0050: supply vcc not found, using dummy regulator ```
![[Screenshot 2026-02-08 at 5.49.48 PM.png]]
For this I'll just blacklist it.
```
sudo rm /etc/modprobe.d/at24.conf

sudo nvim /etc/modprobe.d/blacklist-at24.conf
blacklist at24

sudo reboot
```

# Fixed!!!!!!!
journalctl -b | grep at24. gives nothing
![[Screenshot 2026-02-08 at 6.45.27 PM.png]]


# Next error
```
faux_driver regulatory: Direct firmware load for regulatory.db failed with error -2
```
![[Screenshot 2026-02-08 at 6.51.51 PM.png]]

installing the missing package
```
sudo pacman -S wireless-regdb    
```

# Fixed
![[Screenshot 2026-02-08 at 6.55.53 PM.png]]

# Done fixing boot errors for now.



# I got GTA 5 Working !

Based on how far i've come with arch linux and the problem solving improvements and determination, I knew it was possible.

First I setup the rockstar launcher to run in steam with the arguments: 
```
gamemoderun %command%
```

then I logged in and, while running I startet GTA V. since two rockstar launchers were open and since I changed the password, the real one said "oh you changed your password, login agian".
I had to glitch it into a password reset since the log out wasn't sowing up and it was stuck on the old password. Then I set the time to be autmatice since I set it manually and it was off. probrably connecting to an NTP server. Then I deleted the profile folder in: /documents/rockstar games/Profile. This was in a different folder than the game. finally I went into steam and verified the game files. Then I was able to finally play the game in storymode. 3 years after Windows 8.1 said I couldn't. This was the main goal for installing arch linux, Windows 8.1 said it wasn't supported anymore. This was the only thing I did with the laptop, play GTA 5  and now it wasn't allowed on my hardware. Rockstar games forced me to learn arch linux and now I can run a game on hardware they didn't allow. Also I got my Apple Airpods working on the Aienware laptop. I also got the wifi fixed since it was so low on windows, kilabite speeds along with getting the audio working since it didn't work on windows. This is insain too me, I'm pointing the fingure at Microsoft, Rockstar, and Apple. Two companies who said I wasn't aloud to use my computer to its fullest potential since its and and one company with proprietary technology, I feel like I'm giving the fingure to three different billion dallar companies!


# Network interfaces failing to start up
I recently updated my computer with ```pacman -Syu```		then the wifi and ethernet stopped working. To trouble shoot this I first checked journalctl for anny problems. wpa_supplicant had an error 1 but it was due to no wpa_supplicant.conf file, not the problem.

Nexted I tried running these commands to get a better understanding of the overall network components. I forgot to mention, even in the GUI: NetworkManager, the wifi toggle was glitching off when I tried to turn it on that way.

```
ip a
systemctl status NetworkManager
systemctl status dhcpcd
```

dhcpcd wasn't active so I enabled it.
```
sudo systemctl enable dhcpcd
```
That got the ethenet working but the wifi still isn't. I decided to update the computer to see if it fixes anything.
```
sudo pacman -Syu
```
That did not work.

At this point, I might buy a better wireless NIC since mine is 10 years old and a new one is only 23 - 15 dollars. 

WIFI quick Solution
```

sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
sudo ip link set wlan0 up
```
