# Configure Static IP on Ubuntu using Netplan

Netplan is the default network configuration tool in Ubuntu (from 17.10 onwards).
It uses YAML-based configuration files to configure networking.

This guide explains how to configure a static IP address on Ubuntu using Netplan.

---

## Step 1: Identify Network Interface

First, list all network interfaces:

```bash
ip link show
```

Example output:

```
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
```

In this case, the interface is `ens33`.

---

## Step 2: Backup Current Netplan Configuration

Netplan configs are stored in `/etc/netplan/`.
Backup the existing configuration file:

```bash
sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.bak
```

---

## Step 3: Edit Netplan Configuration

Open the config file:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Example static IP configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

- `ens33` → Replace with your interface name.
- `192.168.1.100/24` → Desired static IP and subnet.
- `192.168.1.1` → Gateway address.
- `8.8.8.8, 1.1.1.1` → DNS servers.

---

## Step 4: Apply Changes

After saving the file, apply configuration:

```bash
sudo netplan apply
```

---

## Step 5: Verify Configuration

Check assigned IP:

```bash
ip addr show ens33
```

Test internet connectivity:

```bash
ping -c 4 google.com
```

---

## Step 6: Troubleshooting

If the connection fails, try:

```bash
sudo netplan try
```

This allows rollback if configuration is invalid.

---

✅ Your Ubuntu server now has a static IP configured using Netplan!
