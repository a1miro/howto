Below are the recommended steps to install Docker CE on Ubuntu 24.04.

1) Remove old versions (if any)
```bash
sudo apt-get remove -y docker docker-engine docker.io containerd runc
```

2) Set up the repository
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

3) Add Docker’s APT repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4) Install Docker CE
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

5) Verify installation
```bash
sudo docker run --rm hello-world
```

Optional: Run Docker as a non-root user
```bash
sudo usermod -aG docker $USER
newgrp docker
```

If you hit any errors, tell me the exact message and I’ll help troubleshoot.
