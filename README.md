# ansible-unredacted-tor-relays

[![License](https://img.shields.io/badge/license-GPLv3-blue.svg)](LICENSE)

This Ansible playbook deploys and manages Tor relay instances on Unredacted servers. It automates the configuration of Tor relays with custom settings for exit policies, DNS-over-TLS resolution via systemd-resolved, and system-level optimizations.

## Overview

This playbook uses the [`nusenu.relayor`](https://github.com/nusenu/ansible-relayor) role to deploy Tor relays with Unredacted's custom configuration. The setup includes:

- Multiple Tor instances (one per public IP address)
- Exit relay configuration with reduced exit policy
- IPv6 support for both ORPort and exiting
- ZeroTier private network integration
- System-level TCP MSS clamping to support tunneling software like ZeroTier
- Prometheus metrics collection and alerting
- DNS-over-TLS (DoT) resolution via systemd-resolved (Cloudflare, Mullvad, WikiMedia)
- DNS resolver blacklisting to prevent censorship

## Prerequisites

### Control Machine (Ansible Host)
- Ansible 2.14 or later
- Python 3.x
- SSH access to target servers

### Target Servers
- Debian/Ubuntu-based Linux distributions
- Root or sudo access
- At least one public IPv4 address
- Optional: Public IPv6 support

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/unredacted/ansible-unredacted-tor-relays.git
   cd ansible-unredacted-tor-relays
   ```

2. Install required Ansible collections and roles:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ansible-galaxy role install -r requirements.yml
   ```

3. Configure your inventory in `inventory.ini`:
   ```ini
   [tor-relays]
   relay1.example.com ansible_user=root
   relay2.example.com ansible_user=root

   [tor-relays:vars]
   target=tor-relays
   ```

4. Customize variables in `vars/main.yml` as needed:
   - Adjust `tor_ExitPolicy` for your exit policy
   - Modify `tor_ports` for different port configurations
   - Update contact information in `tor_ContactInfo`

5. Run the playbook:
   ```bash
   ansible-playbook playbook.yml -i inventory.ini
   ```

## Configuration

### Variables

Key variables that can be customized in `vars/main.yml`:

| Variable | Description | Default |
|----------|-------------|---------|
| `tor_ContactInfo` | Contact information for the relay operator | `email:admin[@]unredacted.org` |
| `tor_ports` | ORPort and DirPort configuration | `[orport: 443, dirport: 80]` |
| `tor_ExitRelay` | Enable exit relay mode | `true` |
| `tor_IPv6` | Enable IPv6 ORPort auto-detection | `false` |
| `tor_IPv6Exit` | Enable IPv6 exiting | `true` |
| `tor_maxPublicIPs` | Maximum public IPs to use | `1` |
| `tor_dnsresolver_blacklist` | Blacklisted DNS resolvers | Quad9 (enabled), Google/OpenDNS (commented) |

### Exit Policy

The default exit policy is a reduced set of ports suitable for common web traffic:
- HTTP/HTTPS (80, 443, 8443)
- DNS (53)
- SSH (22)
- Mail ports (25, 465, 587 - some disabled due to abuse)
- Bitcoin (8332-8333)
- And many more...

See `vars/main.yml` for the complete exit policy list.

## Directory Structure

```
ansible-unredacted-tor-relays/
├── configs/
│   └── zerotier/                           # ZeroTier network configurations
├── files/
│   └── nicknames.csv                       # Tor relay nickname mappings
├── vars/
│   └── main.yml                            # Main variable definitions
├── playbook.yml                            # Main Ansible playbook
├── requirements.yml                        # Ansible Galaxy dependencies
└── README.md
```

## Features

### System Bootstrap
- TCP MSS clamping for IPv4 (1354) and IPv6 (1334)
- Systemd service for applying network settings on boot

### Network Configuration
- ZeroTier private network integration
- DNS-over-TLS (DoT) via systemd-resolved with strict mode and DNSSEC
- Upstream resolvers: Cloudflare, Mullvad, WikiMedia (all non-logging)
- Blacklist of DNS resolvers to prevent censorship

### Monitoring
- Prometheus MetricsPort integration
- Pre-configured alerting rules for:
  - Relay flag changes (Running, Sybil)
  - TCP port exhaustion
  - Certificate expiry (< 15 days warning, < 24 hours critical)
  - DNS timeout rates
  - Onionskin drops

### Security
- Offline master key generation support
- Automatic torrc backups on changes
- Password protection for MetricsPort (optional)

## ZeroTier Integration

ZeroTier network configurations should be placed in `configs/zerotier/` as `.conf` files. The playbook will automatically copy these to `/var/lib/zerotier-one/networks.d/` and restart the ZeroTier service.

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Based on the [`nusenu.relayor`](https://github.com/nusenu/ansible-relayor) Ansible role
- Hosted by [Unredacted](https://unredacted.org)

## Donations

Support this project through:
- GitHub Sponsors: [unredacted](https://github.com/sponsors/unredacted)
- Patreon: [unredacted_org](https://patreon.com/unredacted_org)
- Open Collective: [unredacted](https://opencollective.com/unredacted)
- Liberapay: [unredacted](https://liberapay.com/unredacted)
- Direct donation: https://unredacted.org/donate/