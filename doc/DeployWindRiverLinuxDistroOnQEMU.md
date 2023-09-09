# Deploy Wind River Linux Distro on QEMU

## Index
- [Environment](#environment)
- [Install QEMU](#install-qemu)
- [Download Files](#download-files)
- [Deploy Wind River Linux Distro](#deploy-wind-river-linux-distro)
- [Link](#link)

## Environment
```
+------------------------------------------------------------+
| Ubuntu Server 22.04.3 LTS (x86_64)                         |
|                                                            |
| +--------------------------------------------------------+ |
| | QEMU 6.2.0                                             | |
| |                                                        | |
| | +----------------------------------------------------+ | |
| | | Wind River Linux Graphics LTS 10.22.33.11 (aarch64)| | |
| | +----------------------------------------------------+ | |
| +--------------------------------------------------------+ |
+------------------------------------------------------------+
```

## Install QEMU
1. Refer to [this page](https://github.com/EXPRESSCLUSTER/QEMU/blob/master/doc/RunARMonUbuntu2204.md#install-packages).

## Download Files
1. Submit form to get the download link.
   - https://www.windriver.com/japan/products/linux/download
1. Download the following files.
   - sdk-bcm-2xxx-rpi4.tar.bz2
   - target-minimal-bcm-2xxx-rpi4.tar.bz2

## Deploy Wind River Linux Distro
1. Copy the above files save the home directory of your account on Ubuntu Server.
   - E.g., /home/clpuser
     ```
     $ pwd 
     /home/clpuser
     $ ls -l -h
     -rw-rw-r-- 1 clpuser clpuser 1.4G Aug 30 23:02 sdk-bcm-2xxx-rpi4.tar.bz2
     -rw-rw-r-- 1 clpuser clpuser 173M Aug 30 22:46 target-minimal-bcm-2xxx-rpi4.tar.bz2
     ```
1. Expand sdk-bcm-2xxx-rpi4.tar.bz2.
   ```sh
   tar jxvf sdk-bcm-2xxx-rpi4.tar.bz2
   ```
1. Move to the SDK directory.
   ```cd 
   cd sdk-bcm-2xxx-rpi4
   ```
1. Install SDK.
   ```sh
   ./wrlinux-graphics-10.22.33.11-glibc-x86_64-bcm_2xxx_rpi4-wrlinux-image-full-sdk.sh
   ```
   - Enter target directory.
     - E.g., /home/clpuser/wrlinux-sdk
1. Expand target-minimal-bcm-2xxx-rpi4.tar.bz2.
   ```sh
   tar jxvf target-minimal-bcm-2xxx-rpi4.tar.bz2
   ```
1. Set environment.
   ```sh
   . /home/clpuser/wrlinux-sdk/environment-setup-cortexa72-wrs-linux
   ```
1. Create a directory to save qcow2 image file.
   ```sh
   mkdir -p /home/clpuser/qemu/aarch64/windriver
   ```
1. Move to the following directory.
   ```sh
   cd /home/clpuser/qemu/aarch64/windriver
   ```
1. Create a directory (e.g., path_to_vsd).
   ```sh
   mkdir path_to_vsd
   ```
1. Create a qcow2 file.
   ```sh
   qemu-img create -f raw path_to_vsd/img 8G
   ```   
1. Create a directory (e.g., path_to_img).
   ```sh
   mkdir path_to_img
   ```
1. Copy wrlinux-image-minimal-bcm-2xxx-rpi4.ustart.img.gz to path_to_img.
   ```sh
   cp /home/clpuser/target-minimal-bcm-2xxx-rpi4/wrlinux-image-minimal-bcm-2xxx-rpi4.ustart.img.gz /home/clpuser/qemu/aarch64/windriver/path_to_img
   ```
1. Write the whole data to img file.
   ```sh
   zcat path_to_img/wrlinux-image-minimal-bcm-2xxx-rpi4.ustart.img.gz.gz | dd of=path_to_vsd/img conv=notrunc 
   ```
1. Copy qemu-u-boot-bcm-2xxx-rpi4.bin to path_to_img.
   ```sh
   cp /home/clpuser/target-minimal-bcm-2xxx-rpi4/qemu-u-boot-bcm-2xxx-rpi4.bin /home/clpuser/qemu/aarch64/windriver/path_to_img
   ```
1. Run the following command.
   ```sh
   qemu-system-aarch64 \
   -machine virt \
   -cpu cortex-a57 \
   -device virtio-net-device,netdev=net0 \
   -netdev user,id=net0 \
   -m 512 \
   -bios path_to_img/qemu-u-boot-bcm-2xxx-rpi4.bin \
   -device virtio-gpu-pci \
   -device qemu-xhci \
   -device usb-tablet \
   -device usb-kbd \
   -drive id=disk0,file=path_to_vsd/img,if=none,format=raw \
   -device virtio-blk-device,drive=disk0 \
   -nographic
   ```
1. Enjoy!
   ```
   root@bcm-2xxx-rpi4:~# cat /etc/os-release
   ID=wrlinux-graphics
   NAME="Wind River Linux Graphics LTS"
   VERSION="10.22.33.11"
   VERSION_ID=10.22.33.11
   PRETTY_NAME="Wind River Linux Graphics LTS 22.33 Update 11"
   root@bcm-2xxx-rpi4:~# uname -r
   5.15.120-yocto-standard
   root@bcm-2xxx-rpi4:~# arch
   aarch64
   ```
## Link
- How to Deploy Wind River Linux Distro on QEMU (Japanese)
  - https://www.windriver.com/japan/products/linux/tutorial/start-with-distro