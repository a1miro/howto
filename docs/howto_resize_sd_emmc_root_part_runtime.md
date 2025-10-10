After flashing Yocto SD card image, often the rootfile system occypies a small portion of available disk space, this document describes the procedure for resizing the root partion to all available space.
To resize the **root partition** using `parted`, you need to follow these steps:

**Warning:**  
- Resizing a mounted root partition is risky and generally not supported while the partition is mounted.
- You should boot from a **Live CD/USB** (not your usual OS) to safely resize the root partition.
- Always **backup your data** before modifying partitions.

---

## Step-by-Step Guide: Resize Root Partition Using `parted`

### 1. **Boot from a Live Linux USB/CD**
- Shut down your system.
- Boot from a live environment (Ubuntu live, rescue disk, etc.).

### 2. **Open a Terminal**

### 3. **Identify Your Disk and Partition**
Run:
```sh
lsblk
```
or
```sh
sudo parted -l
```
Find your root partition (e.g., `/dev/sda1`).

### 4. **Start parted**
```sh
sudo parted /dev/sda
```

### 5. **Check Current Partitions**
In parted prompt, type:
```sh
print
```

### 6. **Resize Partition**
Suppose your root partition is number 1 and you want to expand it to the end of the disk.

**Syntax:**  
```sh
resizepart <partition_number> <END>
```
- `<partition_number>`: Partition number (e.g., 1)
- `<END>`: End location (e.g., 100%) or the last sector

**Example:**  
```sh
resizepart 1 100%
```
This resizes partition 1 to occupy all available space.

### 7. **Quit parted**
```sh
quit
```

### 8. **Resize the Filesystem**
After resizing the partition, you need to expand the filesystem.

**For ext4:**
```sh
sudo e2fsck -f /dev/sda1
sudo resize2fs /dev/sda1
```

**For xfs:**
```sh
sudo xfs_growfs /mnt/yourroot
```
(_Replace `/mnt/yourroot` with your actual mount point_)

---

## **Summary Table**

| Step                      | Command Example                       |
|---------------------------|---------------------------------------|
| Boot live CD              | (BIOS/UEFI boot menu)                 |
| Identify disk/partition   | `lsblk` / `sudo parted -l`            |
| Start parted              | `sudo parted /dev/sda`                |
| Print partitions          | `print`                               |
| Resize partition          | `resizepart 1 100%`                   |
| Quit parted               | `quit`                                |
| Resize ext4 filesystem    | `sudo resize2fs /dev/sda1`            |
| Resize xfs filesystem     | `sudo xfs_growfs /mnt/yourroot`       |

---

### **Important Notes**
- Always double-check the device name (`/dev/sda`, `/dev/nvme0n1`, etc.).
- Make sure the partition is unmounted if not using a live environment.
- For root, use a live environment to avoid data loss.
