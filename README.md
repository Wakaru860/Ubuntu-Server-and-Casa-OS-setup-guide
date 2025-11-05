# Ubuntu Server + CasaOS Home Server Setup Guide

## What You'll Need
- A computer/laptop to use as your server (minimum 2GB RAM recommended)
- USB drive (8GB or larger) for installation
- Ethernet cable (recommended for initial setup but not required)
- Another computer to create the installation media

---

## Part 1: Installing Ubuntu Server

### Step 1: Download Ubuntu Server
1. Go to https://ubuntu.com/download/server
2. Download the latest LTS (Long Term Support) version
3. Save the ISO file to your computer

### Step 2: Create Bootable USB Drive

On Windows:
1. Download Rufus from https://rufus.ie
2. Insert your USB drive
3. Open Rufus
4. Select your USB drive
5. Click "SELECT" and choose the Ubuntu Server ISO
6. Click "START" and wait for it to finish

On Mac/Linux:
1. Download balenaEtcher from https://www.balena.io/etcher
2. Insert your USB drive
3. Open Etcher
4. Select the Ubuntu Server ISO
5. Select your USB drive
6. Click "Flash!" and wait for completion

### Step 3: Boot from USB
1. Insert the USB drive into your server machine
2. Restart the machine
3. Press the boot menu key during startup (usually F12, F2, F10, or DEL - depends on your motherboard)
4. Select your USB drive from the boot menu

### Step 4: Install Ubuntu Server
Follow the installation wizard:

1. Language: Select your language 

2. Keyboard: Choose your keyboard layout

3. Installation Type: Select "Ubuntu Server"

4. Network: 
   - If using ethernet, it should auto-configure
   - Note down the IP address shown (you'll need this later)

5. Proxy: Leave blank unless you need one

6. Mirror: Use the default Ubuntu mirror

7. Storage:
   - Select "Use an entire disk"
   - Choose your hard drive
   - Confirm the layout

8. Profile Setup:
   - Your name: Enter your name
   - Server name: Choose a name (e.g., "homeserver")
   - Username: Create a username (all lowercase, no spaces)
   - Password: Create a strong password
   

9. SSH Setup:
   - Check the box for "Install OpenSSH server"
   - This allows remote access

10. Featured Snaps: Skip for now (press Done)

11. Installation: Wait for it to complete (5-15 minutes)

12. Reboot: Remove the USB drive when prompted and press Enter

---

## Part 2: Initial Server Setup

### Step 1: Log In
1. After reboot, you'll see a login prompt
2. Enter your username and password
3. You're now at the command line!

### Step 2: Update the System
Run these commands one at a time:

```bash
sudo apt update
```

```bash
sudo apt upgrade -y
```

This updates all system packages (may take a few minutes).

### Step 3: Set a Static IP (Recommended)
To prevent your server's IP from changing:

1. Check your current IP:
```bash
ip a
```

Look for the IP address under your network interface (usually `eth0` or `enp*`).

2. Find your gateway:
```bash
ip route | grep default
```

3. Edit the network configuration:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

4. Modify it to look like this (adjust IPs to match your network):
```yaml
network:
  ethernets:
    enp3s0:  # Replace with your interface name
      addresses:
        - 192.168.1.100/24  # Your desired static IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: default
          via: 192.168.1.1  # Your router's IP (gateway)
  version: 2
```

5. Save and exit:
   - Press `Ctrl + X`
   - Press `Y`
   - Press `Enter`

6. Apply the changes:
```bash
sudo netplan apply
```

---

## Part 3: Installing CasaOS

### Step 1: Install CasaOS
Run this single command:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

This will:
- Download CasaOS
- Install all dependencies
- Set up the web interface
- Take 5-10 minutes to complete

### Step 2: Access CasaOS Web Interface
1. Once installation completes, you'll see a message with a URL
2. On another computer on the same network, open a web browser
3. Navigate to: `http://your-server-ip` (e.g., `http://192.168.1.100`)
4. You should see the CasaOS welcome screen

### Step 3: Initial CasaOS Setup
1. Create a CasaOS username and password
2. You'll be taken to the CasaOS dashboard
3. Explore the App Store to install applications

---

## Part 4: Using CasaOS

### Installing Apps
CasaOS makes it easy to install services:

1. Click the App Store icon
2. Browse available apps:
   - File Sharing: Nextcloud, Filebrowser
   - Media: Plex, Jellyfin, Emby
   - Download: qBittorrent, Transmission
   - Smart Home: Home Assistant
   - Backup: Duplicati
3. Click an app and press Install
4. Wait for installation to complete
5. Access the app from your dashboard

### Managing Storage
1. Click Storage in the sidebar
2. View connected drives
3. Create folders and manage permissions

### System Settings
- Click the Settings icon 
- View system resources (CPU, RAM, storage)
- Manage network settings
- Configure security options

---

## Part 5: Accessing Your Server Remotely

### From Your Local Network
Simply use: `http://your-server-ip` in any browser

### SSH Access (Command Line)
From another computer:

```bash
ssh your-username@your-server-ip
```

Example:
```bash
ssh john@192.168.1.100
```

### Accessing from Outside Your Home (Optional)
This requires more advanced setup:
1. Set up port forwarding on your router
2. Use a dynamic DNS service (like DuckDNS)
3. Consider using a VPN for security (like Tailscale or WireGuard)
4. Security warning: Exposing your server to the internet requires proper security measures

---

## Useful Commands

### Check CasaOS Status
```bash
sudo systemctl status casaos
```

### Restart CasaOS
```bash
sudo systemctl restart casaos
```

### Update CasaOS
```bash
curl -fsSL https://get.casaos.io | sudo bash
```

### Check System Resources
```bash
htop
```
(Press `q` to quit)

### Check Disk Space
```bash
df -h
```

### Reboot Server
```bash
sudo reboot
```

### Shutdown Server
```bash
sudo shutdown now
```

---

## Troubleshooting

### Can't Access CasaOS Web Interface
1. Check if CasaOS is running:
   ```bash
   sudo systemctl status casaos
   ```

2. Check your server's IP address:
   ```bash
   ip a
   ```

3. Make sure you're on the same network

4. Try accessing with port explicitly: `http://your-ip:80`

### Forgot Your Password
For Ubuntu:
1. Boot into recovery mode
2. Reset password from command line

For CasaOS:
1. SSH into your server
2. Run: `sudo casaos-cli user-reset`

### Server Won't Boot
1. Check if the hard drive is properly connected
2. Check BIOS boot order
3. Try booting from the USB again to repair

## Important Security Tips

1. Keep your system updated:
   ```
   sudo apt update && sudo apt upgrade -y
   ```

2. Use strong passwords for all accounts

3. Enable firewall (UFW):
   ```
   sudo ufw enable
   sudo ufw allow 22
   sudo ufw allow 80
   sudo ufw allow 443
   ```

4. Regular backups - Don't store important data without backups

5. Don't expose unnecessary ports to the internet

6. Consider fail2ban for SSH protection:
   ```
   sudo apt install fail2ban -y
   ```
