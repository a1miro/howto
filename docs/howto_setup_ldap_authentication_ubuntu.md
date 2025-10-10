# How to install LDAP on Ubuntu 24.04 machine
 
As the DNS does not work from straight away, set the domain controller IP address in /etc/hosts file.

```bash
192.168.65.20 slehq.local
``` 
 
##Install required packages 

```bash
sudo apt update
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

## Discover domain

```bash
realm discover slehq.local
```
 
## Configuration steps 

Join the domain, use domain controller admin credentials for this, here I assume the user name is admin
```bash
sudo realm join --user=admin slehq.local
```
 
We have been experienced that the join command might say "it's already joined". If this is a case then  try "leave" command firs and then "join" again

```bash 
sudo realm leave
sudo realm join --user=admin slehq.local
```

Configure SSSD. Edit /etc/sssd/sssd.conf

```bash
[sssd]
domains = slehq.local
config_file_version = 2
services = nss, pam
[domain/slehq.local]
id_provider = ad
default_shell = /bin/bash
use_fully_qualified_names = False
fallback_homedir = /home/%u
```

Set permissions and restart SSSD

```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
```
 
Check SSSD agent status
```bash
sudo systemctl status sssd
```

Test if the configuration works
```bash
id firstname.surname@slehq.local
```
 
To ensure that the home directory will be created at first login 

```bash
sudo pam-auth-update --enable mkhomedir
```
 
Configure sshd, to allow "user domain" group. Add this line to /etc/ssh/sshd_config

```bash
AllowGroups "domain users"
GSSAPIAuthentication yes
GSSAPICleanupCredentials yes
```
 
Try to ssh to the Ubuntu 24.04 machine. Use "-o PreferredAuthentications=password" option for fist time login and create/deploy SSH keys for the passwordless login next time
```bash 
ssh -o PreferredAuthentications=password  firstname.surname@ukcrbld01
```