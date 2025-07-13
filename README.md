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

My hardware includes 2 NVMe SSDs, one is 1 TB (nvme1n1) and the other is 2 TB (nvme0n1). The intent is for the 1 TB drive to house the OS partitions and the 2 TB drive to house shared data. Windows will have a larger OS partition so it can have its own co-located data (probably just games that won't run on Linux).

```
Drive/partition  Description   Mount  Format  Size

nvme1n1                                         1 TB
├─ nvme1n1p1     uefi/gpt             fat32     1 GB
├─ nvme1n1p2     win msr                       16 MB
├─ nvme1n1p3     win11         C:     ntfs    500 GB
├─ nvme1n1p4     win recovery                 718 MB
├─ nvme1n1p5     swap                 swap     32 GB
├─ nvme1n1p6     fedora        /      ext4    200 GB
└─ nvme1n1p7     other linux   /      ext4    200 GB

nvme0n1                                         2 TB
└─ nvme0n1p1     data          /data  ext4      2 TB
```

### Install guide

1. Boot Windows install media
2. On nvme1n1 create 512,000 Windows partition, let Win create nvme1n1p1, nvme1n1p2, nvme1n1p3, nvme1n1p4
3. Install Windows on nvme1n13
4. Boot into Fedora install/live media
5. Open KDE Partition Manager
6. Move nvme1n1p4, nvme1n1p3, nvme1n1p2 to the right by 924 MiB
7. Extend nvme1n1p1 to 1,024 MiB (Win creates a 100 MB partition, we want 1 GB to be safe)
8. On nvme1n1 create 32,768 MiB linuxswap partition (nvme1n1p5)
9. On nvme1n1 create 204,800 MiB ext4 partition (nvme1n1p6)
10. On nvme1n1 create 204,800 MiB ext4 partition (nvme1n1p7)
11. On nvme0n1 create GPT partition table
12. On nvme0n1 create ext4 partition using all remaining space (nvme0n1p1)
13. Open Fedora installer app
14. For installation destination choose Custom
15. Set mount point nvme1n1p1 = `/boot/efi`
16. Set mount point nvme1n1p6 = `/`, check reformat
17. Create user
18. Install Fedora to nvme1n1p6
19. Boot into Fedora
20. Open terminal
21. Run `sudo blkid` to find the UUID for nvme0n1p1
22. Run `sudo nano /etc/fstab` and add a line `UUID={uuid}  /data  ext4  defaults,auto  0  2` where `{uuid}` is the UUID for `nvme0n1p1`
23. Run `lsblk` to check that partitionals are correctly configured
24. Run `swapon --show` to check that swap is correctly configured to use nvme1n1p5
25. [Optional] Boot into other linux install/live media
26. [Optional] Install other linux distro to nvme1n1p7

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

## Software

```sh
sudo dnf install git
```

### Terminal - Ghostty

https://ghostty.org/docs/install/binary#linux-(official)

> NOTE: Check above link for official recommendation per distro first.

```sh
sudo dnf copr enable scottames/ghostty
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
