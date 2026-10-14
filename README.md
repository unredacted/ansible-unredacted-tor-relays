# ansible-unredacted-tor-relays

[![License](https://img.shields.io/badge/license-GPLv3-blue.svg)](LICENSE)

This Ansible playbook deploys and manages Tor relay instances on Unredacted servers. It automates the configuration of Tor relays with custom settings for exit policies, local recursive DNS resolution via Unbound, and system-level optimizations.

## Overview

This playbook uses the [`nusenu.relayor`](https://github.com/nusenu/ansible-relayor) role to deploy Tor relays with Unredacted's custom configuration. The setup includes:

- Multiple Tor instances (one per public IP address)
- Exit relay configuration with reduced exit policy
- IPv6 support for both ORPort and exiting
- Prometheus metrics collection and alerting
- Local recursive DNS resolution via Unbound (no third-party upstream resolvers)
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

2. Install required Ansible roles:
   ```bash
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
| `tor_IPv6` | Enable IPv6 ORPort auto-detection (turned on per host for the MCI1 Pis, see below) | `false` |
| `tor_IPv6Exit` | Enable IPv6 exiting | `true` |
| `tor_mci1_ipv6_prefix` | IPv6 /64 of the MCI1 Pi network; hosts with an address in it get a static `<prefix>::<last IPv4 octet>` and Tor IPv6 (ORPort and exit) | `2602:f7d8:0:1337::/64` |
| `tor_mci1_ipv6_gateway` | IPv6 gateway (the UniFi EFG) for the MCI1 Pi network | `2602:f7d8:0:1337::1` |
| `tor_instance_limit` | Maximum tor instances per server | `1` |
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
├── files/
│   └── nicknames.csv                       # Tor relay nickname mappings
├── vars/
│   └── main.yml                            # Main variable definitions
├── playbook.yml                            # Main Ansible playbook
├── requirements.yml                        # Ansible Galaxy dependencies
└── README.md
```

## Features

### Network Configuration
- Local Unbound recursive resolver on 127.0.0.1 with DNSSEC validation and QNAME minimisation
- systemd-resolved is masked; `/etc/resolv.conf` points directly at Unbound
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