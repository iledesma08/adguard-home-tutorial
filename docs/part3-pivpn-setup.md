# Part 3: PiVPN Setup

## What is PiVPN?

PiVPN is the simplest way to setup and manage a VPN on your Raspberry Pi. It supports both WireGuard and OpenVPN protocols, allowing you to securely access your home network from anywhere in the world.

## Why PiVPN?

- **Secure Remote Access:** Access your home network securely from anywhere
- **Use AdGuard Home Remotely:** Block ads on your mobile devices when away from home
- **Privacy:** Encrypt your internet traffic when using public WiFi
- **Easy Management:** Simple command-line tools for managing VPN users

## Prerequisites

Before installing PiVPN:
- Raspberry Pi with a static IP address (completed in Part 1)
- AdGuard Home installed and working (completed in Part 2)
- Port forwarding configured on Fritz!Box (we'll do this below)

## Choosing Between WireGuard and OpenVPN

| Feature | WireGuard | OpenVPN |
|---------|-----------|---------|
| Speed | Faster | Slower |
| Battery Life | Better | Worse |
| Compatibility | Modern devices | Legacy support |
| Setup Complexity | Simpler | More complex |
| CPU Usage | Lower | Higher |

**Recommendation:** Use WireGuard for better performance and battery life. It's perfect for the Raspberry Pi Zero 2W.

## Installation

### 1. Run PiVPN Installer

```bash
curl -L https://install.pivpn.io | bash
```

### 2. Setup Wizard

The installer will guide you through several configuration screens:

#### Network Interface
- Select: **eth0** (Ethernet interface)
- Press Enter to continue

#### Static IP Confirmation
- Confirm your static IP address: `192.168.178.10`
- Select **Yes**

#### Local User
- Choose the user that will manage PiVPN
- Select your current user (the one you created during Raspberry Pi setup)

#### Choose VPN Protocol
- Select: **WireGuard**
- Press Enter

#### Default WireGuard Port
- Port: **51820** (default)
- You can change this if needed for security through obscurity
- Press Enter

#### DNS Provider
- Select: **Custom**
- Enter: `192.168.178.10` (Your AdGuard Home server)
- This ensures VPN clients use AdGuard Home for DNS

#### Public IP or DNS
This is important for connecting from outside your network.

**Option 1: Use Dynamic DNS (Recommended - covered in Part 5)**
- Select **DNS Entry**
- Enter your DDNS hostname (e.g., `your-home.duckdns.org`)
- We'll set this up in Part 5

**Option 2: Use Current Public IP**
- Select **Public IP**
- The installer will detect your current public IP
- Note: This will break if your ISP changes your public IP

#### Enable Unattended Upgrades
- Select: **Yes** (recommended for security)

#### Installation Summary
- Review the settings
- Select **Yes** to proceed with installation

### 3. Installation Complete

After installation completes, you'll see a summary message. The installer has:
- Installed WireGuard
- Configured firewall rules
- Created management scripts
- Set up the VPN service

Reboot your Raspberry Pi:
```bash
sudo reboot
```

## Configure Fritz!Box Port Forwarding

For external access to your VPN, you need to forward the WireGuard port through your Fritz!Box.

### 1. Access Fritz!Box

Navigate to `http://fritz.box` and login.

### 2. Configure Port Forwarding

1. Go to **Internet** → **Permit Access** → **Port Sharing**
2. Click **New Port Sharing**
3. Configure:
   - **Device:** Select your Raspberry Pi (`adguard-home` or `192.168.178.10`)
   - **Application:** Select **Other applications**
   - **Designation:** `WireGuard VPN`
   - **Protocol:** **UDP**
   - **Port to device:** `51820`
   - **to Port:** `51820`
   - **Enable:** ✓ Enable port sharing
4. Click **OK**

### 3. Verify Port Forwarding

From an external network (using mobile data), test if the port is open:
```bash
nc -zuv YOUR_PUBLIC_IP 51820
```

## Creating VPN Profiles

### Add Your First Client

```bash
pivpn add
```

You'll be prompted:
1. **Enter a name for the client:** e.g., `smartphone`, `laptop`, `tablet`
2. The script will generate the configuration

Example:
```bash
pivpn add
Enter a Name for the Client: smartphone
::: Client config generated at /home/username/configs/smartphone.conf
```

### List All Clients

```bash
pivpn list
```

Or for detailed information:
```bash
pivpn clients
```

## Connecting Clients

### Android/iOS

#### 1. Install WireGuard App

- **Android:** [WireGuard on Play Store](https://play.google.com/store/apps/details?id=com.wireguard.android)
- **iOS:** [WireGuard on App Store](https://apps.apple.com/us/app/wireguard/id1441195209)

#### 2. Transfer Configuration

**Method 1: QR Code (Easiest)**
```bash
pivpn -qr smartphone
```

Scan the QR code with the WireGuard app:
1. Open WireGuard app
2. Tap the **+** button
3. Select **Scan from QR code**
4. Scan the QR code displayed in your terminal

**Method 2: File Transfer**
```bash
# The config file is located at:
~/configs/smartphone.conf

# Transfer via SCP, email, or any secure method
```

#### 3. Activate the VPN

1. Open WireGuard app
2. Toggle the switch next to your profile
3. Allow VPN configuration when prompted

### Windows

#### 1. Install WireGuard

Download and install from: https://www.wireguard.com/install/

#### 2. Import Configuration

```bash
# Transfer the config file from Raspberry Pi
scp username@192.168.178.10:~/configs/laptop.conf .
```

1. Open WireGuard application
2. Click **Import tunnel(s) from file**
3. Select the `.conf` file
4. Click **Activate**

### macOS

#### 1. Install WireGuard

- Download from the Mac App Store
- Or install via Homebrew:
  ```bash
  brew install --cask wireguard-tools
  ```

#### 2. Import and Connect

Same process as Windows.

### Linux

#### 1. Install WireGuard

```bash
# Ubuntu/Debian
sudo apt install wireguard wireguard-tools

# Fedora
sudo dnf install wireguard-tools

# Arch Linux
sudo pacman -S wireguard-tools
```

#### 2. Setup Configuration

```bash
# Copy the config file
sudo cp laptop.conf /etc/wireguard/wg0.conf

# Start the VPN
sudo wg-quick up wg0

# Enable on boot (optional)
sudo systemctl enable wg-quick@wg0
```

## Testing the VPN Connection

### 1. Verify Connection

Once connected to the VPN:

**Check your IP address:**
```bash
curl ifconfig.me
```
You should see your home public IP address.

**Test DNS resolution through AdGuard Home:**
```bash
nslookup google.com
```
The server should be your AdGuard Home IP (`192.168.178.10`).

### 2. Test Ad Blocking

Visit ad-testing websites while connected to the VPN:
- https://ads-blocker.com/testing/
- https://d3ward.github.io/toolz/adblock.html

Ads should be blocked even when you're away from home!

### 3. Check AdGuard Home Dashboard

On your Raspberry Pi, check the AdGuard Home dashboard:
```
http://192.168.178.10:3000
```

You should see queries from your VPN client IP (usually `10.6.0.x` or similar).

## PiVPN Management Commands

### Essential Commands

```bash
# Add a new client
pivpn add

# List all clients
pivpn list
pivpn clients

# Show QR code for a client
pivpn -qr <client-name>

# Remove a client
pivpn remove

# Revoke a client (keeps config but blocks access)
pivpn revoke

# Show VPN statistics
pivpn -c

# Enable/Disable clients
pivpn enable
pivpn disable

# Backup all configurations
pivpn backup

# Update PiVPN
pivpn update

# Uninstall PiVPN
pivpn uninstall

# Debug connection issues
pivpn debug
```

### Check VPN Status

```bash
# Check if WireGuard is running
sudo wg show

# View active connections
sudo wg show all

# Check service status
sudo systemctl status wg-quick@wg0
```

## Split Tunneling (Optional)

By default, all traffic goes through the VPN. For split tunneling (only route specific traffic):

Edit the client configuration before importing:
```bash
nano ~/configs/smartphone.conf
```

Change the `AllowedIPs` line:
```conf
# Full tunnel (default):
AllowedIPs = 0.0.0.0/0, ::/0

# Split tunnel (only home network):
AllowedIPs = 192.168.178.0/24

# Split tunnel (only DNS through VPN):
AllowedIPs = 192.168.178.10/32
```

## Security Best Practices

1. **Use Strong Names:** Don't use obvious client names like "johns-phone"
2. **Regular Audits:** Periodically review and remove old clients
3. **Unique Configs:** Create separate configs for each device
4. **Secure Storage:** Store config files securely, they contain private keys
5. **Rotate Keys:** Regenerate configs periodically for enhanced security
6. **Monitor Connections:** Regularly check connected clients with `pivpn -c`

## Firewall Configuration

PiVPN should have configured UFW automatically, but verify:

```bash
sudo ufw status
```

You should see:
```
51820/udp                  ALLOW       Anywhere
```

If not, add it manually:
```bash
sudo ufw allow 51820/udp
```

## Performance Optimization

### 1. Monitor VPN Performance

```bash
# Check bandwidth usage
sudo iftop -i wg0

# Monitor connections
watch -n 1 'sudo wg show'
```

### 2. Adjust MTU (if experiencing issues)

If you experience connection issues or slow speeds, adjust MTU:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add or modify the MTU setting:
```conf
[Interface]
MTU = 1420
```

Common MTU values to try: 1420, 1400, 1380

Restart WireGuard:
```bash
sudo systemctl restart wg-quick@wg0
```

## Troubleshooting

### Can't Connect to VPN

1. **Check WireGuard is running:**
   ```bash
   sudo systemctl status wg-quick@wg0
   ```

2. **Verify port forwarding:**
   - Check Fritz!Box port forwarding settings
   - Ensure UDP port 51820 is forwarded to your Raspberry Pi

3. **Check firewall:**
   ```bash
   sudo ufw status
   ```

4. **Verify public IP/DDNS:**
   ```bash
   curl ifconfig.me
   ```

5. **Check client configuration:**
   - Ensure the Endpoint in your client config matches your public IP or DDNS hostname

### Connected but No Internet

1. **Check IP forwarding:**
   ```bash
   sysctl net.ipv4.ip_forward
   ```
   Should return `1`. If not:
   ```bash
   sudo sysctl -w net.ipv4.ip_forward=1
   ```

2. **Verify DNS settings:**
   - Ensure DNS in client config points to `192.168.178.10`
   - Test DNS: `nslookup google.com`

3. **Check routing:**
   ```bash
   sudo wg show
   ip route
   ```

### Slow VPN Speed

1. **Test base speed:**
   - Check your internet speed without VPN
   - Raspberry Pi Zero 2W can handle ~50-100 Mbps

2. **Adjust MTU:**
   - Try different MTU values (1420, 1400, 1380)

3. **Check CPU usage:**
   ```bash
   top
   ```

4. **Consider using OpenVPN instead:**
   - Some ISPs throttle UDP traffic
   - OpenVPN over TCP might work better

### DNS Not Working

1. **Verify AdGuard Home is running:**
   ```bash
   sudo systemctl status AdGuardHome
   ```

2. **Test DNS directly:**
   ```bash
   dig @192.168.178.10 google.com
   ```

3. **Check client DNS configuration:**
   - Open the WireGuard app
   - Verify DNS is set to `192.168.178.10`

## Next Steps

Proceed to [Part 4: Tailscale Integration](./part4-tailscale.md) to add an additional layer of connectivity and easier access.

## Additional Resources

- [PiVPN Official Documentation](https://docs.pivpn.io/)
- [WireGuard Official Site](https://www.wireguard.com/)
- [PiVPN GitHub Repository](https://github.com/pivpn/pivpn)
