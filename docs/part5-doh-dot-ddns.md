# Part 5: DoH/DoT/DDNS Configuration

## Overview

In this part, we'll configure:
- **DNS over HTTPS (DoH):** Encrypted DNS queries over HTTPS
- **DNS over TLS (DoT):** Encrypted DNS queries over TLS
- **Dynamic DNS (DDNS):** Keep a hostname pointing to your changing public IP

## Why Encrypted DNS?

Traditional DNS queries are sent in plain text, allowing:
- ISPs to see what websites you visit
- Potential DNS hijacking
- Man-in-the-middle attacks

Encrypted DNS (DoH/DoT) solves these problems by encrypting your DNS queries.

## Prerequisites

For DoH/DoT, you'll need:
- A domain name (can be free via DuckDNS, No-IP, etc.)
- SSL/TLS certificates (we'll use Let's Encrypt)
- Port 443 (HTTPS) or 853 (DoT) forwarded on Fritz!Box

## Part A: Dynamic DNS (DDNS) Setup

Most home internet connections have dynamic IP addresses that change periodically. DDNS keeps a hostname pointed to your current IP.

### Option 1: DuckDNS (Free, Recommended)

DuckDNS is a free DDNS service that's easy to setup.

#### 1. Create DuckDNS Account

1. Visit https://www.duckdns.org/
2. Sign in with one of the supported providers (GitHub, Google, etc.)
3. Create a subdomain (e.g., `your-home.duckdns.org`)
4. Note your token

#### 2. Install DuckDNS on Raspberry Pi

Create a directory for DuckDNS:
```bash
mkdir ~/duckdns
cd ~/duckdns
```

Create the update script:
```bash
nano duck.sh
```

Add the following content (replace with your values):
```bash
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=your-home&token=YOUR-TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
```

Make it executable:
```bash
chmod +x duck.sh
```

Test it:
```bash
./duck.sh
cat duck.log
```

You should see `OK` in the log file.

#### 3. Setup Automatic Updates

Add to crontab:
```bash
crontab -e
```

Add this line to update every 5 minutes:
```bash
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

#### 4. Verify

Visit your DuckDNS domain in a browser or use:
```bash
nslookup your-home.duckdns.org
```

It should resolve to your current public IP.

### Option 2: Fritz!Box Built-in DDNS

Fritz!Box supports several DDNS providers natively.

#### 1. Access Fritz!Box DDNS Settings

1. Go to `http://fritz.box`
2. Navigate to **Internet** → **Permit Access** → **DynDNS**
3. Enable **Use DynDNS**

#### 2. Configure DDNS Provider

Select your provider and enter:
- **DynDNS provider:** Choose from the list (No-IP, DynDNS, etc.)
- **Domain name:** Your registered domain/hostname
- **Username:** Your DDNS account username
- **Password:** Your DDNS account password

#### 3. Apply and Test

Click **Apply** and Fritz!Box will automatically update your DDNS whenever the IP changes.

### Option 3: No-IP (Free with limitations)

#### 1. Create Account

1. Visit https://www.noip.com/
2. Create a free account
3. Create a hostname (e.g., `your-home.hopto.org`)

#### 2. Install No-IP Client

```bash
cd /tmp
wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
tar xzf noip-duc-linux.tar.gz
cd noip-2.1.9-1
sudo make
sudo make install
```

During installation, enter:
- Your No-IP username
- Your No-IP password
- Update interval (30 recommended)

#### 3. Configure as Service

Create systemd service:
```bash
sudo nano /etc/systemd/system/noip2.service
```

Add:
```ini
[Unit]
Description=No-IP Dynamic DNS Update Client
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/noip2
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl enable noip2
sudo systemctl start noip2
```

## Part B: Obtaining SSL/TLS Certificates

For DoH and DoT, you need valid SSL certificates.

### Prerequisites

- A domain name (from DDNS setup above)
- Port 80 forwarded temporarily (for HTTP-01 challenge)
- Port 443 forwarded (for HTTPS/DoH)

### Install Certbot

```bash
sudo apt install certbot -y
```

### Obtain Certificate

#### Method 1: Standalone Mode (Easiest)

This method temporarily stops AdGuard Home to use port 80.

```bash
# Stop AdGuard Home temporarily
sudo /opt/AdGuardHome/AdGuardHome -s stop

# Obtain certificate
sudo certbot certonly --standalone -d your-home.duckdns.org

# Start AdGuard Home
sudo /opt/AdGuardHome/AdGuardHome -s start
```

#### Method 2: DNS Challenge (No port 80 needed)

For DuckDNS:
```bash
# Install DNS plugin
sudo apt install certbot python3-certbot-dns-cloudflare -y

# Manual DNS challenge
sudo certbot certonly --manual --preferred-challenges dns -d your-home.duckdns.org
```

Follow the prompts to add a TXT record to your DNS.

### Certificate Location

Certificates are stored in:
```
/etc/letsencrypt/live/your-home.duckdns.org/
  - cert.pem       (certificate)
  - chain.pem      (intermediate chain)
  - fullchain.pem  (cert + chain)
  - privkey.pem    (private key)
```

### Setup Automatic Renewal

Certbot automatically creates a renewal timer. Verify:
```bash
sudo systemctl status certbot.timer
```

Create a renewal hook to restart AdGuard Home:
```bash
sudo nano /etc/letsencrypt/renewal-hooks/post/restart-adguard.sh
```

Add:
```bash
#!/bin/bash
/opt/AdGuardHome/AdGuardHome -s restart
```

Make executable:
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/restart-adguard.sh
```

Test renewal (dry-run):
```bash
sudo certbot renew --dry-run
```

## Part C: Configure DNS over HTTPS (DoH)

### 1. Stop AdGuard Home

```bash
sudo /opt/AdGuardHome/AdGuardHome -s stop
```

### 2. Copy Certificates

AdGuard Home needs access to the certificates:
```bash
sudo mkdir -p /opt/adguardhome/certs
sudo cp /etc/letsencrypt/live/your-home.duckdns.org/fullchain.pem /opt/adguardhome/certs/
sudo cp /etc/letsencrypt/live/your-home.duckdns.org/privkey.pem /opt/adguardhome/certs/
sudo chown -R AdGuardHome:AdGuardHome /opt/adguardhome/certs
sudo chmod 600 /opt/adguardhome/certs/privkey.pem
```

Or create a renewal hook to copy automatically:
```bash
sudo nano /etc/letsencrypt/renewal-hooks/post/copy-certs.sh
```

Add:
```bash
#!/bin/bash
cp /etc/letsencrypt/live/your-home.duckdns.org/fullchain.pem /opt/adguardhome/certs/
cp /etc/letsencrypt/live/your-home.duckdns.org/privkey.pem /opt/adguardhome/certs/
chown -R AdGuardHome:AdGuardHome /opt/adguardhome/certs
chmod 600 /opt/adguardhome/certs/privkey.pem
/opt/AdGuardHome/AdGuardHome -s restart
```

Make executable:
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/copy-certs.sh
```

### 3. Configure AdGuard Home

Edit the configuration:
```bash
sudo nano /opt/AdGuardHome/AdGuardHome.yaml
```

Update the TLS section:
```yaml
tls:
  enabled: true
  server_name: your-home.duckdns.org
  force_https: true
  port_https: 443
  port_dns_over_tls: 853
  port_dns_over_quic: 784
  port_dnscrypt: 0
  dnscrypt_config_file: ""
  allow_unencrypted_doh: false
  certificate_chain: ""
  private_key: ""
  certificate_path: /opt/adguardhome/certs/fullchain.pem
  private_key_path: /opt/adguardhome/certs/privkey.pem
  strict_sni_check: false
```

Or use the provided `AdGuardHome-advanced.yaml` from this repository:
```bash
sudo cp ~/adguard-home-tutorial/configs/AdGuardHome-advanced.yaml /opt/AdGuardHome/AdGuardHome.yaml
# Edit to set your domain and password
sudo nano /opt/AdGuardHome/AdGuardHome.yaml
```

### 4. Update Firewall

```bash
sudo ufw allow 443/tcp   # HTTPS / DoH
sudo ufw allow 853/tcp   # DoT
```

### 5. Configure Port Forwarding on Fritz!Box

1. Go to `http://fritz.box`
2. Navigate to **Internet** → **Permit Access** → **Port Sharing**
3. Add new port sharing:
   - **Device:** Raspberry Pi (`192.168.178.10`)
   - **Application:** HTTPS
   - **Protocol:** TCP
   - **Port to device:** 443
   - **to Port:** 443
4. Add another for DoT:
   - **Protocol:** TCP
   - **Port to device:** 853
   - **to Port:** 853

### 6. Start AdGuard Home

```bash
sudo /opt/AdGuardHome/AdGuardHome -s start
```

### 7. Verify DoH is Working

**Access Web Interface over HTTPS:**
```
https://your-home.duckdns.org
```

You should see the AdGuard Home login page with a valid SSL certificate.

**Test DoH Endpoint:**
```bash
curl -H 'accept: application/dns-json' 'https://your-home.duckdns.org/dns-query?name=google.com&type=A'
```

You should see a JSON response with DNS information.

## Part D: Configure DNS over TLS (DoT)

DoT is automatically configured when you enable TLS in AdGuard Home (port 853).

### Test DoT

**Using kdig (from knot-dnsutils):**
```bash
sudo apt install knot-dnsutils -y
kdig -d @your-home.duckdns.org +tls-ca +tls-host=your-home.duckdns.org google.com
```

**Using drill:**
```bash
sudo apt install ldnsutils -y
drill @your-home.duckdns.org -p 853 google.com
```

## Part E: Client Configuration

### Configure Devices to Use DoH/DoT

#### Android

**System-wide DoT (Android 9+):**
1. Go to **Settings** → **Network & Internet** → **Advanced** → **Private DNS**
2. Select **Private DNS provider hostname**
3. Enter: `your-home.duckdns.org`
4. Tap **Save**

**Using AdGuard App for DoH:**
1. Install [AdGuard for Android](https://adguard.com/en/adguard-android/overview.html)
2. Go to **Settings** → **DNS filtering** → **DNS server**
3. Select **Custom DNS server**
4. Add: `https://your-home.duckdns.org/dns-query`

#### iOS

**DoT (iOS 14+):**
1. Install [DNSCloak](https://apps.apple.com/app/dnscloak-secure-dns-client/id1452162351) or similar
2. Add custom DNS server: `your-home.duckdns.org`

**Configuration Profile (Advanced):**
Create a mobile config profile with DoT settings.

#### Windows

**DoH with Firefox:**
1. Open Firefox
2. Go to **Settings** → **General** → **Network Settings**
3. Enable **DNS over HTTPS**
4. Choose **Custom** and enter:
   ```
   https://your-home.duckdns.org/dns-query
   ```

**DoH with Chrome:**
1. Go to **Settings** → **Privacy and security** → **Security**
2. Enable **Use secure DNS**
3. Choose **Custom** and enter:
   ```
   https://your-home.duckdns.org/dns-query
   ```

**System-wide (Windows 11):**
Windows 11 supports DoH natively in network adapter settings.

#### macOS

**DoH with Firefox/Chrome:**
Same as Windows instructions above.

**System-wide DoH/DoT:**
Use [DNSCrypt Proxy](https://github.com/DNSCrypt/dnscrypt-proxy) or similar tools.

#### Linux

**Using systemd-resolved (DoT):**

Edit `/etc/systemd/resolved.conf`:
```ini
[Resolve]
DNS=your-home.duckdns.org
DNSOverTLS=yes
```

Restart:
```bash
sudo systemctl restart systemd-resolved
```

Verify:
```bash
resolvectl status
```

**Using dnscrypt-proxy (DoH):**
```bash
sudo apt install dnscrypt-proxy -y
```

Edit `/etc/dnscrypt-proxy/dnscrypt-proxy.toml`:
```toml
server_names = ['custom']

[static.'custom']
stamp = 'sdns://AgcAAAAAAAAADzE5Mi4xNjguMTc4LjEwIBw...'
```

Generate the stamp at: https://dnscrypt.info/stamps/

## Part F: Testing Encrypted DNS

### Test DoH

**Command Line:**
```bash
curl -H 'accept: application/dns-json' 'https://your-home.duckdns.org/dns-query?name=example.com&type=A'
```

**Browser:**
Open:
```
https://your-home.duckdns.org/dns-query?name=example.com&type=A
```

### Test DoT

```bash
kdig -d @your-home.duckdns.org +tls-ca +tls-host=your-home.duckdns.org example.com
```

### Verify Encryption

**Check DNS Leaks:**
Visit https://www.dnsleaktest.com/ from a device configured to use your DoH/DoT.

You should see your home public IP, not your ISP's DNS servers.

**Use Wireshark:**
Capture DNS traffic and verify queries are encrypted (you won't see plain text domains).

## Performance Considerations

### DoH vs DoT vs Plain DNS

| Metric | Plain DNS | DoT | DoH |
|--------|-----------|-----|-----|
| **Speed** | Fastest | Fast | Fast |
| **Overhead** | None | TLS handshake | TLS + HTTP |
| **CPU Usage** | Lowest | Medium | Higher |
| **Port** | 53 | 853 | 443 |

**On Raspberry Pi Zero 2W:**
- Plain DNS: Handles easily
- DoT: Handles well
- DoH: Works but adds more overhead

### Optimization Tips

1. **Use DoT over DoH:**
   - Lower overhead
   - Better performance on limited hardware

2. **Enable DNSSEC:**
   - Already enabled in our config
   - Adds security with minimal overhead

3. **Optimize TLS:**
   - Use TLS 1.3 (faster handshakes)
   - Enable session resumption

4. **Monitor Performance:**
   ```bash
   # Check CPU usage
   top

   # Check AdGuard Home performance
   curl -s http://192.168.178.10:3000/control/stats
   ```

## Security Best Practices

1. **Keep Certificates Updated:**
   - Certbot auto-renews, but monitor:
   ```bash
   sudo certbot certificates
   ```

2. **Use Strong Ciphers:**
   - Let's Encrypt uses strong defaults
   - Verify with: https://www.ssllabs.com/ssltest/

3. **Enable Force HTTPS:**
   - Already enabled in our config
   - Prevents downgrade attacks

4. **Regular Security Updates:**
   ```bash
   sudo apt update && sudo apt upgrade
   ```

5. **Monitor Access Logs:**
   ```bash
   sudo journalctl -u AdGuardHome -f
   ```

6. **Consider Fail2Ban:**
   - Protect against brute force attacks
   - Monitor failed login attempts

## Troubleshooting

### Certificate Errors

**"Certificate not valid":**
1. Check certificate is not expired:
   ```bash
   sudo certbot certificates
   ```
2. Verify domain matches certificate:
   ```bash
   openssl s_client -connect your-home.duckdns.org:443 -servername your-home.duckdns.org
   ```
3. Check certificate files exist:
   ```bash
   ls -la /opt/adguardhome/certs/
   ```

### DoH Not Working

1. **Verify port 443 is forwarded:**
   Test externally:
   ```bash
   curl -I https://your-home.duckdns.org
   ```

2. **Check AdGuard Home is listening:**
   ```bash
   sudo netstat -tulpn | grep 443
   ```

3. **Review logs:**
   ```bash
   sudo journalctl -u AdGuardHome -f
   ```

4. **Test locally first:**
   ```bash
   curl -k https://192.168.178.10/dns-query?name=google.com
   ```

### DoT Not Working

1. **Verify port 853 is open:**
   ```bash
   sudo ufw status | grep 853
   ```

2. **Test with kdig:**
   ```bash
   kdig @192.168.178.10 +tls-ca +tls-host=your-home.duckdns.org google.com
   ```

3. **Check TLS configuration:**
   ```bash
   sudo nano /opt/AdGuardHome/AdGuardHome.yaml
   # Verify port_dns_over_tls: 853
   ```

### DDNS Not Updating

**DuckDNS:**
1. Check the update log:
   ```bash
   cat ~/duckdns/duck.log
   ```
2. Manually run the update script:
   ```bash
   ~/duckdns/duck.sh
   ```
3. Verify token and domain are correct

**No-IP:**
1. Check service status:
   ```bash
   sudo systemctl status noip2
   ```
2. Check configuration:
   ```bash
   sudo /usr/local/bin/noip2 -S
   ```

### Certificate Renewal Fails

1. **Check certbot logs:**
   ```bash
   sudo cat /var/log/letsencrypt/letsencrypt.log
   ```

2. **Test renewal manually:**
   ```bash
   sudo certbot renew --dry-run
   ```

3. **Verify port 80 is accessible:**
   - Temporarily forward port 80 on Fritz!Box
   - Or use DNS challenge instead

4. **Check domain resolves correctly:**
   ```bash
   nslookup your-home.duckdns.org
   ```

## Next Steps

Proceed to [Part 6: Fritz!Box Integration](./part6-fritzbox-integration.md) for advanced Fritz!Box configuration and optimization.

## Additional Resources

- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [DuckDNS Documentation](https://www.duckdns.org/spec.jsp)
- [AdGuard Home TLS Guide](https://github.com/AdguardTeam/AdGuardHome/wiki/Encryption)
- [DNS over HTTPS RFC 8484](https://tools.ietf.org/html/rfc8484)
- [DNS over TLS RFC 7858](https://tools.ietf.org/html/rfc7858)
