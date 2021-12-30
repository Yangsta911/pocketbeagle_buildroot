# pocketbeagle_buildroot
A guide on how to set up a simple linux kernel for a pocketbeagle board using buildroot.

## Steps
1. Clone the buildroot source code from its git repository ``` git clone https://git.buildroot.net/buildroot ``` 
2. Run the command ```make list-defconfigs``` to find the corresponding board that you are trying to build.
3. To edit the configurations that are given run ```make menuconfig```, a list of settings will appear. Changes these to fit your needs.
4. Go to bootloader section and change the custom U-boot version to 2020.04.
5. Make the kernel image using ```make```
6. Set up the SD card for flashing
  - Unmount all of the partitions of the SD card
  - Erase the SD card ```sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=16```. Replace mmcblk0 with where the SD card is mounted
  - Start the cfdisk tool ```sudo cfdisk /dev/mmcblk0``` to create two partitions
  - select dos partition table type
  - Create a first small partition (128 MB), primary, with type e (W95 FAT16) and mark it bootable
  - Create a second partition, also primary, with the rest of the available space, with type 83 (Linux).
  - Format the first partition as a FAT32 filesystem: ```sudo mkfs.vfat -F 32 -n boot /dev/mmcblk0p1``` Replace mmcblk0p1 with where the SD card is mounted
  - ormat the second partition as an ext4 filesystem: ```sudo mkfs.ext4 -L rootfs -E nodiscard /dev/mmcblk0p2```Replace mmcblk0p2 with where the SD card is mounted
7. Flash the system by copying the MLO, u-boot.img, zImage and am335x-boneblack.dtb from output/images/ to the boot partition of the SD card.
8. Extract the rootfs.tar file to the rootfs partition of the SD card, using: ```sudo tar -C /media/$USER/rootfs/ -xf output/images/rootfs.tar```
9. Connect the board by serial port and download picocom ```sudo apt install picocom```.
10. Give the user access to read the usb port by ```sudo adduser $USER dialout```.
11. Run ```picocom -b 115200 /dev/ttyUSB0``` to start serial communications.
12. Insert the SD card into the board and plug in the micro-usb cabel.
13. Interupt the auto-boot to enter the u-boot menu.
14. Run the following commands to boot the system:
```
fatload mmc 0:1 0x82000000 zImage 
fatload mmc 0:1 0x88000000 am335x-boneblack.dtb 
setenv bootargs console=ttyS0,115200n8 root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait

bootz 0x82000000 - 0x88000000
```
