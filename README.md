# BootFlash

## Linux images with CUDA and Docker for cloning (fast compute cluster provision)

The described steps below create a [DGX-style](dgxa100-user-guide.pdf) environment on bare metal PC & NVIDIA GPU hardware:

Step1: To clone the below Linux images, first create a bootable USB FlashStick with [Clonezilla](https://clonezilla.org) disk imaging and cloning software. For long-term compatibility, here is a link to the copy of Clonezilla [ISO image](http://bootflash.pinitree.com:7789/uploads/clonezilla-live-3.0.1-8-amd64.iso) (380MB) version actually used for cloning Ubuntu images below. There are [many ways](https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview) to create a bootable USB FlashStick from ISO image. 

Step 2: Unzip the appropriate Linux image and copy the image folder on the separate USB FlashStick (standard FAT32 formatting is OK, although EXT4 is confirmed to work on DGX A100 server). Alternatively, create and use a separate partition on the same Clonezilla FlashStick created in the Step 1. 

Step 3: Then boot a computer from the Clonezilla USB FlashStick and restore IMAGE TO DISK. The target DISK should be at least 128GB of size - it could be the internal HDD or SSD in the computer or another USB FlashStick, SD Card, external drive. 

Step 4: Finally, resize the restored Ubuntu partition to the actual size of the target DISK. This can be done from within the restored Ubuntu with the DiskUtility (does not require unmounting or reboot). GUI access with H2020 consortium name, twice, lower-case.

## Ubuntu Linux images

http://bootflash.pinitree.com:7789/uploads/2022-08-21-14-img.zip (11.7GB) Ubuntu 20.04 LTS + CUDA 11.4 + Docker + Kubernetes

http://bootflash.pinitree.com:7789/uploads/2022-08-21-15-img.zip (12.8GB) Ubuntu 20.04 LTS + CUDA 11.7 + Docker

http://bootflash.pinitree.com:7789/uploads/2022-08-22-21nerf-img.zip (17.7GB) Ubuntu 20.04 LTS + CUDA 11.7 + Docker + InstantNeRF

http://bootflash.pinitree.com:7789/uploads/2023-07-05-05-img.zip (15.2GB) Ubuntu 22.04 LTS + CUDA 12.2 + Docker (restorable only to 3.8TB or larger disk; includes NVIDIA DGX A100 fabricmanager driver; autoupgrades disabled)


## Bugs

Sometimes the Ethernet interface gets stuck at 100Mbps speed, despite hardware supporting 1Gbps.
Not confirmed if the problem was real, but confirmed this does not harm and 1Gbps works afterwards ([source](https://askubuntu.com/questions/1398448/ethernet-internet-speed-stuck-at-100-mbps-for-gigabit-connection-ubuntu-20-04)):

sudo nano /etc/default/grub 

  Edit the following line and add pcie_aspm=off
  
GRUB_CMDLINE_LINUX_DEFAULT="splash pcie_aspm=off"

sudo update-grub

====

CUDA version `realpath /usr/local/cuda` shows CUDA version 10.1. The CUDA Toolkit files are usually embedded into the Docker Container itself, however, the NVIDIA kernel drivers are not, they are shared from the host system (enabled via specific Docker run command line arguments), so they must be up to date for the CUDA Toolkit version embedded into the container. This is a problem also for Docker containers requiring newer CUDA versions.

```scp guntis@194.8.1.225:/home/guntis/cuda_12.0.0_525.60.13_linux.run .```

nvidia-smi shows version 11.4. `sudo init 3` - stops X-server, before upgrading CUDA drivers. `sudo sh cuda_12.0.0_525.60.13_linux.run` (if returns an error about kernel module removal, execute first the following: `apt-get remove --purge nvidia-*`
Afterwards execute the following: `sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit` - for NVIDIA driver support into Docker containers).
To return the system to original runlevel: `sudo reboot`

To list installed NVIDIA drivers via DKMS (Dynamic Kernel Module Support) system, use `dkms status`. To see the current running Linux kernel version, use `uname -a` or `uname -r`.

See `dkms --help` on how to remove specific driver version, NOTE that Linux kernel version (using `-k` option) and module (i.e., NVIDIA kernel driver) version (using `-v` option) must be specified, e.g., `dkms remove -k Linux-kernel-version -v NVIDIA-driver-version`

To verify currently installed NVIDIA driver version, use `nvidia-smi`.

====

CUDA Toolkit by default is installed into `/usr/local/cuda*`, the default version is `/usr/local/cuda` (symlink to specific CUDA version, e.g., `/usr/local/cuda -> /usr/local/cuda-12.0`).
The second component is NVIDIA kernel driver (it must not be older than the required version by the CUDA Toolkit). Use `nvidia-smi` to see the NVIDIA kernel driver version.

It is advised to install it via dkms system (Dynamic Kernel Module Support), i.e., when installing CUDA Toolkit via (self-contained)
[shell script](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local)
from command line, use the `--dkms` option. It will then self-manage, i.e., recompile the driver module after Linux kernel updates. Use `dkms status` to list the installed modules.

========

For NVIDIA DGX A100 systems an additional driver needs to be installed via below commands.

Before installing below fabricmanager driver disable automatic updates (because Ubuntu automatically upgrades fabricmanager, but not CUDA drivers - leading to version mismatch) by following these steps in GUI: Step 1: Open the “Software & Updates” app from the applications menu. Step 2: Click the “Updates” tab and select “Automatically check for updates:“. Here, you can choose between “Never”, “Every day”, “Every two days”, “Weekly” and “Monthly”. Select “Never“.

 `wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/nvidia-fabricmanager-515_515.65.01-1_amd64.deb`
 
 `sudo dpkg -i nvidia-fabricmanager-515_515.65.01-1_amd64.deb`
 
 `sudo systemctl enable nvidia-fabricmanager`

