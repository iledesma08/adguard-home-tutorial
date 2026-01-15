# Part 2: AdGuard Home Installation and Configuration

## What is AdGuard Home?

AdGuard Home is a network-wide software for blocking ads and tracking. It operates as a DNS server that re-routes tracking domains to a "black hole," preventing your devices from connecting to those servers.

## Installation

### Method 1: Automated Installation (Recommended)

```bash
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

The installer will:
- Download the latest version
- Install AdGuard Home as a system service
- Start the service automatically

### Method 2: Manual Installation

```bash
# Download the latest release
cd /tmp
wget https://static.adguard.com/adguardhome/release/AdGuardHome_linux_arm64.tar.gz

# Extract the archive
tar -xvf AdGuardHome_linux_arm64.tar.gz

# Move to /opt
sudo mv AdGuardHome /opt/

# Install as a service
cd /opt/AdGuardHome
sudo ./AdGuardHome -s install

# Start the service
sudo ./AdGuardHome -s start
```

## Initial Configuration

### 1. Access the Web Interface

Open your browser and navigate to:
```
http://192.168.178.10:3000
```

You'll be greeted by the initial setup wizard.

### 2. Setup Wizard Steps

**Step 1: Welcome Screen**
- Click "Get Started"

**Step 2: Admin Interface Configuration**
- Admin Web Interface: Leave as `0.0.0.0:3000` (or change to port 80)
- Click "Next"

**Step 3: DNS Settings**
- DNS Server: Leave as `0.0.0.0:53`
- Click "Next"

**Step 4: Create Admin Account**
- Username: Choose a secure username
- Password: Use a strong password (at least 12 characters)
- Click "Next"

**Step 5: Configure Devices**
- Follow the on-screen instructions or skip for now
- Click "Next"

**Step 6: Completion**
- Click "Open Dashboard"

### 3. Login to Dashboard

Login with your admin credentials at:
```
http://192.168.178.10:3000
```

## DNS Configuration

### 1. Configure Upstream DNS Servers

Navigate to **Settings** → **DNS Settings** → **Upstream DNS servers**

Add the following DNS servers (one per line):

**For DNS over HTTPS (DoH):**
```
https://dns.cloudflare.com/dns-query
https://dns.google/dns-query
https://dns.quad9.net/dns-query
```

**For DNS over TLS (DoT):**
```
tls://1.1.1.1
tls://1.0.0.1
tls://dns.quad9.net
```

**Bootstrap DNS servers:**
```
1.1.1.1
1.0.0.1
8.8.8.8
9.9.9.9
```

### 2. Configure DNS Server Settings

In the same **DNS Settings** page:

- **DNS cache configuration:**
  - Enable "Optimistic cache"
  - Cache size: 4 MB (default is fine)

- **Rate limiting:**
  - Set to 20 requests per second (adjust based on your needs)

- **DNSSEC:**
  - Enable "Enable DNSSEC" for additional security

- **Enable EDNSClientSubnet:**
  - Keep disabled for privacy

### 3. Enable Query Logging

Navigate to **Settings** → **Query Log Settings**:
- Enable "Enable log"
- Set retention period: 90 days (or your preference)
- Enable "Anonymize client IP"

## Filters Configuration

### 1. Enable Default Filters

Navigate to **Filters** → **DNS blocklists**:

The following filters should be enabled by default:
- AdGuard DNS filter
- AdAway Default Blocklist

### 2. Add Additional Filters

Click **"Add blocklist"** → **"Add a custom list"**

Recommended filters:
```
# StevenBlack's Unified Hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts

# Peter Lowe's List
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=adblockplus&showintro=0&mimetype=plaintext

# EasyList
https://easylist.to/easylist/easylist.txt

# NoCoin Filter List (Cryptocurrency mining protection)
https://raw.githubusercontent.com/hoshsadiq/adblock-nocoin-list/master/hosts.txt

# Phishing Army
https://phishing.army/download/phishing_army_blocklist.txt
```

### 3. Update Filters

Click **"Update filters"** to download and apply all blocklists.

## Client Configuration

### Configure Fritz!Box to Use AdGuard Home

#### Option 1: Set as Primary DNS (Recommended)

1. Access Fritz!Box web interface at `http://fritz.box`
2. Navigate to **Home Network** → **Network** → **Network Settings**
3. Click on **IPv4 Addresses** tab
4. Under **DNS Server**:
   - DNS Server 1: `192.168.178.10` (Your AdGuard Home IP)
   - DNS Server 2: `1.1.1.1` (Fallback)
5. Click **Apply**

All devices connected to your Fritz!Box will now use AdGuard Home for DNS.

#### Option 2: Configure per Device

Configure DNS manually on each device:
- Primary DNS: `192.168.178.10`
- Secondary DNS: `1.1.1.1`

## Testing AdGuard Home

### 1. Test DNS Resolution

From any device on your network:
```bash
nslookup google.com 192.168.178.10
```

You should see a response from your AdGuard Home server.

### 2. Test Ad Blocking

Visit these test pages:
- https://ads-blocker.com/testing/
- https://d3ward.github.io/toolz/adblock.html

You should see that most ads are blocked.

### 3. Check the Dashboard

Visit your AdGuard Home dashboard:
```
http://192.168.178.10:3000
```

You should see:
- DNS queries being logged
- Blocked queries statistics
- Top blocked domains
- Top clients

## Configuration Files

### Backup Configuration

The main configuration file is located at:
```
/opt/AdGuardHome/AdGuardHome.yaml
```

To backup:
```bash
sudo cp /opt/AdGuardHome/AdGuardHome.yaml ~/AdGuardHome-backup.yaml
```

### Using Provided Configs

This repository includes two configuration files:

**1. Basic Configuration (`AdGuardHome-basic.yaml`):**
- Standard DNS settings
- No TLS/encryption
- Basic filters
- Good for initial setup

**2. Advanced Configuration (`AdGuardHome-advanced.yaml`):**
- DNS over HTTPS (DoH) and DNS over TLS (DoT) enabled
- Additional filters
- Enhanced security settings
- Requires SSL certificates

To use a provided configuration:
```bash
# Stop AdGuard Home
sudo /opt/AdGuardHome/AdGuardHome -s stop

# Backup current config
sudo cp /opt/AdGuardHome/AdGuardHome.yaml /opt/AdGuardHome/AdGuardHome.yaml.bak

# Copy the new config
sudo cp ~/adguard-home-tutorial/configs/AdGuardHome-basic.yaml /opt/AdGuardHome/AdGuardHome.yaml

# IMPORTANT: Edit the config to set your password
sudo nano /opt/AdGuardHome/AdGuardHome.yaml
# Change the password hash in the 'users' section

# Start AdGuard Home
sudo /opt/AdGuardHome/AdGuardHome -s start
```

## Managing AdGuard Home Service

```bash
# Check status
sudo /opt/AdGuardHome/AdGuardHome -s status

# Start service
sudo /opt/AdGuardHome/AdGuardHome -s start

# Stop service
sudo /opt/AdGuardHome/AdGuardHome -s stop

# Restart service
sudo /opt/AdGuardHome/AdGuardHome -s restart

# View logs
sudo journalctl -u AdGuardHome -f
```

## Performance Optimization for Raspberry Pi Zero 2W

### 1. Reduce Log Verbosity

Edit `/opt/AdGuardHome/AdGuardHome.yaml`:
```yaml
verbose: false
log_compress: true
log_max_size: 50
```

### 2. Optimize Query Log

In the web interface:
- Settings → Query Log Settings
- Reduce retention to 24-48 hours for limited RAM devices

### 3. Monitor Resources

```bash
# Check memory usage
free -h

# Check CPU usage
top

# Check AdGuard Home memory
ps aux | grep AdGuardHome
```

## Security Best Practices

1. **Change Default Port:**
   - Change web interface from port 3000 to a non-standard port

2. **Use Strong Password:**
   - Minimum 12 characters
   - Mix of letters, numbers, and symbols

3. **Enable Authentication:**
   - Always keep authentication enabled

4. **Regular Updates:**
   ```bash
   # Check for updates in the web interface
   # Settings → General Settings → Update
   ```

5. **Firewall Rules:**
   - Keep UFW enabled
   - Only open necessary ports

## Troubleshooting

### AdGuard Home Won't Start

```bash
# Check service status
sudo systemctl status AdGuardHome

# Check logs
sudo journalctl -u AdGuardHome -n 50

# Verify configuration
/opt/AdGuardHome/AdGuardHome --check-config
```

### DNS Not Resolving

1. Check if AdGuard Home is running:
   ```bash
   sudo netstat -tulpn | grep :53
   ```

2. Verify firewall rules:
   ```bash
   sudo ufw status
   ```

3. Test DNS directly:
   ```bash
   dig @192.168.178.10 google.com
   ```

### High Memory Usage

- Reduce query log retention
- Disable unnecessary filters
- Reduce cache size
- Consider using swap space

### Web Interface Not Accessible

1. Check if service is running:
   ```bash
   sudo systemctl status AdGuardHome
   ```

2. Verify port is open:
   ```bash
   sudo netstat -tulpn | grep :3000
   ```

3. Check firewall:
   ```bash
   sudo ufw status | grep 3000
   ```

## Next Steps

Proceed to [Part 3: PiVPN Setup](./part3-pivpn-setup.md) to add VPN capabilities to your setup.

## Additional Resources

- [AdGuard Home Official Documentation](https://github.com/AdguardTeam/AdGuardHome/wiki)
- [AdGuard Home GitHub Repository](https://github.com/AdguardTeam/AdGuardHome)
- [DNS Blocklists Collection](https://filterlists.com/)
