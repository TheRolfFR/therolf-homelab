# TheRolf's Homelab

*Why a git? So I can version my config files in case I mess something up.*

## Intro

I decided to self-host when I ran out of space on my Google Photos, due to recent changes on space provided now using Google Drive total storage space.

It was my excuse (yours can differ), but it was the right thing to OWN my data. Like everything. I had tech skills, and could share storage and system with my friends and family.

According to my NAS, my system was installed on July 3rd, 2023, at 10:41PM. This documentation first line is October 26th, 2024, at 1:03PM. So it took me
other a year to convince myself to document this. 

```sh
# How to show system age
stat / | awstat / | awk '/Birth: /{print $2 " " substr($3,1,5)}'
```

You'd say that this repo was made for my friends to share when in fact it just for me in case I mess something up bad and I can't remember how to do it.

## The setup

Setup started very simple, with a Raspberry Pi 4 4GB as my traffic router / DNS server routing data with a reverse-proxy and my NAS for my services.

As my services grew, I realised my Pi could be a bottleneck point (it does not but why not) so I retired him and transferred all my services to my NAS.

## The NAS

I made a DIY NAS with almost second-hand hardware bought online:
- An old case but with 8 3.5" hard-drive trays with noise absorbing materiel and space and had some BeQuiet fans preinstalled
- A basic APU so that I can still have a screen if needed
- A descent SSD with jsut enough storage for system
- Running on Ubuntu Server 22.04 to get help and just use apt
- 10G SFP+ NIC to be like: *I am speed*

Now the specs:
- CPU: Ryzen 5 3400G
- RAM: 8GB + 2*16GB DDR4
- MB: Asus PRIME B350M-A
- SSD: 1 To Samsung 970 Evo Plus
- HDD: 4x4To Seagate IronWolf
- NIC: Asus XG-C100F 10Gbit SFP+
- PSU: BeQuiet PurePower 11 500W
- Case: Fractal Design Define R3

### Disk config

Disk config is in RAID1, not optimal but okay for now, and I still have 4 trays left lol:

```sh
cat /proc/mdstat
```

```
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10] 
md127 : active raid1 sdc1[0] sdb1[1]
      3906884608 blocks super 1.2 [2/2] [UU]
      bitmap: 0/30 pages [0KB], 65536KB chunk

md1 : active raid1 sda[0] sdd[1]
      3906886464 blocks super 1.2 [2/2] [UU]
      bitmap: 8/30 pages [32KB], 65536KB chunk
```

And disks are stored like this:

```sh
sudo lsblk
```

```
sda           8:0    0   3,6T  0 disk  
└─md1         9:1    0   3,6T  0 raid1 
  └─md1p1   259:3    0   3,6T  0 part  /mnt/b
sdb           8:16   0   3,6T  0 disk  
└─sdb1        8:17   0   3,6T  0 part  
  └─md127     9:127  0   3,6T  0 raid1 /mnt/a
sdc           8:32   0   3,6T  0 disk  
└─sdc1        8:33   0   3,6T  0 part  
  └─md127     9:127  0   3,6T  0 raid1 /mnt/a
sdd           8:48   0   3,6T  0 disk  
└─md1         9:1    0   3,6T  0 raid1 
  └─md1p1   259:3    0   3,6T  0 part  /mnt/b
```

Disk detailed scan is stored in :
```sh
sudo mdadm --detail --scan > nas-system/raid-scan.conf
```
And can be restored just with scan if disks are the same with previous command ([source](https://unix.stackexchange.com/a/458607)).

### The Packages

I have many packages installed that I did forgot to list.
So I follwed this tutorial ([link](https://www.cyberciti.biz/tips/linux-get-list-installed-software-reinstallation-restore.html)) to backup a list and restore it in [nas-system/installed-software.log](nas-system/installed-software.log):

Backup:
```sh
dpkg --get-selections > nas-system/installed-software.log
```

Restore:
```sh
dpkg --set-selections < nas-system/installed-software.log
apt-get dselect-upgrade
```

### The VMs

I enabled Virtualization in the BIOS so I could do some nice Kernel-based virtual machines (KVM)

## Services strategy: Independant and auto-update

Despite having to update system packages automatically, I wanted my services to be independant and automatically update because I am a lazy person.

To do so I have decided to use Docker. I could pull my services and bind my folders to the data required. It is easier to move services, maintain, store and organize data and services between them. [Docker Compose](https://docs.docker.com/compose/) has been one of the best discoveries in my life to describe and run multi-container applications.

Auto updates for containers are made with a container-based solution called [Watchtower](https://containrrr.dev/watchtower/):

> With watchtower you can update the running version of your containerized app simply by pushing a new image to the Docker Hub or your own image registry. Watchtower will pull down your new image, gracefully shut down your existing container and restart it with the same options that were used when it was deployed initially.


