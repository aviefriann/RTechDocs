# Enable SSH Root Login on Ubuntu 22.04

By default, Ubuntu **disables root login over SSH** for security reasons. Enabling root login is strongly discouraged in production environments because it increases the attack surface. However, if you need it (e.g., for testing or internal use), follow these steps carefully.

## Prerequisites

- Server running **Ubuntu 22.04 LTS**.

- Access as a **non-root user with sudo privileges**.

- SSH already installed and running.

## Steps

1. **Login as a sudo user**
   Connect to your server using your regular user account:

   ```bash
   ssh user@your_server_ip
   ```

2. **Switch to the root user**
   Once logged in, elevate privileges:
   ```bash
   sudo -i
   ```
   You should now see the prompt as `root@your_server:~#`.
