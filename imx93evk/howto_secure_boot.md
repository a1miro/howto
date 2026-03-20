# Secure boot on i.MX93 EVK

## Table of Contents

## Introduction

## Installing NXP Code Signing Tool

**1.** ***Download the CST tool****

from NXP website download [tar archive](https://www.nxp.com/webapp/Download?colCode=IMX_CST_TOOL_NEW&appType=license).

**2.** ***Untar the archive***

If you run Yocto build on a docker container, I recommend to install the CST under /opt/apps folder.

```bash
 tar xvf IMX_CST_TOOL_NEW.tar -C /opt/apps/
```

And add the SIG_TOOL_PATH in local.conf. 

```bash
SIG_TOOL_PATH="/opt/apps/cst-4.0.1"
SIG_DATA_PATH="/opt/apps/cst-4.0.1/keys"
```

**3.** ***(Optional) Create and edit serial and key_pass.txt files***
Go to CST tool folder
```bash
cd /opt/apps/cst-4.0.1
```
Create/edit two files: serial and key_pass.txt
serial – used by OpenSSL for certificate serial numbers
```bash
echo 42424242 > serial
```
key_pass.txt – custom passphrase that will protect the AHAB code signing
private keys
```bash
echo "your_password_here" > key_pass.txt
```

**4.** ***Generate keys****
Go to keys folder 
```bash
cd /opt/apps/cst-4.0.1/keys
```
generate PKI tree
```bash
./ahab_pki_tree.sh
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Do you want to use an existing CA key (y/n)?: n

Key type options (confirm targeted device supports desired key type):
Select the key type (possible values: rsa, rsa-pss, ecc)?: ecc
Enter length for elliptic curve to be used for PKI tree:
Possible values p256, p384, p521:  p256
Enter the digest algorithm to use: sha256
Enter PKI tree duration (years): 20
Do you want the SRK certificates to have the CA flag set? (y/n)?: n
```
you should see that these keys and certificates have been generated in the keys folder:
```bash
SRK1_sha256_secp256r1_v3_usr_key.der
SRK1_sha256_secp256r1_v3_usr_key.pem
SRK2_sha256_secp256r1_v3_usr_key.der
SRK2_sha256_secp256r1_v3_usr_key.pem
SRK3_sha256_secp256r1_v3_usr_key.der
SRK3_sha256_secp256r1_v3_usr_key.pem
SRK4_sha256_secp256r1_v3_usr_key.der
SRK4_sha256_secp256r1_v3_usr_key.pem
```

**5.** Generate fuse binaries
Using the public key certificates from the previous step, we can now
create
the SRK table (a table of the public SRKs)
the SRK fuse table to be programmed into the SOC fuses:
```bash
$ cd ../crts/
$ ../linux64/bin/srktool -a -d sha256 -s sha384 -t SRK_1_2_3_4_table.bin \
    -e SRK_1_2_3_4_fuse.bin -f 1 -c \
    ../keys/SRK1_sha256_secp256r1_v3_usr_key.pem \
    ../keys/SRK2_sha256_secp256r1_v3_usr_key.pem \
    ../keys/SRK3_sha256_secp256r1_v3_usr_key.pem \
    ../keys/SRK4_sha256_secp256r1_v3_usr_key.pem
```
you should see ***SRK_1_2_3_4_table.bin*** and ***SRK_1_2_3_4_fuse.bin*** generated in the crts folder.

**6.** ***Program the fuses on the board***

**Note**: Programming fuses is a one-time operation. Make sure to triple check the fuse values before programming. Incorrectly programmed fuses can brick the board.

The fuse.bin file contains values that need to
be flashed in SOC.

```bash
od -t x4 SRK_1_2_3_4_fuse.bin
0000000 69a703dc ae487d0c 081db9ab 3d758739
0000020 48fe99e7 8d6a942d 7923cffe 9004862e
```

Use U-Boot fuse command

```bash
fuse prog 16 0 0x69a703dc
fuse prog 16 1 0xae487d0c
fuse prog 16 2 0x081db9ab
fuse prog 16 3 0x3d758739
fuse prog 16 4 0x48fe99e7
fuse prog 16 5 0x8d6a942d
fuse prog 16 6 0x7923cffe
fuse prog 16 7 0x9004862e
```




**7.** *** Create symlinks required for Yocto build***
For some reasons the Yocto build looks for the different key file and crt names. Create the following symlinks in the keys folder:
```bash
cd /opt/apps/cst-4.0.1/keys
ln -sf SRK1_sha256_secp256r1_v3_usr_key.pem SRK1_sha256_prime256v1_v3_ca_key.pem 
ln -sf SRK2_sha256_secp256r1_v3_usr_key.pem SRK2_sha256_prime256v1_v3_ca_key.pem 
ln -sf SRK3_sha256_secp256r1_v3_usr_key.pem SRK3_sha256_prime256v1_v3_ca_key.pem 
ln -sf SRK4_sha256_secp256r1_v3_usr_key.pem SRK4_sha256_prime256v1_v3_ca_key.pem 
```

```bash
cd /opt/apps/cst-4.0.1/crts
ln -sf SRK1_sha256_secp256r1_v3_usr_crt.pem SRK1_sha256_prime256v1_v3_ca_crt.pem 
ln -sf SRK2_sha256_secp256r1_v3_usr_crt.pem SRK2_sha256_prime256v1_v3_ca_crt.pem 
ln -sf SRK3_sha256_secp256r1_v3_usr_crt.pem SRK3_sha256_prime256v1_v3_ca_crt.pem 
ln -sf SRK4_sha256_secp256r1_v3_usr_crt.pem SRK4_sha256_prime256v1_v3_ca_crt.pem 
```



## Secure Provisioning SDK

Source can be found at [Github](https://github.com/nxp-mcuxpresso/spsdk?tab=readme-ov-file).

Documentation can be found at [SPSDK Documentation](https://spsdk.readthedocs.io/en/latest/).

Project [page](https://www.nxp.com/design/design-center/software/development-software/secure-provisioning-sdk-spsdk:SPSDK).


## Retrieving the Yocto Project Source Code

**Note**: See the i.MX Yocto Project User's Guide (UG10164) for how to set up and build Yocto project.
To set up the Yocto project for secure boot build, perform the following steps:

**1.** ***Set up Yocto Build Environment and Configuration***

**Note**: If the manifest file does not exist for a specific release, set up the Yocto build according to the **i.MX
Yocto Project User's Guide (UG10164)** and download the **meta-nxp-security-reference-design**
meta layer from [meta-nxp-security-reference-design](https://github.com/nxp-imx-support/meta-nxp-security-reference-design/) in the sources directory. Then proceed with Step 2.
```bash
repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-styhead -m
imx-6.12.3-1.0.0_security-reference-design.xml
repo sync
```
then you need to set up the build environment and configuration. When setting up the build environment, make sure to select the correct MACHINE and DISTRO for your build.
```bash
DISTRO=<DISTRO> MACHINE=<MACHINE> source imx-setup-release.sh -b <build directory>
```

Here are available DISTRO otions:
- **fsl-imx-wayland**: Pure Wayland graphics.
- **fsl-imx-xwayland**: Wayland graphics and X11. X11 applications using EGL are not supported.
- **fsl-imx-fb**: Frame Buffer graphics - no X11 or Wayland. Frame Buffer is not supported on i.MX 8 and i.MX9.

For this build we will use **fsl-imx-xwayland** and MACHINE will be **imx93evk**.

````bash
DISTRO=fsl-imx-xwayland MACHINE=imx93evk source imx-setup-release.sh -b build-imx93evk
````

**2** ***Add the meta-secure-boot layer to the Yocto project***
```bash
bitbake-layers add-layer ../sources/meta-nxp-security-reference-design/meta-secure-boot
```

**3**. ***Add CST or SPSDK in SIG_TOOL_PATH in local.conf***

**Note**: The absolute location of CST or SPSDK is required.
echo "SIG_TOOL_PATH = \"<path to cst package>\"" >> conf/local.conf

**4.** ****(Optional) Add the keys and crts directories in local.conf****
Note: The absolute location of the folder containing the keys and crts folders is required. If
SIG_DATA_PATH is not provided, the SIG_TOOL_PATH env value is used.
```bash
echo "SIG_DATA_PATH = \"<keys and crts folder>\"" >> conf/local.conf
```




## References
TP-IOT-SECURITY-IP-PROTECTION-IMX9-APP-PROCESSORS.pdf

UG10163
i.MX Linux User's Guide
Rev. LF6.12.34_2.1.0 — 30 September 2025 

UG10164
i.MX Yocto Project User's Guide
Rev. LF6.12.34_2.1.0 — 25 September 2025 