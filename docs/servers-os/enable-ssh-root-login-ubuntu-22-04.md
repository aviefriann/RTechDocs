# Enable SSH Root Login on Ubuntu 22.04

By default, Ubuntu disables root login over SSH for security reasons. If you still want to enable it (not recommended for production), follow these steps carefully.

## Steps

## 1. **Login as a sudo user**

```bash
ssh user@your_server_ip
```

## 2. **Switch to root**

```bash
sudo -i
```

## 3. **Edit SSH configuration**

```bash
nano /etc/ssh/sshd_config
```

Find and update these lines (uncomment or modify if needed):

```
PermitRootLogin yes
PasswordAuthentication yes
```

## 4. **Restart SSH service**

```bash
systemctl restart ssh
```

## 5. **Test SSH root login**

```bash
ssh root@your_server_ip
```

## Security Warning

Enabling SSH root login is **not recommended**. Use SSH keys and a normal user with `sudo` privileges instead for better security.
