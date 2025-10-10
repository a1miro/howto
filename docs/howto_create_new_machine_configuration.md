# How to create a new machine **defconfig** for the Linux kernel and a **machine configuration** in a Yocto BSP meta layer.

Here’s a step-by-step guide for **adding a new machine configuration based on the i.MX8MP (NXP i.MX 8M Plus) and a Variscite CustomBoard** in both the **Linux kernel** and **Yocto BSP**. This guide assumes you have a working Yocto environment (e.g., using `meta-freescale`, `meta-variscite`, or similar BSPs).

---

## **1. Linux Kernel: Adding a New defconfig for Your Custom Board**

### **a. Start with an Existing defconfig**
- Find a similar board’s defconfig (e.g., `imx8mp_var_dart_defconfig` in `arch/arm64/configs/`).
- Copy it as a base:
  ```bash
  cp arch/arm64/configs/imx8mp_var_dart_defconfig arch/arm64/configs/imx8mp_var_custom_defconfig
  ```

### **b. Customize Configuration**
- Run:
  ```bash
  make ARCH=arm64 imx8mp_var_custom_defconfig
  make ARCH=arm64 menuconfig
  ```
- Enable/disable kernel options as required for your CustomBoard (e.g., peripherals, drivers).

### **c. Save Your defconfig**
- After changes, save:
  ```bash
  make ARCH=arm64 savedefconfig
  cp defconfig arch/arm64/configs/imx8mp_var_custom_defconfig
  ```

### **d. Device Tree**
- Duplicate and modify an existing device tree (e.g., `imx8mp-var-dart.dts`) for your hardware:
  ```bash
  cp arch/arm64/boot/dts/freescale/imx8mp-var-dart.dts arch/arm64/boot/dts/freescale/imx8mp-var-custom.dts
  # Edit imx8mp-var-custom.dts for your board specifics
  ```

---

## **2. Yocto BSP: Adding a New Machine Configuration**

### **a. Create a New Machine Configuration File**
- Go to your BSP layer (e.g., `meta-variscite-bsp/conf/machine/`).
- Copy a similar machine file:
  ```bash
  cp imx8mp-var-dart.conf imx8mp-var-custom.conf
  ```
- Edit `imx8mp-var-custom.conf`:
  - Set:
    ```conf
    MACHINEOVERRIDES =. "imx8mp-var-custom:imx8mp:mx8:mx8m:use-mainline-bsp:"
    UBOOT_CONFIG ??= "sd"
    KERNEL_DEVICETREE = "freescale/imx8mp-var-custom.dtb"
    ```

### **b. Add/Override Kernel defconfig and Device Tree**
- In your kernel recipe (often `linux-variscite` or `linux-fslc`):
  - Create a directory:  
    `recipes-kernel/linux/linux-variscite/imx8mp-var-custom/`
  - Place your `defconfig` and `imx8mp-var-custom.dts` here (or patch the kernel recipe to use your files).

### **c. Update Yocto Build Configuration**
- In your Yocto build directory:
  ```bash
  MACHINE=imx8mp-var-custom bitbake core-image-minimal
  ```
- Ensure your new machine is available by adding your BSP layer to `bblayers.conf`.

---

## **3. References and Examples**

### **Yocto Machine Example**
```conf
# meta-variscite-bsp/conf/machine/imx8mp-var-custom.conf
require conf/machine/include/imx8mp.inc

SOC_FAMILY = "mx8"
KERNEL_DEVICETREE = "freescale/imx8mp-var-custom.dtb"
UBOOT_CONFIG ??= "sd"
SERIAL_CONSOLE = "115200 ttyLP0"
IMAGE_FSTYPES += "wic.zst"
```

### **Kernel defconfig Location**
```
meta-variscite-bsp/recipes-kernel/linux/linux-variscite/imx8mp-var-custom/defconfig
```

### **Device Tree Location**
```
meta-variscite-bsp/recipes-kernel/linux/linux-variscite/imx8mp-var-custom/imx8mp-var-custom.dts
```

---

## **4. Useful References**
- [Yocto Project Mega-Manual: Creating a New Machine](https://docs.yoctoproject.org/dev-manual/new-machine.html)
- [meta-freescale: Supported Machines](https://github.com/Freescale/meta-freescale/tree/master/conf/machine)
- [Variscite Yocto Wiki](https://variwiki.com/index.php?title=Yocto_Build_Release)  
- [NXP i.MX Yocto Guide](https://www.nxp.com/docs/en/user-guide/UG10164.pdf)

---

**Summary of Steps:**
1. **Kernel:** Copy/modify defconfig and device tree for your board.
2. **Yocto:** Copy/modify machine `.conf`, point to your device tree and defconfig, and build.
3. **Test:** Build and boot, then iterate as needed.

Let me know if you want a concrete example for any of these files or need help with a specific Yocto layer or kernel version!