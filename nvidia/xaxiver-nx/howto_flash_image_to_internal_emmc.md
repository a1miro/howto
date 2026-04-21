To flash a Yocto-built image to the internal eMMC of your Jetson Xavier NX, you must use the tegraflash artifacts generated during your Yocto build. This process requires putting the device into Force Recovery Mode (RCM) and using a Linux host PC. [1, 2, 3, 4] 
1. Locate the Flash Artifacts
After a successful bitbake (e.g., bitbake core-image-minimal), Yocto produces a compressed archive containing all necessary flashing scripts and binaries.

* Path: tmp/deploy/images/jetson-xavier-nx-devkit-emmc/
* File: Look for a file named <image-name>-jetson-xavier-nx-devkit-emmc.tegraflash.tar.gz. [5, 6] 

2. Prepare the Host PC
Extract the archive into a dedicated working directory on your Linux host: [7] 

mkdir jetson_flash
tar -xvf <image-name>-jetson-xavier-nx-devkit-emmc.tegraflash.tar.gz -C jetson_flash
cd jetson_flash

3. Put Device in Recovery Mode (RCM) [8] 

   1. Power off the Xavier NX.
   2. Short the Recovery Pins: On the developer kit carrier board (J14), short Pin 9 (FC REC) to Pin 10 (GND) using a jumper.
   3. Connect the Xavier NX to your host PC via the Micro-USB port.
   4. Power on the device.
   5. Verify: Run lsusb on your host. You should see an entry for Nvidia Corp..
   6. Remove the jumper once the device is detected. [5, 8, 9, 10, 11] 

4. Execute the Flash Script
Inside your extracted folder, run the provided helper script:

sudo ./doflash.sh

This script automates the call to tegraflash.py, partitioning the eMMC and writing the rootfs, kernel, and bootloader. [5, 12, 13, 14, 15] 
Troubleshooting

* Machine Configuration: Ensure your Yocto local.conf or machine variable is explicitly set to jetson-xavier-nx-devkit-emmc. If it is set to the standard devkit (which targets SD cards), the partition layout will not match the 16GB internal eMMC.
* Permissions: Always run the flash script with sudo, as it requires direct access to USB interfaces and loop devices for image mounting.
* Partition Size: If your Yocto image is too large, it may exceed the eMMC's 16GB limit. You can check or adjust the partition layout in the flash.xml file generated in the artifacts. [2, 12, 16, 17, 18] 

Would you like to know how to adjust the IMAGE_ROOTFS_SIZE in your Yocto configuration to ensure it fits on the 16GB eMMC?

[1] [https://github.com](https://github.com/orgs/OE4T/discussions/1235#:~:text=LudeeD%20on%20Apr%2018%2C%202023%20Hello%2C%20My,important%20parts%2C%20am%20I%20missing%20any%20steps?)
[2] [https://developer.ridgerun.com](https://developer.ridgerun.com/wiki/index.php/Yocto_Support_for_NVIDIA_Jetson_Platforms_-_Flashing_the_Jetson_Platform)
[3] [https://mediatek.gitlab.io](https://mediatek.gitlab.io/aiot/doc/aiot-dev-guide/master/sw/yocto/get-started/env-setup/flash-env-windows.html#:~:text=You%20must%20use%20a%20Linux%20host%20computer,the%20board%20and%20communicating%20with%20the%20board.)
[4] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/flashing-to-nvme-initially-worked-but-now-i-cannot-flash-again/352108#:~:text=Ensure%20that%20the%20NX%20is%20in%20forced%20recovery%20mode%20before%20attempting%20to%20flash.)
[5] [https://hub.mender.io](https://hub.mender.io/t/nvidia-tegra-jetson-xavier-nx/2615)
[6] [https://developer.ridgerun.com](https://developer.ridgerun.com/wiki/index.php/Yocto_Support_for_NVIDIA%C2%AE_Jetson%E2%84%A2_with_JetPack_6_Integration_-_Flashing_the_Jetson_Platform)
[7] [https://docs.ipi.wiki](https://docs.ipi.wiki/smarc/ipi-smarc-imx8mp/BootFromeMMC.html)
[8] [https://github.com](https://github.com/balena-os/balena-jetson/blob/master/jetson-xavier-nx-devkit-emmc.coffee#:~:text=deviceTypesCommon%20=%20require%20%27@resin.io/device%2Dtypes/common%27%20%7B%20networkOptions%2C%20commonImg%2C,partition:%20primary:%209%20path:%20%27/config.json%27%20initialization:%20commonImg.initialization.)
[9] [https://koansoftware.com](https://koansoftware.com/yocto-project-on-nvidia-jetson-agx-xavier/)
[10] [https://developer.ridgerun.com](https://developer.ridgerun.com/wiki/index.php/Yocto_Support_for_NVIDIA%C2%AE_Jetson%E2%84%A2_with_JetPack_6_Integration_-_Flashing_the_Jetson_Platform)
[11] [https://aiot-ist.github.io](https://aiot-ist.github.io/eos-jnx/howtoflashimage/)
[12] [https://stackoverflow.com](https://stackoverflow.com/questions/58080403/flashing-custom-yocto-image-to-jetson-nano-production-module-emmc)
[13] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/boot-jetson-xavier-nx-from-emmc-encrypt-external-ssd-for-data-storage/342938#:~:text=Boot%20Jetson%20Xavier%20NX%20from%20eMMC%2C%20encrypt,is%20required%20for%20auto%20format/mount%20in%20initrd.)
[14] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/flashing-xavier-nx-emmc/187379)
[15] [https://forums.developer.nvidia.com](https://forums.developer.nvidia.com/t/xavier-nx-l4t-32-7-2-system-backup-and-recovery/331159)
[16] [https://github.com](https://github.com/OE4T/meta-tegra/issues/1439)
[17] [https://docs.nvidia.com](https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/FlashingSupport.html)
[18] [https://support.criticallink.com](https://support.criticallink.com/redmine/projects/armc8-platforms/wiki/Booting_Win_EC_2013_from_SD_Card#:~:text=Booting%20Win%20EC%202013%20from%20SD%20Card,11%5D:%20000100000000%20which%20is%20our%20standard%20configuration.)

