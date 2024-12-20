# AWS WireGuard VPN with Privacy Enhancements

This repository contains instructions and configurations for setting up a privacy-focused WireGuard VPN on AWS that emphasizes identity fractalization and traffic pattern protection.

## Project Overview

We implement a lightweight but secure VPN solution running on AWS t2.nano instances, with additional privacy features beyond standard WireGuard implementations. The goal is to create a personal VPN service that:

- Minimizes attack surface with only essential ports open
- Maintains clear routing rules between interfaces
- Costs approximately $5-15 per month to operate
- Provides options for additional traffic encryption

## Core Architecture

The system uses a standard WireGuard implementation with carefully configured routing rules that enable:

1. Client to server secure tunneling
2. Proper NAT masquerading
3. Clean traffic flow between interfaces (eth0 ↔ wg0)
4. Options for pre-AWS encryption

## Prerequisites

- AWS Account
- Basic understanding of networking concepts
- WireGuard client (mobile or desktop)
- Terminal access for configuration

## Installation Instructions

### AWS Setup

1. Launch a t2.nano instance in your preferred region
2. Configure security groups:
   - UDP 51820 (WireGuard)
   - TCP 22 (SSH initial setup only)
   - All outbound traffic allowed

### Server Configuration

```ini
[Interface]
PrivateKey = <server_private_key>
Address = 10.9.0.1/24
ListenPort = 51820
PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.9.0.2/32
```

### Client Configuration

```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.9.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = <server_public_key>
Endpoint = <aws_public_ip>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Key Management

The system requires proper key management between server and client:
- Each side generates its own keypair
- Private keys stay local
- Public keys are exchanged
- Each config references only its own private key and the peer's public key

## Cost Management

The solution is designed to be cost-effective:
- t2.nano instance: ~$3.50/month
- Data transfer: $0.09/GB outbound
- Typical monthly total: $5-15 depending on usage

## Advanced Privacy Features

### Optional Pre-AWS Encryption
The system can be enhanced with additional encryption layers:
1. Simple pre-encryption tunnel
2. Layered encryption mesh
3. Traffic pattern masking

These features prevent AWS from seeing actual traffic content while maintaining the benefits of WireGuard's performance.

## Monitoring and Maintenance

Essential commands for system monitoring:
```bash
# Check connection status
sudo wg show

# Monitor logs
sudo journalctl -fu wg-quick@wg0

# Start/Stop service
sudo wg-quick up wg0
sudo wg-quick down wg0
```

## Security Considerations

1. Restrict SSH access to known IPs after initial setup
2. Monitor connection logs regularly
3. Keep private keys secure
4. Consider implementing additional encryption layers
5. Set up AWS billing alerts

## Troubleshooting

Common issues and solutions:
1. No handshake: Check key matching between peers
2. No forwarding: Verify IP forwarding is enabled
3. No connectivity: Confirm iptables rules are correct
4. Connection drops: Verify security group settings

## Contributing

Contributions to enhance privacy features or improve documentation are welcome. Please submit pull requests with detailed descriptions of changes.

## License

Open source

## Acknowledgments

This implementation builds on the WireGuard protocol while adding privacy-focused enhancements for AWS deployment.
