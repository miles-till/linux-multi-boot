# smiles Linux multi-boot config

This setup is for multi-booting between a main Linux distro, Win11, and a secondary Linux distro (more can easily be added). The Win install is there to allow running software/games that refuse to cooperate with Linux. The secondary Linux install is there to allow for testing other distros/kernels/risky configs/whatever.

## Choosing distro

| Distro     | Difficulty | Update cycle | Package manager |
| ---------- | ---------- | ------------ | --------------- |
| Kubuntu    | easy       | slow         | apt + snap      |
| Fedora KDE | easy       | medium       | dnf + COPR      |
| Arch + KDE | hard       | fast         | pacman + AUR    |

Kubuntu has all the familiarity of Ubuntu and apt, with the nice KDE Plasma DE, but snap sucks and slow update cycle
Arch has rolling updates that could cause problems at any time, and hard to set up
Fedora KDE seems like a happy medium with reasonable update cycle and package availability

tldr; Fedora KDE

## OS

### Disk partitions

My hardware includes 2 NVMe SSDs, one is 1 TB (sda) and the other is 2 TB (sdb). The intent is for the 1 TB drive to house the OS partitions and the 2 TB drive to house shared data. Windows will have a larger OS partition so it can have its own co-located data (probably just games that won't run on Linux).

```
Drive/partition  Description   Mount  Format  Size

sda                                             1 TB
├─ sda1          uefi/gpt             fat32     1 GB
├─ sda2          win msr                       16 MB
├─ sda3          win11         C:     ntfs    500 GB
├─ sda4          win recovery                 718 MB
├─ sda5          swap                 swap     32 GB
├─ sda6          fedora        /      ext4    200 GB
└─ sda7          other linux   /      ext4    200 GB

sdb                                             2 TB
└─ sdb1          data          /data  ext4      2 TB
```

### Install guide

1. Boot Windows install media
2. On sda create 512,000 Windows partition, let Win create sda1, sda2, sda3, sda4
3. Install Windows on sda3
4. Boot into Fedora install/live media
5. Open KDE Partition Manager
6. Move sda4, sda3, sda2 to the right by 924 MiB
7. Extend sda1 to 1,024 MiB (Win creates a 100 MB partition, we want 1 GB to be safe)
8. On sda create 32,768 MiB linuxswap partition (sda5)
9. On sda create 204,800 MiB ext4 partition (sda6)
10. On sda create 204,800 MiB ext4 partition (sda7)
11. On sdb create GPT partition table
12. On sdb create ext4 partition using all remaining space (sdb1)
13. Open Fedora installer app
14. For installation destination choose Custom
15. Set mount point sda1 = `/boot/efi`
16. Set mount point sda6 = `/`, check reformat
17. Create user
18. Install Fedora to sda6
19. Boot into Fedora
20. Open terminal
21. Run `blkid` to find the UUID for sdb1
22. Run `sudo nano /etc/fstab` and add a line `UUID={uuid}  /data  ext4  defaults,auto  0  2` where `{uuid}` is the UUID for `sdb1`
23. Run `lsblk` to check that partitionals are correctly configured
24. Run `swapon --show` to check that swap is correctly configured to use sda5
25. [Optional] Boot into other linux install/live media
26. [Optional] Install other linux distro to sda7

## Software

```sh
dnf install git
```

### Terminal - Ghostty

https://ghostty.org/docs/install/binary#linux-(official)

```sh
dnf copr enable pgdev/ghostty
dnf install ghostty
```

### Shell setup - zsh + ohmyzsh + p10k

#### Zsh

https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH

```sh
dnf install zsh
chsh -s /bin/zsh
```

#### Ohmyzsh

https://github.com/ohmyzsh/ohmyzsh?tab=readme-ov-file#basic-installation

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### P10K

https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#oh-my-zsh

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
nano ~/.zshrc
```

set `ZSH_THEME` to `powerlevel10k/powerlevel10k`
restart `zsh`

### Fonts

> [!NOTE]
> Don't need additional fonts for terminal if using `ghostty` as it comes with JetBrains Nerd Font

#### Meslo Nerd Font

https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/Meslo

preferred: MesloLGS NF

```sh
git clone --depth 1 https://github.com/ryanoasis/nerd-fonts.git ~/nerd-fonts

git clone --filter=blob:none --sparse https://github.com/ryanoasis/nerd-fonts.git ~/nerd-fonts
cd ~/nerd-fonts
git sparse-checkout add patched-fonts/Meslo
./install.sh Meslo
```

#### JetBrains Mono

https://github.com/JetBrains/JetBrainsMono

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/JetBrains/JetBrainsMono/master/install_manual.sh)"
```

### VSCode

https://code.visualstudio.com/docs/setup/linux#_rhel-fedora-and-centos-based-distributions

### Steam

> ![WARN]
> TODO

### Discord

> ![WARN]
> TODO

## Hardware

### CPU

> ![WARN]
> TODO

Intel i5-11400F

### GPU

> ![WARN]
> TODO

Nvidia GeForce GTX 1060 6GB

### Audio

> ![WARN]
> TODO

Vocaster Two mixer

### Input

> ![WARN]
> TODO

- Razer Deathadder mouse
- Logitech G815 keyboard
