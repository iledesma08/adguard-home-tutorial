# Part 4: Tailscale Integration

## What is Tailscale?

Tailscale creates a secure mesh network between your devices using WireGuard under the hood. Unlike traditional VPN that routes all traffic through your home, Tailscale provides:
- Direct peer-to-peer connections between devices
- Automatic NAT traversal (no port forwarding needed)
- Easy access to your Raspberry Pi from anywhere
- Integration with multiple devices across different networks

## Why Use Both PiVPN and Tailscale?

You might wonder why we need both. Here's the difference:

| Feature | PiVPN | Tailscale |
|---------|-------|-----------|
| **Purpose** | Route traffic through home network | Connect devices directly |
| **Ad Blocking** | Routes all traffic through AdGuard | Can route DNS through AdGuard |
| **Port Forwarding** | Required | Not required |
| **Public IP** | Required | Not required |
| **Use Case** | Full tunnel VPN for privacy | Easy device-to-device access |
| **Complexity** | Moderate | Simple |

**Use PiVPN when:** You want to route all your traffic through your home network (ad blocking, privacy on public WiFi)

**Use Tailscale when:** You want easy access to specific devices (SSH to your Pi, access home services)

## Installation

### 1. Install Tailscale on Raspberry Pi

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

The script will:
- Add Tailscale's package repository
- Install Tailscale
- Enable the service

### 2. Authenticate

Start Tailscale and authenticate:
```bash
sudo tailscale up
```

This will output a URL like:
```
To authenticate, visit:

https://login.tailscale.com/a/XXXXXXXXXXXX
```

1. Copy the URL and open it in your browser
2. Login with your Tailscale account (or create a free account)
3. Authorize the device
4. Give your device a name (e.g., "raspberry-pi-adguard")

### 3. Verify Connection

Check Tailscale status:
```bash
sudo tailscale status
```

You should see your device listed with its Tailscale IP (typically `100.x.x.x`).

Get your Tailscale IP:
```bash
tailscale ip -4
```

## Configure Tailscale as Subnet Router

To access your entire home network (not just the Raspberry Pi) through Tailscale:

### 1. Enable IP Forwarding (if not already enabled)

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

### 2. Advertise Subnet Routes

```bash
sudo tailscale up --advertise-routes=192.168.178.0/24 --accept-dns=false
```

This tells Tailscale to route traffic for your home network through this device.

### 3. Enable Subnet Routes in Tailscale Admin

1. Go to https://login.tailscale.com/admin/machines
2. Find your Raspberry Pi
3. Click the **⋮** menu → **Edit route settings**
4. Enable the subnet route `192.168.178.0/24`
5. Click **Save**

## Configure DNS Through Tailscale

To use AdGuard Home for all devices connected to your Tailscale network:

### 1. Advertise as DNS Server

```bash
sudo tailscale up --advertise-routes=192.168.178.0/24 --accept-dns=false --advertise-exit-node
```

### 2. Configure in Tailscale Admin

1. Go to https://login.tailscale.com/admin/dns
2. Add nameserver: Your Raspberry Pi's Tailscale IP (e.g., `100.x.x.x`)
3. Enable **Override local DNS**

Now all Tailscale devices can use your AdGuard Home for DNS!

## Install Tailscale on Client Devices

### Android/iOS

1. Install the Tailscale app:
   - **Android:** [Google Play Store](https://play.google.com/store/apps/details?id=com.tailscale.ipn)
   - **iOS:** [App Store](https://apps.apple.com/us/app/tailscale/id1470499037)

2. Open the app and sign in with the same account
3. Your devices will appear in the same network automatically

### Windows

1. Download installer from: https://tailscale.com/download/windows
2. Run the installer
3. Sign in with your Tailscale account
4. You're connected!

### macOS

1. Download from: https://tailscale.com/download/mac
2. Install and open Tailscale
3. Sign in with your Tailscale account

### Linux

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Follow the authentication URL in your browser.

## Access AdGuard Home Through Tailscale

### Option 1: Use Tailscale IP

From any device connected to Tailscale:
```
http://100.x.x.x:3000
```

Replace `100.x.x.x` with your Raspberry Pi's Tailscale IP.

### Option 2: Use MagicDNS

Tailscale provides MagicDNS for easy access:

1. Enable MagicDNS in Tailscale admin:
   - Go to https://login.tailscale.com/admin/dns
   - Enable **MagicDNS**

2. Access your Pi using its hostname:
   ```
   http://raspberry-pi-adguard:3000
   ```

## SSH Access Through Tailscale

### From Any Device on Tailscale

```bash
# Using Tailscale IP
ssh username@100.x.x.x

# Using MagicDNS
ssh username@raspberry-pi-adguard
```

This works from anywhere, even without port forwarding!

## Tailscale + AdGuard Home Integration

### Method 1: Use AdGuard Home for DNS Globally

Configure all Tailscale devices to use AdGuard Home:

1. **On Raspberry Pi:**
   ```bash
   sudo tailscale up --advertise-routes=192.168.178.0/24 --accept-dns=false
   ```

2. **In Tailscale Admin Console:**
   - Go to https://login.tailscale.com/admin/dns
   - Under **Global nameservers**, add:
     - `100.x.x.x` (Your Raspberry Pi's Tailscale IP)
   - Enable **Override local DNS**

3. **On Each Client:**
   - Ensure **Use Tailscale DNS** is enabled in the Tailscale app settings

Now all Tailscale devices will use AdGuard Home for DNS, blocking ads everywhere!

### Method 2: Split DNS (Advanced)

Use AdGuard Home only for specific domains:

1. **In Tailscale Admin:**
   - Go to DNS settings
   - Add **Split DNS** configuration
   - Route specific domains through your Tailscale DNS

Example:
```
*.home → 100.x.x.x
*.local → 100.x.x.x
```

## Exit Node Configuration (Optional)

Turn your Raspberry Pi into a Tailscale exit node to route all traffic through your home:

### 1. Configure as Exit Node

```bash
sudo tailscale up --advertise-exit-node --advertise-routes=192.168.178.0/24 --accept-dns=false
```

### 2. Approve in Tailscale Admin

1. Go to https://login.tailscale.com/admin/machines
2. Find your Raspberry Pi
3. Click **⋮** → **Edit route settings**
4. Enable **Use as exit node**

### 3. Use Exit Node on Clients

On your client device:
```bash
# Linux/macOS
sudo tailscale up --exit-node=raspberry-pi-adguard

# Or use the Tailscale app on mobile/desktop to select exit node
```

**Note:** Using the Pi Zero 2W as an exit node works but may be slow due to CPU/network limitations.

## Combining PiVPN and Tailscale

You can run both simultaneously:

- **Use PiVPN** for friends/family who don't want to create a Tailscale account
- **Use Tailscale** for your personal devices for easier management

Both can coexist without conflicts as they use different WireGuard interfaces.

## ACL Configuration (Advanced)

Control which devices can talk to each other:

### 1. Edit ACLs in Tailscale Admin

Go to https://login.tailscale.com/admin/acls

Example ACL to restrict access:
```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["*"],
      "dst": ["raspberry-pi-adguard:*"]
    },
    {
      "action": "accept",
      "src": ["raspberry-pi-adguard"],
      "dst": ["*:*"]
    }
  ]
}
```

This allows:
- All devices to access the Raspberry Pi
- Raspberry Pi to access all devices
- Prevents peer-to-peer connections between other devices

## Monitoring and Management

### Check Tailscale Status

```bash
# View connection status
sudo tailscale status

# View detailed network info
sudo tailscale netcheck

# View logs
sudo journalctl -u tailscaled -f

# Ping another device
tailscale ping <device-name-or-ip>
```

### Tailscale Web Interface

Access the admin console at: https://login.tailscale.com/admin

From here you can:
- View all connected devices
- Manage ACLs
- Configure DNS settings
- View connection logs
- Revoke device access

## Performance Considerations

### Raspberry Pi Zero 2W Limitations

- **CPU:** Quad-core 1GHz (limited processing power)
- **Network:** 100Mbps Ethernet (via USB)
- **Use Cases:**
  - ✅ SSH access
  - ✅ DNS queries to AdGuard Home
  - ✅ Accessing home network services
  - ✅ Light file transfers
  - ⚠️ Exit node (works but may be slow)
  - ❌ High-bandwidth streaming

### Optimization Tips

1. **Use Direct Connections:**
   - Tailscale tries to establish direct peer-to-peer connections
   - These are faster than relaying through Tailscale's servers

2. **Check Connection Type:**
   ```bash
   tailscale status
   ```
   Look for "direct" vs "relay" in the output

3. **Disable Exit Node if Not Needed:**
   - Only enable exit node when necessary
   - It adds extra processing overhead

## Security Best Practices

1. **Use ACLs:**
   - Restrict access between devices
   - Follow principle of least privilege

2. **Keep Tailscale Updated:**
   ```bash
   sudo apt update && sudo apt upgrade tailscale
   ```

3. **Monitor Connected Devices:**
   - Regularly review devices in Tailscale admin
   - Remove devices you no longer use

4. **Enable Two-Factor Authentication:**
   - Secure your Tailscale account with 2FA
   - Go to Account Settings in Tailscale admin

5. **Separate Admin Account:**
   - Consider using a separate Tailscale account for admin access

## Troubleshooting

### Tailscale Won't Start

```bash
# Check service status
sudo systemctl status tailscaled

# Restart service
sudo systemctl restart tailscaled

# Check logs
sudo journalctl -u tailscaled -n 50
```

### Can't Connect to Other Devices

1. **Check Tailscale status:**
   ```bash
   sudo tailscale status
   ```

2. **Verify both devices are authenticated:**
   - Check https://login.tailscale.com/admin/machines

3. **Check ACLs:**
   - Ensure ACLs aren't blocking connections

4. **Test connectivity:**
   ```bash
   tailscale ping <other-device>
   ```

### Subnet Routes Not Working

1. **Verify IP forwarding is enabled:**
   ```bash
   sysctl net.ipv4.ip_forward
   ```

2. **Check routes are advertised:**
   ```bash
   sudo tailscale status
   ```

3. **Verify routes are approved:**
   - Check Tailscale admin console
   - Ensure subnet routes are enabled

4. **Test routing:**
   ```bash
   # From a Tailscale device, try to ping another device on your home network
   ping 192.168.178.1
   ```

### DNS Through Tailscale Not Working

1. **Check DNS configuration:**
   ```bash
   sudo tailscale status
   ```

2. **Verify AdGuard Home is accessible:**
   ```bash
   dig @100.x.x.x google.com
   ```

3. **Check Tailscale DNS settings:**
   - Go to https://login.tailscale.com/admin/dns
   - Verify your Pi is listed as a nameserver

4. **On clients, ensure "Accept DNS" is enabled:**
   - Check Tailscale app settings

## Next Steps

Proceed to [Part 5: DoH/DoT/DDNS Configuration](./part5-doh-dot-ddns.md) to enable encrypted DNS and dynamic DNS.

## Additional Resources

- [Tailscale Official Documentation](https://tailscale.com/kb/)
- [Tailscale GitHub](https://github.com/tailscale/tailscale)
- [Subnet Routers Guide](https://tailscale.com/kb/1019/subnets/)
- [Exit Nodes Guide](https://tailscale.com/kb/1103/exit-nodes/)
