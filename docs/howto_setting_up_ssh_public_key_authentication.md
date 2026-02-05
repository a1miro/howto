# Setting Up SSH Public Key Authentication from Windows 11 to Ubuntu

## Step 1: Check if SSH Client is Installed on Windows 11

Open **PowerShell** and run:

```powershell
ssh -V
```

If not installed, install OpenSSH Client:
1. Go to **Settings → Apps → Optional Features**
2. Click **Add a feature**
3. Search for **OpenSSH Client** and install it

## Step 2: Generate SSH Key Pair (if you don't have one)

Open **PowerShell** and run:

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

**Or use RSA (older systems):**
```powershell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- Press **Enter** to accept default location: `C:\Users\YourUsername\.ssh\id_ed25519`
- Enter a passphrase (optional but recommended)
- This creates:
  - **Private key:** `C:\Users\YourUsername\.ssh\id_ed25519`
  - **Public key:** `C:\Users\YourUsername\.ssh\id_ed25519.pub`

## Step 3: Copy Public Key to Ubuntu Host

### Option A: Using `ssh-copy-id` (if available)

```powershell
ssh-copy-id username@rol-build-01
```

### Option B: Manual Copy (PowerShell)

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh username@rol-build-01 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Option C: Manual Copy Step-by-Step

1. **Display your public key:**
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

2. **Copy the output** (starts with `ssh-ed25519` or `ssh-rsa`)

3. **Connect to Ubuntu host:**
```powershell
ssh username@rol-build-01
```

4. **On Ubuntu, create .ssh directory and add the key:**
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

5. **Paste your public key** into the file, save and exit (`Ctrl+X`, `Y`, `Enter`)

6. **Set correct permissions:**
```bash
chmod 600 ~/.ssh/authorized_keys
exit
```

## Step 4: Test the Connection

```powershell
ssh username@rol-build-01
```

You should now connect **without entering a password** (only SSH key passphrase if you set one).

## Step 5: Create SSH Config File (Optional - Recommended)

Create/edit: `C:\Users\YourUsername\.ssh\config`

```powershell
notepad $env:USERPROFILE\.ssh\config
```

Add this configuration:

```conf
Host rol-build-01
    HostName rol-build-01
    User your_username
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Now you can simply connect with:
```powershell
ssh rol-build-01
```

## Troubleshooting

**If authentication still asks for password:**

1. **Check permissions on Ubuntu:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

2. **Verify SSH server config on Ubuntu:**
```bash
sudo nano /etc/ssh/sshd_config
```

Ensure these lines are present and uncommented:
```conf
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

3. **Restart SSH service on Ubuntu:**
```bash
sudo systemctl restart ssh
```

4. **Check SSH logs on Ubuntu:**
```bash
sudo tail -f /var/log/auth.log
```

5. **Verbose connection from Windows:**
```powershell
ssh -v username@rol-build-01
```

## Security Tips

- **Never share your private key** (`id_ed25519`)
- **Use a passphrase** to protect your private key
- **Disable password authentication** on Ubuntu after key setup:
  ```bash
  sudo nano /etc/ssh/sshd_config
  # Set: PasswordAuthentication no
  sudo systemctl restart ssh
  ```

You're all set! 🎉