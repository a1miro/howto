
## Crypto device

In the `imx8mp.dtsi` device tree file for NXP i.MX8M Plus, the line:

```
crypto: crypto@30900000 { ... }
```

defines a **device tree node** for the **Data Co-Processor (DCP)** hardware block, which provides hardware-accelerated cryptographic functions, such as:

- AES encryption/decryption
- SHA hashing
- CRC32 calculation (including the one used for hardware-accelerated CRC32)

### Explanation of the Syntax

- **`crypto:`**  
  This is the node label. You can refer to this node elsewhere in the device tree using `&crypto`.

- **`crypto@30900000`**  
  This means a device called **"crypto"** is memory-mapped at address `0x30900000`.  
  (On i.MX8MP, this is the base address of the DCP hardware block.)

### Example from imx8mp.dtsi

```dts
crypto: crypto@30900000 {
    compatible = "fsl,imx8mp-dcp", "fsl,imx6sl-dcp";
    reg = <0x30900000 0x40000>;
    interrupts = <GIC_SPI 91 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 92 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 93 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 94 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk IMX8MP_CLK_DCP_CLK_ROOT>;
    clock-names = "ipg";
    status = "disabled";
};
```

### What does this node do?

- It **declares the presence and configuration of the DCP hardware** to the Linux kernel or other OS.
- The kernel's **DCP driver** (e.g., `drivers/crypto/fsl-dcp.c`) will probe this node and load the DCP driver if `status = "okay";`.
- The DCP driver then exposes hardware crypto (AES, SHA, CRC32, etc.) to userspace via the Linux Crypto API.

---

#### **Summary**

- `crypto: crypto@30900000` is the device tree node for the i.MX8MP's hardware cryptographic accelerator (DCP).
- It is essential for enabling hardware-accelerated crypto and CRC32 on i.MX8MP through the kernel.

## IMX8MP DCP module

To enable DCP (Data Co-Processor) in the i.MX8MP device tree, you generally need to ensure that the DCP node is present and set to status = "okay"; in your device tree source (.dts) file. The DCP is responsible for hardware-accelerated cryptographic operations, including CRC32.

Here’s how you can enable DCP in the device tree:

### Locate the DCP Node
The DCP node is typically found in the SoC's device tree include file, for example:
arch/arm64/boot/dts/freescale/imx8mp.dtsi

It looks like this dts:

```dts
dcp: dcp@30900000 {
    compatible = "fsl,imx8mq-dcp", "fsl,imx6sl-dcp";
    reg = <0x30900000 0x40000>;
    interrupts = <GIC_SPI 91 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 92 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 93 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 94 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk IMX8MP_CLK_DCP_CLK_ROOT>;
    clock-names = "ipg";
    status = "disabled";
};
```

### Enable the Node
In your board-specific .dts or .dtsi (e.g., imx8mp-evk.dts), enable the DCP by setting:

```dts
&dcp {
    status = "okay";
};
```

This overrides the default status = "disabled"; in the include file.

### Recompile and Deploy the Device Tree
Rebuild your device tree:

```bash
make ARCH=arm64 dtbs
Deploy the updated .dtb to your bootloader or kernel.
```

### Check in Linux
After boot, confirm DCP is enabled:

```bash
dmesg | grep dcp
```
or

```bash
ls /sys/class/misc/dcp*
```


### Verify Kernel Support
Ensure the following kernel configs are enabled:
```
CONFIG_CRYPTO_DEV_FSL_DCP
CONFIG_CRYPTO_DEV_FSL_DCP_CRC
```
---
### Summary
Add this to your board .dts:

```dts
&dcp {
    status = "okay";
};
```

And make sure the DCP node exists in the SoC .dtsi and the kernel driver is enabled.
