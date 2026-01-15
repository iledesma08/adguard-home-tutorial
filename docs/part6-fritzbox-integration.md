# Part 6: Fritz!Box Integration and Optimization

## Overview

The Fritz!Box is a popular router in Germany and Europe. This section covers advanced integration between your Raspberry Pi setup and the Fritz!Box, including optimization, monitoring, and best practices.

## Fritz!Box Models

This guide is compatible with most Fritz!Box models, including:
- Fritz!Box 7590
- Fritz!Box 7530
- Fritz!Box 7490
- Fritz!Box 6850 LTE/5G
- Other models with similar firmware

## Basic Integration

### 1. Access Fritz!Box Interface

Default address:
```
http://fritz.box
or
http://192.168.178.1
```

Login with your Fritz!Box password.

### 2. Verify Raspberry Pi Connection

1. Go to **Home Network** → **Network** → **Network Connections**
2. Find your Raspberry Pi (should show as `adguard-home` or similar)
3. Verify it has the static IP `192.168.178.10`
4. Check connection type (should be Ethernet/LAN)

### 3. Set DNS to AdGuard Home

**Method 1: Router-wide DNS (Recommended)**

1. Go to **Home Network** → **Network** → **Network Settings**
2. Click on **IPv4 Addresses** tab
3. Under **IPv4 Settings**:
   - **Local DNS Server:** Change from "Use the DNS server provided by the internet service provider" to "Use custom DNS server"
   - **Preferred DNS server:** `192.168.178.10`
   - **Alternative DNS server:** `1.1.1.1` (fallback)
4. Click **Apply**

Now all devices connected to Fritz!Box will use AdGuard Home.

**Method 2: Using Fritz!Box as DNS Relay**

Some Fritz!Box models allow configuring upstream DNS:
1. Go to **Internet** → **Account Information** → **DNS Server**
2. Enable **Use custom DNS servers**
3. **Primary DNS:** `192.168.178.10`
4. **Secondary DNS:** `1.1.1.1`

### 4. Configure DHCP Settings

Ensure DHCP doesn't conflict with your static IP:

1. Go to **Home Network** → **Network** → **Network Settings**
2. Under **IPv4 Addresses**:
   - **DHCP Range:** Ensure it doesn't include `192.168.178.10`
   - Example: `192.168.178.20` to `192.168.178.200`

## Port Forwarding Setup

### Required Ports

For complete functionality, forward these ports:

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| DNS | 53 | TCP/UDP | DNS queries (if external) |
| AdGuard Web | 3000 | TCP | Web interface (optional) |
| HTTPS/DoH | 443 | TCP | DNS over HTTPS |
| DoT | 853 | TCP | DNS over TLS |
| WireGuard | 51820 | UDP | PiVPN |

### Configure Port Forwarding

1. Go to **Internet** → **Permit Access** → **Port Sharing**
2. Click **New Port Sharing**

**For WireGuard VPN:**
- Device: `192.168.178.10` (Raspberry Pi)
- Application: Other applications
- Designation: `WireGuard VPN`
- Protocol: UDP
- Port to device: `51820`
- to Port: `51820`

**For DNS over HTTPS:**
- Device: `192.168.178.10`
- Application: HTTPS
- Protocol: TCP
- Port to device: `443`
- to Port: `443`

**For DNS over TLS:**
- Device: `192.168.178.10`
- Application: Other applications
- Designation: `DNS over TLS`
- Protocol: TCP
- Port to device: `853`
- to Port: `853`

**Security Note:** Only forward ports you actually need. Don't forward the web interface port (3000) externally unless necessary.

## IPv6 Configuration

### Enable IPv6 on Fritz!Box

1. Go to **Internet** → **Account Information** → **IPv6**
2. Enable **IPv6 support active**
3. Note your IPv6 prefix

### Configure IPv6 on Raspberry Pi

If Fritz!Box provides IPv6:

```bash
# Check current IPv6 address
ip -6 addr show eth0

# If no IPv6, enable it
sudo nano /etc/dhcpcd.conf
```

Add:
```bash
# Enable IPv6
noipv6rs
denyinterfaces eth0
```

For static IPv6:
```bash
interface eth0
static ip6_address=YOUR_IPV6_PREFIX::10/64
static routers=YOUR_IPV6_GATEWAY
```

Restart networking:
```bash
sudo systemctl restart dhcpcd
```

### Configure IPv6 DNS in AdGuard Home

Edit AdGuard Home config:
```bash
sudo nano /opt/AdGuardHome/AdGuardHome.yaml
```

Update DNS settings:
```yaml
dns:
  bind_hosts:
    - 0.0.0.0
    - '::'  # Enable IPv6 binding
```

Restart AdGuard Home:
```bash
sudo /opt/AdGuardHome/AdGuardHome -s restart
```

## Fritz!Box DNS Rebind Protection

Fritz!Box has DNS rebind protection that might block local DNS queries.

### Disable for AdGuard Home

1. Go to **Home Network** → **Network** → **Network Settings**
2. Click **Advanced Settings**
3. Disable **DNS Rebind Protection** for your local domain
4. Or add an exception for `*.local` and `*.lan`

## Guest Network Configuration

### Setup Guest Network

1. Go to **WLAN** → **Guest Access**
2. Enable **Guest access active**
3. Configure guest network settings

### Separate DNS for Guest Network

**Option 1: Use AdGuard Home (with restrictions)**

Configure AdGuard Home to allow guest network:
```bash
sudo nano /opt/AdGuardHome/AdGuardHome.yaml
```

Update allowed_clients:
```yaml
dns:
  allowed_clients:
    - 192.168.178.0/24    # Main network
    - 192.168.179.0/24    # Guest network (check your Fritz!Box guest network range)
```

**Option 2: Direct DNS for Guests**

1. In Fritz!Box guest network settings
2. Set DNS servers to public DNS:
   - Primary: `1.1.1.1`
   - Secondary: `8.8.8.8`

This prevents guests from using your AdGuard Home.

## Fritz!Box MyFRITZ! Integration

MyFRITZ! is Fritz!Box's built-in DDNS service.

### Setup MyFRITZ!

1. Go to **Internet** → **MyFRITZ! Account**
2. Create or login to your MyFRITZ! account
3. Note your MyFRITZ! address: `https://[hash].myfritz.net`

### Use MyFRITZ! for VPN

Update PiVPN to use MyFRITZ! address:

```bash
# Edit WireGuard config
sudo nano /etc/wireguard/wg0.conf
```

Update Endpoint:
```conf
[Peer]
Endpoint = [hash].myfritz.net:51820
```

Regenerate client configs:
```bash
pivpn add
```

### MyFRITZ! vs DuckDNS

| Feature | MyFRITZ! | DuckDNS |
|---------|----------|---------|
| **Setup** | Automatic | Manual |
| **Updates** | Automatic | Cron job needed |
| **Reliability** | High | High |
| **SSL Certs** | Limited | Full Let's Encrypt |
| **Flexibility** | Fritz!Box only | Works anywhere |

**Recommendation:** Use DuckDNS for SSL certificates, but MyFRITZ! works well for VPN endpoints.

## Traffic Monitoring

### View Network Traffic in Fritz!Box

1. Go to **Home Network** → **Network** → **Network Connections**
2. Click on your Raspberry Pi
3. View traffic statistics

### Monitor DNS Queries

In AdGuard Home dashboard:
```
http://192.168.178.10:3000
```

View:
- Total queries
- Queries per client
- Top blocked domains
- Query types

### Advanced Monitoring

Install additional monitoring on Raspberry Pi:

```bash
# Install vnStat for bandwidth monitoring
sudo apt install vnstat -y
sudo systemctl enable vnstat
sudo systemctl start vnstat

# View stats
vnstat -i eth0

# Real-time monitoring
vnstat -i eth0 -l
```

## Parental Controls

### Fritz!Box Parental Controls

Fritz!Box has built-in parental controls that work alongside AdGuard Home.

1. Go to **Internet** → **Filters** → **Parental Controls**
2. Create profiles for different family members
3. Assign devices to profiles
4. Set time restrictions and content filters

### AdGuard Home Parental Controls

In AdGuard Home:
1. Go to **Settings** → **General Settings**
2. Enable **Use AdGuard browsing security web service**
3. Enable **Use AdGuard parental control web service**
4. Enable **Safe Search** (forces safe search on search engines)

### Client-Specific Filtering

Create different filtering rules for different clients:

1. Go to **Settings** → **Client Settings**
2. Add a client (by IP or MAC address)
3. Configure specific settings:
   - Different upstream DNS
   - Different filtering rules
   - Custom blocked services

Example use cases:
- Kids' devices: Strict filtering, blocked services (social media)
- Guest devices: Basic ad blocking only
- Personal devices: Full ad blocking, no restrictions

## Performance Optimization

### Fritz!Box Settings

#### 1. Disable Unused Features

Go through Fritz!Box and disable features you don't use:
- UPnP (if not needed)
- IPv6 (if not using it)
- DECT (if no DECT devices)
- Answering machine
- Fax

#### 2. Optimize WLAN

1. Go to **WLAN** → **Radio Channel**
2. Use **Set radio channel automatically**
3. Or manually select least congested channels
4. Enable **802.11k/v** for better roaming

#### 3. Update Firmware

1. Go to **System** → **Update**
2. Check for updates
3. Enable automatic updates

### Network Optimization

#### 1. QoS Settings

Prioritize important traffic:

1. Go to **Internet** → **Filters** → **Prioritization**
2. Enable real-time prioritization
3. Add custom rules for important services

#### 2. DNS Caching

Fritz!Box caches DNS queries. To optimize:

1. Set DNS cache size (if available in your model)
2. Consider disabling Fritz!Box DNS cache if using AdGuard Home
3. Monitor DNS query times in AdGuard Home

#### 3. Connection Settings

1. Go to **Internet** → **Account Information**
2. Verify connection type and settings
3. For DSL: Check line quality and SNR margins
4. For Cable: Check signal levels

## Backup and Restore

### Backup Fritz!Box Settings

1. Go to **System** → **Backup**
2. Click **Back Up**
3. Save the exported file securely
4. Store password-protected

### Backup AdGuard Home Settings

```bash
# Backup AdGuard Home config
sudo cp /opt/AdGuardHome/AdGuardHome.yaml ~/adguard-backup-$(date +%Y%m%d).yaml

# Backup query logs and stats (optional)
sudo tar -czf ~/adguard-data-backup-$(date +%Y%m%d).tar.gz /opt/AdGuardHome/data/
```

### Restore Process

**Fritz!Box:**
1. Go to **System** → **Backup**
2. Click **Restore**
3. Select your backup file

**AdGuard Home:**
```bash
sudo /opt/AdGuardHome/AdGuardHome -s stop
sudo cp ~/adguard-backup-YYYYMMDD.yaml /opt/AdGuardHome/AdGuardHome.yaml
sudo /opt/AdGuardHome/AdGuardHome -s start
```

## Security Hardening

### Fritz!Box Security

#### 1. Change Default Password

1. Go to **System** → **Fritz!Box Users**
2. Change the default admin password
3. Use a strong password (20+ characters)

#### 2. Disable Remote Access (if not needed)

1. Go to **Internet** → **Permit Access** → **FRITZ!Box Services**
2. Disable **Internet access to the FRITZ!Box over HTTPS enabled**
3. Only enable if you need remote management

#### 3. Enable HTTPS Only

1. Go to **System** → **Push Service**
2. Enable **HTTPS Only** for web interface

#### 4. Review Permitted Access

1. Go to **Internet** → **Permit Access**
2. Review all port forwarding rules
3. Remove any unused rules
4. Document what each rule is for

#### 5. Enable Firewall

Fritz!Box firewall should be enabled by default:
1. Go to **Internet** → **Filters** → **Lists**
2. Verify firewall is active
3. Review blocked attempts

### Network Segmentation

Consider using VLANs (if your Fritz!Box supports it):

1. **Management VLAN:** Router, Raspberry Pi, NAS
2. **Main VLAN:** Trusted devices (computers, phones)
3. **IoT VLAN:** Smart home devices
4. **Guest VLAN:** Guest devices

This limits damage if one device is compromised.

## Troubleshooting

### Fritz!Box Can't Reach Raspberry Pi

1. **Check physical connection:**
   ```bash
   ethtool eth0
   ```
   Should show "Link detected: yes"

2. **Check Fritz!Box port:**
   - Try different LAN port on Fritz!Box
   - Check port LED is lit

3. **Check IP configuration:**
   ```bash
   ip addr show eth0
   ```

4. **Ping Fritz!Box:**
   ```bash
   ping 192.168.178.1
   ```

### DNS Not Working Through Fritz!Box

1. **Verify Fritz!Box DNS settings:**
   - Check that AdGuard Home IP is correctly set

2. **Test DNS directly:**
   ```bash
   nslookup google.com 192.168.178.10
   ```

3. **Check if AdGuard Home is accessible from Fritz!Box:**
   - From a device on the network, try accessing AdGuard Home

4. **Verify firewall rules:**
   ```bash
   sudo ufw status
   ```

### Port Forwarding Not Working

1. **Test internally first:**
   - Verify service works on local network

2. **Check Fritz!Box public IP:**
   ```bash
   curl ifconfig.me
   ```

3. **Test port from external network:**
   - Use mobile data or online port checker

4. **Verify DS-Lite:**
   - Some ISPs use DS-Lite (IPv6 transition)
   - DS-Lite doesn't support IPv4 port forwarding
   - Check: **Internet** → **Account Information** → **Connection**
   - If DS-Lite, you'll need to request a real IPv4 address from your ISP

### High CPU on Raspberry Pi

1. **Check what's using CPU:**
   ```bash
   top
   htop
   ```

2. **Reduce AdGuard Home load:**
   - Reduce query log retention
   - Disable unnecessary filters
   - Optimize cache settings

3. **Check for DNS loops:**
   - Ensure Fritz!Box isn't forwarding queries back to itself

4. **Monitor temperature:**
   ```bash
   vcgencmd measure_temp
   ```
   Should stay below 70°C

## Best Practices

1. **Regular Updates:**
   - Keep Fritz!Box firmware updated
   - Keep Raspberry Pi updated
   - Keep AdGuard Home updated

2. **Monitoring:**
   - Check AdGuard Home dashboard weekly
   - Review Fritz!Box logs monthly
   - Monitor network performance

3. **Documentation:**
   - Document all port forwarding rules
   - Keep backup of all configurations
   - Note any custom settings

4. **Testing:**
   - Test VPN connectivity monthly
   - Verify DNS resolution works
   - Check SSL certificates validity

5. **Security:**
   - Review access logs
   - Update passwords regularly
   - Audit connected devices

## Advanced Topics

### UPnP and NAT-PMP

If you want automatic port forwarding:

1. Go to **Home Network** → **Network** → **Network Settings**
2. Under **UPnP Settings**, enable **Permit access for configured applications only**
3. Configure specific applications

**Security Note:** UPnP can be a security risk. Manual port forwarding is more secure.

### Fritz!Box API

Fritz!Box has a TR-064 API for automation:

```bash
# Install fritzconnection
pip3 install fritzconnection

# Example: Get connection status
python3 -c "from fritzconnection import FritzConnection; fc = FritzConnection(address='192.168.178.1'); print(fc.call_action('WANIPConn1', 'GetStatusInfo'))"
```

### Integration with Home Assistant

If using Home Assistant, integrate Fritz!Box:

```yaml
# configuration.yaml
fritz:
  devices:
    - host: 192.168.178.1
      username: your-username
      password: your-password
```

## Conclusion

Your Fritz!Box and Raspberry Pi are now fully integrated with:
- Network-wide ad blocking
- Secure VPN access
- Encrypted DNS
- Dynamic DNS
- Optimized performance

## Additional Resources

- [Fritz!Box Knowledge Base (German)](https://avm.de/service/wissensdatenbank/)
- [Fritz!Box Knowledge Base (English)](https://en.avm.de/service/knowledge-base/)
- [Fritz!Box API Documentation](https://avm.de/service/schnittstellen/)
- [IPFire Fritz!Box Guide](https://wiki.ipfire.org/configuration/network/fritzbox)

---

**Congratulations!** You've completed the full tutorial. Your Raspberry Pi Zero 2W is now a powerful network security and privacy hub!
