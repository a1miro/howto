## 1. Download Yocto images
Download from [here](http://home.paralleldynamic.com:8022/images/imx93evk/) the following two files:
- imx-boot-imx93evk-sd.bin-flash_singleboot
- rolec-everest-image-qt6-dev-imx93evk.rootfs.wic.zst

## 2. Install IMX UUU tool 
Download the UUU executable from [here](https://github.com/nxp-imx/mfgtools/releases). You can get more information about mfgtools from [github](https://github.com/nxp-imx/mfgtools).

## 3. Set IMX93EVK board to Serial Download Protocol (SDP) mode

Change dip switch SW1301 to **1100**

## 4. Connect

Connect USB-C cable to the USB1 of the IMX93EVK board and then to your PC.

## 5. Verify the USB connection to the board
Power on the board and run the following command to verify that the board is detected by your PC.
```bash
uuu -lsusb
```
the output should be similar to this:

```bash
uuu (Universal Update Utility) for nxp imx chips -- libuuu_1.5.233-1-g2084007
Connected Known USB Devices
	Path	 Chip	 Pro	 Vid	 Pid	 BcdVersion	 Serial_no
	====================================================================
	20:11	 MX93	 SDPS:	 0x1FC9	0x014E	 0x0001	 7B7C5181B49A41D
```

## 6.Flash the image
You need to be in a folder where you have downloaded the two files in step 1. Run the following command to flash the image to internal eMMC:

```bash
sudo uuu -b emmc_all imx-boot-imx93evk-sd.bin-flash_singleboot rolec-everest-image-qt6-dev-imx93evk.rootfs.wic.zst
```
you should see similar output:

```bash
uuu (Universal Update Utility) for nxp imx chips -- libuuu_1.5.233-1-g2084007

Success 1    Failure 0

20:11-7B7C51 8/ 8 [Done                                  ] FB: done
```
 
## 7.  Set the board to boot from internal eMMC

Set dip switch SW1301 to all zeros **0000**

## 8. Reconnect usb-c serial cable to Debug port and reboot the board
Power off the board and then power it on again. The board should boot from internal eMMC and you should see boot sequence and bash prompt at the end of the booting process . 