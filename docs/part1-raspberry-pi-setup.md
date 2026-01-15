# Part 1: Raspberry Pi Zero 2W Setup

## Hardware Requirements

- Raspberry Pi Zero 2W
- MicroSD card (16GB or larger, Class 10 recommended)
- Power supply (5V, 2.5A minimum)
- Ethernet adapter (USB to Ethernet)
- Case (optional but recommended for cooling)

## Initial Setup

### 1. Download Raspberry Pi OS

Download the latest Raspberry Pi OS Lite (64-bit) from the official website:
```bash
https://www.raspberrypi.com/software/operating-systems/
```

**Recommendation:** Use Raspberry Pi OS Lite as it's lightweight and perfect for headless servers.

### 2. Flash the SD Card

Use Raspberry Pi Imager or balenaEtcher to flash the OS to your microSD card.

**Using Raspberry Pi Imager (Recommended):**
1. Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Select "Raspberry Pi OS Lite (64-bit)"
3. Choose your SD card
4. Click on the gear icon (⚙️) to configure advanced options:
   - Set hostname: `adguard-home`
   - Enable SSH with password authentication
   - Set username and password
   - Configure WiFi (optional, we'll use Ethernet)
   - Set locale settings
5. Write the image to the SD card

### 3. Initial Boot and Connection

1. Insert the microSD card into the Raspberry Pi Zero 2W
2. Connect the USB Ethernet adapter
3. Connect the Ethernet cable to your Fritz!Box
4. Power on the Raspberry Pi

### 4. Find the Raspberry Pi IP Address

**Option 1: Check Fritz!Box web interface**
- Access your Fritz!Box at `http://fritz.box`
- Navigate to Home Network → Network → Network Connections
- Look for a device named `adguard-home` or similar

**Option 2: Use nmap (from your computer)**
```bash
nmap -sn 192.168.178.0/24
```
Replace the IP range with your Fritz!Box network range.

### 5. SSH into the Raspberry Pi

```bash
ssh username@<raspberry-pi-ip>
```

Enter your password when prompted.

### 6. Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

### 7. Configure Static IP Address

Edit the dhcpcd configuration:
```bash
sudo nano /etc/dhcpcd.conf
```

Add the following at the end (adjust values for your network):
```bash
interface eth0
static ip_address=192.168.178.10/24
static routers=192.168.178.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

Restart the networking service:
```bash
sudo systemctl restart dhcpcd
```

### 8. Configure Hostname (Optional)

```bash
sudo hostnamectl set-hostname adguard-home
```

Edit `/etc/hosts`:
```bash
sudo nano /etc/hosts
```

Change the line with `127.0.1.1` to:
```
127.0.1.1       adguard-home
```

### 9. Enable and Configure UFW Firewall

```bash
sudo apt install ufw -y
sudo ufw allow 22/tcp     # SSH
sudo ufw allow 53/tcp     # DNS
sudo ufw allow 53/udp     # DNS
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS
sudo ufw allow 853/tcp    # DNS over TLS
sudo ufw allow 3000/tcp   # AdGuard Home Web Interface
sudo ufw enable
```

### 10. Set Timezone

```bash
sudo timedatectl set-timezone Europe/Berlin
```

### 11. Reboot

```bash
sudo reboot
```

## Verification

After reboot, SSH back into your Raspberry Pi using the static IP:
```bash
ssh username@192.168.178.10
```

Check that everything is working:
```bash
# Check network configuration
ip addr show eth0

# Check system time
timedatectl

# Check firewall status
sudo ufw status
```

## Next Steps

Proceed to [Part 2: AdGuard Home Installation](./part2-adguard-home.md) to install and configure AdGuard Home.

## Troubleshooting

### Can't SSH into the Raspberry Pi
- Verify the Ethernet connection is working
- Check if the Ethernet adapter LED is blinking
- Verify the IP address from Fritz!Box interface
- Try connecting a keyboard and monitor to debug directly

### Static IP not working
- Verify the IP range matches your Fritz!Box network
- Ensure the static IP is outside the DHCP range of Fritz!Box
- Check for typos in `/etc/dhcpcd.conf`
- Verify the interface name with `ip addr` (should be `eth0` for Ethernet)

### System is slow
- Ensure you're using a fast microSD card (Class 10 or better)
- Consider using an SSD via USB adapter for better performance
- Check CPU temperature: `vcgencmd measure_temp`
