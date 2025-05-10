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
21. Run `sudo blkid` to find the UUID for sdb1
22. Run `sudo nano /etc/fstab` and add a line `UUID={uuid}  /data  ext4  defaults,auto  0  2` where `{uuid}` is the UUID for `sdb1`
23. Run `lsblk` to check that partitionals are correctly configured
24. Run `swapon --show` to check that swap is correctly configured to use sda5
25. [Optional] Boot into other linux install/live media
26. [Optional] Install other linux distro to sda7

## Software

```sh
sudo dnf install git
```

### Terminal - Ghostty

https://ghostty.org/docs/install/binary#linux-(official)

```sh
sudo dnf copr enable pgdev/ghostty
sudo dnf install ghostty
```

### Shell setup - zsh + ohmyzsh + p10k

#### Zsh

https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH

```sh
sudo dnf install zsh
sudo chsh -s /bin/zsh
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

Preferred font for terminals: MesloLGS NF

```sh
git clone --depth 1 https://github.com/ryanoasis/nerd-fonts.git ~/nerd-fonts

git clone --filter=blob:none --sparse https://github.com/ryanoasis/nerd-fonts.git ~/nerd-fonts
cd ~/nerd-fonts
git sparse-checkout add patched-fonts/Meslo
./install.sh Meslo
```

#### JetBrains Mono

https://github.com/JetBrains/JetBrainsMono

Preferred font for code editors: JetBrains Mono

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/JetBrains/JetBrainsMono/master/install_manual.sh)"
```

### VSCode

https://code.visualstudio.com/docs/setup/linux#_rhel-fedora-and-centos-based-distributions

### Steam

https://docs.fedoraproject.org/en-US/gaming/proton/#_using_the_terminal

```sh
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm -y
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
sudo dnf install steam -y
```

If first launch fails, try launching from terminal with this env var set (https://www.reddit.com/r/Fedora/comments/1k0f36m/steam_launch_problem/)

```sh
__GL_CONSTANT_FRAME_RATE_HINT=3 steam
```

Be sure to enable Proton - https://docs.fedoraproject.org/en-US/gaming/proton/#_enable_proton_engine

### Discord

```sh
sudo dnf install discord
```

## Hardware

### CPU

Intel i5-11400F

### GPU

Nvidia RTX 5070 12GB

https://rpmfusion.org/Howto/NVIDIA#InstallingTheDrivers

```sh
sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install akmod-nvidia
sudo dnf install xorg-x11-drv-nvidia-cuda
```

Wait until the following command returns a driver version number before continuing.

```sh
modinfo -F version nvidia
```

https://rpmfusion.org/Howto/NVIDIA#KernelOpen

RTX 5070 requires new NVIDIA open source driver.

```sh
sudo sh -c 'echo "%_with_kmod_nvidia_open 1" > /etc/rpm/macros.nvidia-kmod'
sudo akmods --kernels $(uname -r) --rebuild
reboot
```

### Audio

> [!WARN]
> TODO

Vocaster Two mixer
