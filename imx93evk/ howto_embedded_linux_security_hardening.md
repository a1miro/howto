## 1. International Standards

### IEC 62443
The most widely adopted standard for **Industrial Automation and Control Systems (IACS)** security.
- Covers product development, system integration, and operations
- Defines **Security Levels (SL 1–4)** — from basic to state-actor resistance
- Relevant parts: IEC 62443-4-1 (secure development lifecycle), IEC 62443-4-2 (component requirements)

### ISO/SAE 21434
**Automotive cybersecurity** standard — mandatory for vehicles (UNECE WP.29 regulation).
- Covers threat analysis (TARA), secure development, incident response
- Relevant for embedded Linux in automotive ECUs

### ISO 27001 / 27002
General **information security management** — applicable to IoT backend/cloud parts.

---

## 2. IoT-Specific Standards & Guidelines

### ETSI EN 303 645
European standard for **consumer IoT security** (smart home, wearables).
Key provisions:
- No universal default passwords
- Implement a means to manage reports of vulnerabilities
- Keep software updated
- Securely store credentials
- Minimize exposed attack surface

### NIST IR 8259 / SP 800-213
US NIST guidelines for **IoT device cybersecurity**.
- Defines baseline IoT security capabilities
- SP 800-213 specifically for federal IoT systems

### PSA Certified (ARM)
**Platform Security Architecture** — hardware-rooted security for Arm-based embedded devices.
- Defines threat models and security requirements
- Levels: PSA Certified Level 1, 2, 3

---

## 3. Linux / Software Specific

### CIS Benchmarks (Linux)
**Center for Internet Security** hardening benchmarks for Linux distributions — filesystem permissions, kernel parameters, service hardening.

### STIG (Security Technical Implementation Guides)
US DoD hardening guidelines — very detailed, applicable to embedded Linux.

### OpenChain ISO 5230
Focuses on **open source license compliance** (indirectly security-relevant for supply chain).

### SBOM (Software Bill of Materials)
Not a standard itself but required by:
- US Executive Order 14028 (2021)
- NTIA guidelines
- Helps track vulnerable components (CVEs)

---

## 4. Embedded Linux Specific Practices (aligned with above standards)

| Area | Relevant Standard |
|------|------------------|
| Secure boot | IEC 62443-4-2, PSA Certified |
| Kernel hardening | CIS Benchmarks, STIG |
| OTA updates | ETSI EN 303 645, NIST IR 8259 |
| Credential management | ETSI EN 303 645 |
| Automotive | ISO/SAE 21434 |
| Industrial/ICS | IEC 62443 |
| Supply chain / SBOM | NIST SP 800-161, EO 14028 |

---

## 5. Emerging / Regional Regulations

- **EU Cyber Resilience Act (CRA)** — 2024, mandatory for products with digital elements sold in EU, including IoT
- **UK PSTI Act** — Product Security and Telecommunications Infrastructure Act, in force 2024
- **UNECE WP.29 R155** — Mandatory automotive cybersecurity type approval

For embedded Linux on i.MX93 (your current context), **IEC 62443-4-2** and **PSA Certified** are the most directly applicable, with **ETSI EN 303 645** if the product is a consumer device.


Here is a comprehensive checklist organized by security domain:

---

## 6. Secure Boot Chain

**Goal:** Ensure only authenticated firmware/OS runs on the device.

- Enable **AHAB (Advanced High Assurance Boot)** on i.MX93 — NXP's hardware-enforced secure boot
- Sign bootloader (U-Boot), kernel, and device tree with your private key
- Fuse/lock the SRK (Super Root Key) hash into eFuses — one-way, irreversible
- Enable **HAB/AHAB closed configuration** in production (prevents unsigned images)
- Use **dm-verity** for read-only rootfs integrity verification at runtime
- Use **IMA/EVM** (Integrity Measurement Architecture) for file-level integrity

Yocto:
```conf
MACHINE_FEATURES:append = " optee"
```

---

## 7. Cryptographic Key Management

- Store keys in **i.MX93 EdgeLock Secure Enclave (ELE)** — hardware security module on-chip
- Use **OP-TEE** (Trusted Execution Environment) for key operations
- Never store private keys in rootfs in plaintext
- Use **PKCS#11** interface to access hardware-stored keys from applications
- Rotate keys via secure OTA process

---

## 8. Filesystem & Storage Security

- Mount rootfs **read-only** (`ro` in fstab / kernel cmdline)
- Use **dm-verity** for rootfs (integrity) or **dm-crypt / LUKS** for confidentiality
- Separate writable partitions (var, data) with `noexec`, `nosuid`, `nodev` mount flags
- Encrypt sensitive data partitions (using ELE-derived keys)
- Disable unused filesystems in kernel config (`CONFIG_CRAMFS=n`, etc.)

Example fstab flags:
```
/dev/mmcblk0p2  /data  ext4  noexec,nosuid,nodev  0 2
```

---

## 9. Kernel Hardening

Enable in `linux-imx` kernel config or via `meta-security` layer:

```
CONFIG_SECURITY=y
CONFIG_SECURITY_YAMA=y          # restrict ptrace
CONFIG_STACKPROTECTOR_STRONG=y
CONFIG_FORTIFY_SOURCE=y
CONFIG_RANDOMIZE_BASE=y         # KASLR
CONFIG_STRICT_KERNEL_RWX=y
CONFIG_DEBUG_RODATA=y
CONFIG_PANIC_ON_OOPS=y
CONFIG_MODULES=n                # disable loadable modules if not needed
CONFIG_KPROBES=n
CONFIG_DEVMEM=n                 # disable /dev/mem
CONFIG_DEVKMEM=n
```

Kernel command line:
```
init_on_alloc=1 init_on_free=1 page_alloc.shuffle=1 slub_debug=F
```

---

## 10. User & Privilege Management

- Do **not** run applications as root — create dedicated service users
- Use `systemd` service hardening directives:
```ini
[Service]
User=myapp
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```
- Remove or lock the `root` password in production
- Disable root login over SSH (`PermitRootLogin no`)
- Use **PAM** with strong password policies if local login is needed

---

## 11. Network Security

- Disable all unused network services (telnet, FTP, rpcbind)
- Use **nftables** or **iptables** — default-deny firewall, allow only needed ports
- Enable **SSH hardening** in sshd_config:
```
PermitRootLogin no
PasswordAuthentication no       # key-based only
Protocol 2
AllowUsers myuser
```
- Use **TLS 1.2+ only** for all network communication
- Disable unused network interfaces at boot
- Consider **MAC address randomization** if applicable

---

## 12. Secure OTA Updates

- Sign update packages (e.g., **SWUpdate** or **Mender** with signature verification)
- Verify signature before applying any update
- Use A/B partition scheme for atomic rollback
- Encrypt update payloads in transit (TLS) and optionally at rest
- Authenticate the update server (certificate pinning)

Yocto recipes: `swupdate`, `mender`

---

## 13. Mandatory Access Control (MAC)

Use **SELinux** or **AppArmor** to confine processes:

```conf
# In Yocto distro config
DISTRO_FEATURES:append = " selinux"
IMAGE_INSTALL:append = " packagegroup-core-selinux"
```

Or use **Smack** (simpler, common in automotive/embedded):
```conf
DISTRO_FEATURES:append = " smack"
```

---

## 14. Remove Debug & Development Artifacts

In production images:
- Remove: `gdb`, `strace`, `ltrace`, `tcpdump`, `wget`, `curl`, `ssh-server` (if not needed), shells (`bash` → use `sh` only or remove)
- Disable UART debug console (or require authentication)
- Disable JTAG/SWD in production (fuse via eFuses)
- Remove kernel debug symbols
- Set `EXTRA_IMAGE_FEATURES` appropriately:

```conf
# Remove from production:
# EXTRA_IMAGE_FEATURES ?= "debug-tweaks"

# Production:
EXTRA_IMAGE_FEATURES = ""
```

---

## 15. Secure Communication & Certificates

- Use a **TPM 2.0** or the **i.MX93 ELE** for device identity (device certificate)
- Provision unique per-device TLS certificates at manufacturing
- Use **certificate pinning** for any cloud connection
- Implement certificate rotation via OTA

---

## 16. Logging & Monitoring

- Enable **auditd** for security-relevant events (login, privilege escalation, file access)
- Protect logs from tampering (send to remote syslog over TLS)
- Log boot integrity events (IMA log, AHAB log)
- Define alerts for: repeated auth failures, unexpected process spawns, rootfs write attempts

---

## 17. Supply Chain & SBOM

- Generate an **SBOM** (Software Bill of Materials) from Yocto:
  ```sh
  bitbake -c create_spdx <image>
  ```
- Or use tools like `Vigiles` to continuously monitor CVEs for components in your SBOM
  ```sh
   BBLAYERS += "${BSPDIR}/sources/meta-timesys"
   INHERIT += "vigiles"
  ```
- Regularly scan SBOM against CVE databases (e.g., with `cve-check` Yocto class)
- Pin layer revisions in `bblayers.conf` / use `kas` for reproducible builds
- Audit all third-party recipes and patches

Yocto recipes: `vigiles`, `spdx-tools`

---

## 18. Physical Security

- Disable JTAG/SWD debug ports via eFuse (on i.MX93)
- Disable UART boot mode in production (set boot fuses)
- Enable **tamper detection** if hardware supports it
- Store sensitive keys in ELE, not accessible from Linux userspace directly

---

## 20. Recommended Yocto Layers

| Layer | Purpose |
|-------|---------|
| `meta-security` | SELinux, AppArmor, IMA, checksec |
| `meta-selinux` | SELinux policies |
| `meta-tpm` | TPM 2.0 support |
| `meta-imx` | NXP ELE / AHAB support |
| `meta-updater` (Mender) | OTA updates |
| `meta-swupdate` | SWUpdate support |
| `meta-timesys` | Vigiles CVE scanning and SBOM generation |

---

