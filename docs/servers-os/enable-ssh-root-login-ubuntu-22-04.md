# **Enable SSH Root Login on Ubuntu 22.04**

By default, Ubuntu **disables root login over SSH** for security reasons. Enabling root login is strongly discouraged in production environments because it increases the attack surface. However, if you need it (e.g., for testing or internal use), follow these steps carefully.

---

## **Prerequisites:**

- Server running **Ubuntu 22.04 LTS**.

- Access as a **non-root user with sudo privileges**.

- SSH already installed and running.

## **Steps to Enable SSH Root Login**

### **1. Login as a sudo user**

Connect to your server using your regular user account:

```
ssh username@your_server_ip
```

### **2. Switch to the root user**

Once logged in, elevate privileges:

```
sudo -i
```

You should now see the prompt as `root@your_server:~#`.

### **3. Edit the SSH configuration file**

Open the SSH daemon config file:

```
nano /etc/ssh/sshd_config
```

Look for the following parameters and modify as needed:

- **PermitRootLogin**
  Change from `prohibit-password` or `no` to:

  ```
  PermitRootLogin yes
  ```

- **PasswordAuthentication**
  Make sure it’s set to:

  ```
  PasswordAuthentication yes
  ```

  ⚠️ **Notes**:

- If lines are commented with `#`, remove the `#` before editing.

- Using `yes` allows password login, which is less secure. A better alternative is `PermitRootLogin prohibit-password` with SSH keys.

### **4. Save and exit**

In `nano`, press:

- `CTRL + O` → Enter (save)

- `CTRL + X` → Exit

### **5. Restart SSH service**

Apply changes by restarting the SSH service:

```
systemctl restart ssh
```

Check service status:

```
systemctl status ssh
```

Make sure it shows `active (running)`.

### **6. Test SSH root login**

From your local machine, try connecting directly as root:

```
ssh root@your_server_ip
```

If you enabled password authentication, enter the root password when prompted.

## **_Security Recommendations_**

Enabling root login directly over SSH is **not recommended**. To improve security:

- **Use SSH keys**
  Generate a key pair on your local machine:
  ```
  ssh-keygen -t ed25519
  ```
  Copy it to the server:
  ```
  ssh-copy-id root@your_server_ip
  ```
  Then disable password authentication:
  ```
  PasswordAuthentication no
  ```
- **Restrict SSH access**
  Edit `/etc/ssh/sshd_config`:
  ```
  AllowUsers root username
  ```
- **Change default SSH port**
  In `/etc/ssh/sshd_config`:
  ```
  Port 2222
  ```
  (Don’t forget to allow the new port in your firewall).
- **Use a firewall** (UFW or iptables) to limit SSH access.
  Example with UFW:

  ```
  ufw allow 2222/tcp
  ufw enable
  ```

  ✅ Now you should be able to log in as root via SSH, but consider disabling it once your task is complete for better security.
