
## 1. **Change Docker’s Root Directory (images, volumes, containers, etc.)**

Docker stores all images, volumes, containers, etc. under `/var/lib/docker` by default. You can change this by setting the `--data-root` option.

### **a. Edit the daemon configuration file (Recommended for modern Docker)**

On most systems, edit `/etc/docker/daemon.json` and add:

````json name=/etc/docker/daemon.json
{
  "data-root": "/mnt/new-docker-root"
}
````

- Replace `/mnt/new-docker-root` with the path on your desired drive or partition.

### **b. Restart Docker**

```bash
sudo systemctl restart docker
```

---

## 2. **Move Existing Data (Optional)**

If you already have images/volumes and want to keep them:
1. Stop Docker:
    ```bash
    sudo systemctl stop docker
    ```
2. Copy/move the contents of `/var/lib/docker` to your new directory:
    ```bash
    sudo rsync -aHAXx /var/lib/docker/ /mnt/new-docker-root/
    ```
3. Set `data-root` as above and restart Docker.

---

## 3. **Verify**

- Run `docker info` and check `Docker Root Dir:` to confirm the new location.
- All new images, volumes, and containers will now be stored under the new path.

---

## 4. **For Docker Compose Volumes**

For bind mounts or advanced setups, use absolute paths in your `docker-compose.yml` as previously described. Named volumes will still be placed under the new `data-root`.

---

## 5. **Summary Table**

| Purpose                    | Configuration File            | Option/Entry                | Example Value               |
|----------------------------|-------------------------------|-----------------------------|-----------------------------|
| Change Docker root storage | /etc/docker/daemon.json       | `"data-root"`               | `"/mnt/new-docker-root"`    |
| Move existing data         | —                             | —                           | Use `rsync` as above        |

---

### **Notes**
- This changes the storage for **all** Docker-managed data (images, volumes, containers, etc.).
- Make sure the new path is on your desired partition/drive and has proper permissions for Docker (`root` owns it).
- If you have a lot of data, moving can take time.
- Volumes created via `docker volume create` or `docker-compose` named volumes will also be stored in the new location.

---

**Let me know if you need help with a specific OS, storage setup, or if you want step-by-step migration instructions!**




1. **Stopped the Docker service**
2. **Copied or moved** all contents from `/var/lib/docker` to your new location (as set in `/etc/docker/daemon.json` with `"data-root": "/mnt/new-docker-root"` or similar)
3. **Restarted Docker** and verified that everything (containers, images, volumes) works correctly from the new location

**…then you can safely delete the original `/var/lib/docker` directory.**

---

### **Checklist Before Deleting `/var/lib/docker`:**

- [x] Docker is stopped:  
  ```bash
  sudo systemctl stop docker
  ```
- [x] All contents are copied (use `rsync -aHAXx` for best results):
  ```bash
  sudo rsync -aHAXx /var/lib/docker/ /mnt/new-docker-root/
  ```
- [x] `/etc/docker/daemon.json` is set with `"data-root"` pointing to your new location
- [x] Docker is restarted and verified:
  ```bash
  sudo systemctl start docker
  docker info | grep "Docker Root Dir"
  ```
- [x] All containers/images/volumes are visible and functioning

---

### **To Delete:**
```bash
sudo rm -rf /var/lib/docker
```

---

**Warning:**  
- Once deleted, you cannot revert unless you have backups.
- Always verify your data in the new location before deleting.

---

**Let me know if you want a step-by-step migration script or checks for your setup!**
