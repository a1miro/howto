## Read-Only Rootfs in Yocto with Writable /etc, /var, /data

### Step 1 — Enable read-only rootfs IMAGE_FEATURE

In your image recipe or `local.conf`:

```conf
EXTRA_IMAGE_FEATURES += "read-only-rootfs"
```

This sets `ro` on the rootfs kernel cmdline and configures the image accordingly.

---

### Step 2 — Writable /var via tmpfs (volatile)

Yocto's `volatile-binds` package automatically bind-mounts volatile tmpfs paths over var subdirectories. Add it to your image:

```conf
IMAGE_INSTALL:append = " volatile-binds"
```

It uses `/etc/volatile.conf` (or `/usr/lib/tmpfiles.d/`) to create and bind-mount dirs like log, run, tmp from a tmpfs at boot. This is **volatile** — content is lost on reboot, which is correct for run, `/var/lock`, etc.

If you need var to be **persistent** across reboots, mount it from a dedicated partition (see Step 4).

---

### Step 3 — Writable /etc via overlayfs-etc

Since Yocto Kirkstone (4.0), there is built-in `overlayfs-etc` support. It mounts an OverlayFS over etc using a persistent upper layer stored on a writable partition (e.g., `/data`).

In your distro or machine config:

```conf
# Enable overlayfs-etc feature
EXTRA_IMAGE_FEATURES += "read-only-rootfs read-only-rootfs-delayed-writes"
IMAGE_INSTALL:append = " overlayfs-etc"
```

Configure the upper layer location in `overlayfs-etc.conf` (a bbappend or config file):

```conf
# /etc/overlayfs-etc.conf
OVERLAY_UPPER=/data/overlay/etc/upper
OVERLAY_WORK=/data/overlay/etc/work
```

This means etc changes are written to `/data/overlay/etc/upper/` — persistent and survives reboots.

---

### Step 4 — Dedicated writable /data partition

Define a separate ext4 partition in your WIC kickstart file (`*.wks`):

```wks
part /boot  --source bootimg-partition --ondisk mmcblk0 --fstype=vfat --label boot   --active --align 4096 --size 64M
part /       --source rootfs            --ondisk mmcblk0 --fstype=ext4 --label rootfs  --align 4096 --size 512M
part /var    --ondisk mmcblk0           --fstype=ext4 --label var     --align 4096 --size 128M
part /data   --ondisk mmcblk0           --fstype=ext4 --label data    --align 4096 --size 256M
```

Mount them in `/etc/fstab` (baked into the image):

```
/dev/mmcblk0p2   /        ext4   ro,noatime                    0 1
/dev/mmcblk0p3   /var     ext4   rw,noatime,nosuid,nodev       0 2
/dev/mmcblk0p4   /data    ext4   rw,noatime,nosuid,nodev,noexec 0 2
tmpfs            /tmp     tmpfs  rw,nosuid,nodev,size=64m      0 0
```

---

### Step 5 — First-boot /data initialization

On first boot `/data` is empty, so the overlayfs-etc upper dir and var content won't exist yet. Add a systemd service or use `tmpfiles.d` to create required directories:

```ini
# /usr/lib/tmpfiles.d/data-dirs.conf
d /data/overlay/etc/upper  0755 root root -
d /data/overlay/etc/work   0755 root root -
d /var/log                 0755 root root -
d /var/lib                 0755 root root -
```

Or handle in a first-boot systemd oneshot service that `mkfs.ext4` and mounts `/data` if unformatted.

---

### Summary

| Mount Point | Mechanism | Persistent |
|-------------|-----------|-----------|
| `/` (rootfs) | `read-only-rootfs` IMAGE_FEATURE | yes (read-only) |
| etc | `overlayfs-etc` (upper on `/data`) | yes |
| var | Separate ext4 partition or tmpfs | yes (partition) / no (tmpfs) |
| `/data` | Separate ext4 partition | yes |
| tmp | tmpfs | no (volatile) |

---

### Minimal Yocto config recap

```conf
# image recipe or local.conf
EXTRA_IMAGE_FEATURES += "read-only-rootfs"
IMAGE_INSTALL:append = " volatile-binds overlayfs-etc"
```

```conf
# distro.conf or machine.conf
MACHINE_FEATURES:append = " overlayfs"
```