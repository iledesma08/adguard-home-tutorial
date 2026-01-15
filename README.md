# AdGuard Home + PiVPN + Tailscale + DoH/DoT/DDNS on Raspberry Pi Zero 2W

A comprehensive guide to setting up a network-wide ad blocker, VPN server, and encrypted DNS resolver on a Raspberry Pi Zero 2W connected to a Fritz!Box router in Germany.

## 🎯 What You'll Build

This tutorial will guide you through creating a complete home network security and privacy setup:

- **AdGuard Home**: Network-wide ad blocking and tracking protection
- **PiVPN (WireGuard)**: Secure VPN access to your home network
- **Tailscale**: Easy mesh network for device-to-device connectivity
- **DNS over HTTPS (DoH)**: Encrypted DNS queries via HTTPS
- **DNS over TLS (DoT)**: Encrypted DNS queries via TLS
- **Dynamic DNS (DDNS)**: Maintain access with a changing public IP
- **Fritz!Box Integration**: Optimized configuration for Fritz!Box routers

## 🔧 Hardware Requirements

- **Raspberry Pi Zero 2W** (1GB RAM, Quad-core ARM Cortex-A53)
- **MicroSD Card** (16GB or larger, Class 10 recommended)
- **USB to Ethernet Adapter** (for wired connection)
- **Power Supply** (5V, 2.5A minimum)
- **Fritz!Box Router** (any model with current firmware)
- **Ethernet Cable** (Cat5e or better)
- Optional: Case with cooling

## 📚 Tutorial Index

### [Part 1: Raspberry Pi Zero 2W Setup](./docs/part1-raspberry-pi-setup.md)
- Initial Raspberry Pi OS installation
- Network configuration and static IP setup
- System updates and security hardening
- SSH access configuration
- Firewall setup with UFW

### [Part 2: AdGuard Home Installation and Configuration](./docs/part2-adguard-home.md)
- AdGuard Home installation
- DNS server configuration
- Upstream DNS providers (DoH/DoT)
- Ad blocking filters and blocklists
- Query logging and statistics
- Fritz!Box DNS integration
- Performance optimization

### [Part 3: PiVPN Setup](./docs/part3-pivpn-setup.md)
- WireGuard VPN installation via PiVPN
- Client configuration generation
- Port forwarding on Fritz!Box
- Mobile and desktop client setup
- Split tunneling configuration
- VPN management and troubleshooting

### [Part 4: Tailscale Integration](./docs/part4-tailscale.md)
- Tailscale installation and authentication
- Subnet routing configuration
- MagicDNS setup
- Exit node configuration
- ACL (Access Control Lists) setup
- Integration with AdGuard Home DNS

### [Part 5: DoH/DoT/DDNS Configuration](./docs/part5-doh-dot-ddns.md)
- Dynamic DNS setup (DuckDNS, No-IP, MyFRITZ!)
- SSL/TLS certificate with Let's Encrypt
- DNS over HTTPS (DoH) configuration
- DNS over TLS (DoT) configuration
- Client device configuration
- Certificate auto-renewal

### [Part 6: Fritz!Box Integration and Optimization](./docs/part6-fritzbox-integration.md)
- Fritz!Box network configuration
- Port forwarding best practices
- IPv6 setup
- Guest network configuration
- Performance optimization
- Monitoring and maintenance
- Security hardening

## 🚀 Quick Start

1. **Clone or download this repository:**
   ```bash
   git clone https://github.com/iledesma08/adguard-home-tutorial.git
   cd adguard-home-tutorial
   ```

2. **Follow the tutorials in order:**
   - Start with [Part 1: Raspberry Pi Setup](./docs/part1-raspberry-pi-setup.md)
   - Progress through each part sequentially
   - Each part builds on the previous setup

3. **Use the provided configuration files:**
   - `configs/AdGuardHome-basic.yaml` - Basic AdGuard Home configuration
   - `configs/AdGuardHome-advanced.yaml` - Advanced setup with DoH/DoT

## 📁 Repository Structure

```
adguard-home-tutorial/
├── README.md                          # This file - main index and overview
├── docs/                              # Tutorial documentation
│   ├── part1-raspberry-pi-setup.md    # Raspberry Pi initial setup
│   ├── part2-adguard-home.md          # AdGuard Home installation
│   ├── part3-pivpn-setup.md           # PiVPN/WireGuard setup
│   ├── part4-tailscale.md             # Tailscale mesh network
│   ├── part5-doh-dot-ddns.md          # Encrypted DNS and DDNS
│   └── part6-fritzbox-integration.md  # Fritz!Box configuration
├── configs/                           # Configuration files
│   ├── AdGuardHome-basic.yaml         # Basic AdGuard Home config
│   └── AdGuardHome-advanced.yaml      # Advanced config with encryption
└── images/                            # Images and screenshots (add your own)
```

## ✨ Features

### Network-Wide Ad Blocking
- Blocks ads on all devices on your network
- Protects against tracking and malware domains
- Customizable blocklists
- Per-client filtering rules

### Secure Remote Access
- WireGuard VPN for secure remote access
- Tailscale for easy peer-to-peer connections
- Access your home network from anywhere
- Use ad blocking on mobile devices remotely

### Privacy and Security
- Encrypted DNS queries (DoH/DoT)
- DNSSEC validation
- Protection against DNS hijacking
- Query logging for monitoring

### Easy Management
- Web-based AdGuard Home dashboard
- Simple PiVPN command-line tools
- Tailscale mobile and desktop apps
- Fritz!Box web interface integration

## 🌍 Why This Setup?

This guide is specifically designed for:
- **German Users**: Optimized for Fritz!Box routers common in Germany
- **Privacy Conscious**: Multiple layers of encryption and security
- **Low Power**: Raspberry Pi Zero 2W is energy efficient (2-3W)
- **Cost Effective**: Uses affordable hardware and free software
- **Always On**: 24/7 protection for your entire network

## 🔒 Security Considerations

- All services run on a dedicated, hardened Raspberry Pi
- Firewall configured with UFW (only necessary ports open)
- Regular automatic security updates
- SSL/TLS certificates for encrypted communication
- Strong authentication on all services
- Regular backups recommended

## 📊 Performance Notes

The Raspberry Pi Zero 2W is capable of handling:
- ✅ DNS queries for a household (thousands per day)
- ✅ Multiple VPN clients (3-5 concurrent connections)
- ✅ AdGuard Home filtering and logging
- ✅ Tailscale mesh networking
- ⚠️ Limited throughput as VPN exit node (~50-100 Mbps)

For more demanding use cases, consider upgrading to Raspberry Pi 4 or 5.

## 🛠️ Maintenance

### Regular Tasks
- **Daily**: Check AdGuard Home dashboard for anomalies
- **Weekly**: Review VPN connection logs
- **Monthly**: Update system packages and AdGuard Home
- **Quarterly**: Review and update blocklists
- **Yearly**: Backup all configurations

### Update Commands
```bash
# Update Raspberry Pi OS
sudo apt update && sudo apt upgrade -y

# Update AdGuard Home (via web interface)
# Settings → General Settings → Update

# Update PiVPN
pivpn update

# Update Tailscale
sudo apt update && sudo apt install tailscale
```

## 🤝 Contributing

This tutorial is open for contributions! If you:
- Find errors or typos
- Have suggestions for improvements
- Want to add additional guides or configurations
- Created useful scripts or tools

Please feel free to:
1. Open an issue
2. Submit a pull request
3. Share your feedback

## 📖 Additional Resources

### Official Documentation
- [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)
- [PiVPN](https://www.pivpn.io/)
- [WireGuard](https://www.wireguard.com/)
- [Tailscale](https://tailscale.com/kb/)
- [Raspberry Pi](https://www.raspberrypi.org/documentation/)
- [Fritz!Box Support (AVM)](https://en.avm.de/service/)

### Community
- [AdGuard Home Subreddit](https://www.reddit.com/r/Adguard/)
- [WireGuard Subreddit](https://www.reddit.com/r/WireGuard/)
- [Raspberry Pi Forums](https://www.raspberrypi.org/forums/)

### Related Projects
- [Pi-hole](https://pi-hole.net/) - Alternative DNS-based ad blocker
- [Unbound](https://nlnetlabs.nl/projects/unbound/) - Recursive DNS resolver
- [DNSCrypt](https://www.dnscrypt.org/) - DNS encryption protocol

## ⚠️ Disclaimer

This tutorial is provided for educational purposes. While these tools are designed to enhance privacy and security:
- Follow your ISP's terms of service
- Ensure compliance with local laws
- Use responsibly and ethically
- Keep all software updated
- Regularly backup your configurations

## 📝 License

This tutorial is released under the MIT License. See [LICENSE](LICENSE) for details.

## 👤 Author

Created by **Ignacio Ledesma** ([@iledesma08](https://github.com/iledesma08))

## ⭐ Support

If you find this tutorial helpful:
- Star this repository
- Share it with others
- Report issues or suggest improvements
- Consider contributing to the projects used:
  - [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)
  - [PiVPN](https://github.com/pivpn/pivpn)
  - [Tailscale](https://tailscale.com/)

---

**Happy Ad Blocking!** 🚀🔒

*Last Updated: January 2026*