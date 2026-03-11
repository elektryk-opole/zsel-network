# zsel-network: Network Infrastructure Layer

**Purpose**: On-premises network infrastructure management - MikroTik routers, VLANs, DNS, VPN, firewall

**Part of**: [ZSEL 7-Layer Architecture](../SERWERY/docs/ARCHITECTURE.md)

## Overview

Network infrastructure for ZSEL school:
- **MikroTik Router**: RouterOS 7 with advanced routing
- **VLANs**: Network segmentation by department, labs, admin
- **VPN**: Secure remote access for staff and students
- **DNS**: Internal DNS with zsel.opole.pl domain
- **Firewall**: Layer 3 & Layer 7 filtering
- **Network Monitoring**: Real-time traffic analysis

## Directory Structure

```
zsel-network/
├── mikrotik/
│   ├── RouterOS-config.rsc       RouterOS configuration script
│   ├── interface-setup.rsc         Interface configuration
│   ├── vlan-configuration.rsc      VLAN setup script
│   ├── firewall-rules.rsc          Firewall rules and policies
│   └── backup/                    RouterOS configuration backups
├── vlans/
│   ├── VLAN-MAPPING.md            VLAN ID and purpose mapping
│   ├── admin-vlan.conf            Administration network
│   ├── classroom-vlan.conf        Classroom networks
│   ├── lab-vlan.conf              Laboratory networks
│   └── guest-vlan.conf            Guest/visitor network
├── vpn/
│   ├── wireguard-config.conf      WireGuard VPN setup
│   ├── client-configs/            Client configuration files
│   └── vpn-users.txt              VPN user accounts
├── dns/
│   ├── zones/
│   │   └── zsel.opole.pl.zone    DNS zone file
│   ├── records.md                 DNS record documentation
│   └── dnsmasq.conf              DNS configuration
├── firewall/
│   ├── inbound-rules.rsc          Inbound traffic rules
│   ├── outbound-rules.rsc         Outbound traffic rules
│   ├── nat-rules.rsc              NAT configuration
│   └── rate-limit.rsc             Traffic rate limiting
├── documentation/
│   ├── TOPOLOGY.md                Network topology diagram
│   ├── IP-ADDRESSING.md           IP addressing scheme
│   ├── SECURITY.md                Network security policies
│   └── PROCEDURES.md              Operational procedures
└── README.md
```

## Network Topology

### Backbone
```
ISP Connection
    ↓ (1Gbps)
MikroTik Router (RouterOS 7)
    ↓
LAG to Kubernetes Cluster
    ↓
10Gbps internal network (Cilium overlay)
```

### VLAN Segmentation

| VLAN ID | Name | Purpose | Subnet | Notes |
|---------|------|---------|--------|-------|
| 10 | Admin | Administration, management | 10.10.0.0/24 | Restricted access |
| 20 | Classrooms | Classroom networks | 10.20.0.0/22 | Multiple subnets |
| 30 | Labs | Laboratory networks | 10.30.0.0/22 | Isolated |
| 40 | Kubernetes | K8s cluster | 10.42.0.0/16 | Pod CIDR |
| 50 | Guest | Guest/visitor network | 10.50.0.0/24 | Limited access |
| 99 | Mgmt | Network management | 10.99.0.0/24 | Staff only |

## MikroTik Configuration

### Setup from Scratch
1. Import RouterOS-config.rsc
2. Configure interfaces with interface-setup.rsc
3. Apply VLANs with vlan-configuration.rsc
4. Apply firewall rules with firewall-rules.rsc
5. Enable VPN with wireguard-config.conf

```bash
# SSH to MikroTik
ssh admin@10.189.5.1

# Import configuration
import file=RouterOS-config.rsc

# Verify configuration
/interface print
/ip address print
/interface vlan print
```

### Backup Configuration
```bash
# Backup to file
/system backup save name=backup-2026-03-11

# Export configuration
/system export file=config-export.rsc

# Download via SCP
scp admin@10.189.5.1:config-export.rsc ./
```

## VLANs

### Classroom VLAN (20)
- **Purpose**: Student and instructor access
- **Devices**: Laptops, tablets, printers
- **Security**: Standard firewall rules
- **DNS**: zsel.opole.pl domain
- **Subnets**: 10.20.0.0/25 through 10.20.3.128/25 (one per classroom)

### Lab VLAN (30)
- **Purpose**: Lab equipment and experiments
- **Devices**: Specialized hardware, test equipment
- **Security**: Strict firewall rules, no internet access by default
- **Isolation**: Cannot reach other VLANs without firewall rule
- **Monitoring**: All traffic logged

### Admin VLAN (10)
- **Purpose**: Server and administrative access
- **Devices**: Management interfaces, admin workstations
- **Security**: Strongest security policies
- **Access**: Limited to staff only
- **Logging**: Comprehensive audit logging

### Guest VLAN (50)
- **Purpose**: Visitor and guest access
- **Devices**: Personal devices, guest laptops
- **Security**: Cannot reach internal networks
- **Bandwidth**: Limited QoS policy (fair share)
- **Captive Portal**: Must accept AUP

## DNS

### zsel.opole.pl Zone

```dns
zsel.opole.pl.           3600    IN    A       10.189.5.1
kubernetes.zsel.opole.pl 3600   IN    A       10.189.5.2
api.zsel.opole.pl        3600   IN    A       10.189.5.2
moodle.zsel.opole.pl     3600   IN    A       10.189.5.2
nextcloud.zsel.opole.pl  3600   IN    A       10.189.5.2
keycloak.zsel.opole.pl   3600   IN    A       10.189.5.2
```

### Internal DNS
DMasq configuration for internal DNS:
```
address=/zsel.opole.pl/10.189.5.1
address=/kubernetes.zsel.opole.pl/10.189.5.2
```

## VPN Configuration

### WireGuard VPN
```bash
# Generate client keys
cd vpn/
wg genkey | tee private.key | wg pubkey > public.key

# Create client config
cat > client.conf << EOF
[Interface]
PrivateKey = <private_key>
Address = 10.99.2.10/32
DNS = 10.189.5.1

[Peer]
PublicKey = <server_public_key>
AllowedIPs = 10.0.0.0/8
Endpoint = vpn.zsel.opole.pl:51820
PersistentKeepalive = 25
EOF
```

### VPN Access Policies
- Teachers: Full access to all networks
- Students: Classroom + guest VLANs only
- Visitors: Guest VLAN only
- Admin: Management VLAN only

## Firewall Rules

### Ingress Filter
- Drop invalid connections
- Block known bad sources
- Rate limit suspicious traffic
- Log all connections

### Egress Filter
- Allow DNS (UDP 53)
- Allow NTP (UDP 123)
- Allow HTTPS (TCP 443)
- Allow SSH (TCP 22) from admin VLAN only
- Block torrents, P2P
- Block known malware domains

### Rate Limiting
- 100Mbps per classroom
- 500Mbps per lab
- 1Gbps per admin VLAN
- 50Mbps per guest

## Network Monitoring

### Monitor Traffic
```bash
ssh admin@10.189.5.1
/ip traffic flow
```

### Bandwidth Monitoring
```bash
/interface monitor-traffic ether1
```

### Connection Tracking
```bash
/ip firewall connection print
```

## Documentation

See [documentation/](documentation/) for:
- **TOPOLOGY.md** - Complete network diagram
- **IP-ADDRESSING.md** - IP allocation and CIDR blocks
- **SECURITY.md** - Network security policies and procedures
- **PROCEDURES.md** - Operational procedures and troubleshooting

## Disaster Recovery

### Network Failover
1. Primary router fails → Secondary router takes over
2. All services redirect through secondary
3. Restore primary and fail back

### Configuration Recovery
1. RouterOS restores from backup
2. VLANs and firewall rules reapplied
3. VPN and DNS restored
4. Network comes online in < 5 minutes

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- RouterOS configuration standards
- Security policy updates
- VLAN and firewall rule changes
- Network documentation updates

## License

MIT License - See [LICENSE](LICENSE)

---

**Maintainers**: Network Engineering Team  
**Last Updated**: March 11, 2026
