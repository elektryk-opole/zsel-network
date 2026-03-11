# Contributing to zsel-network

Welcome to network infrastructure contributions!

## Getting Started

1. **Fork** the repository
2. **Clone** to your local machine
3. **Create a branch**: `git checkout -b feature/network-feature`
4. **Document changes** thoroughly
5. **Submit pull request**

## Network Changes

### Procedure
Every network change requires:
1. **Design document**: Why and how
2. **Impact analysis**: What's affected
3. **Testing plan**: How to verify
4. **Rollback procedure**: How to revert
5. **Maintenance window**: When to deploy
6. **Verification**: Proof of success

### RouterOS Configuration

```bash
# Backup current config
ssh admin@10.189.5.1
/system backup save name=backup-pre-change

# Export configuration
/system export file=config-pre-change.rsc

# Review changes before applying
# Apply new configuration
import file=new-config.rsc

# Test connectivity
/ip route check 8.8.8.8
```

## VLAN Changes

When adding/modifying VLANs:
1. **Document** VLAN ID and purpose
2. **Plan** IP addressing
3. **Configure** switch ports
4. **Test** connectivity
5. **Update** documentation
6. **Communicate** to affected users

## Firewall Rule Changes

Rules must include:
- Clear description of what rule does
- Why the rule is needed
- Impact on users/services
- Test procedure
- Rollback step

## VPN Configuration

For VPN changes:
1. Generate and test keys
2. Update client configurations
3. Document access policy
4. Test connectivity
5. Update user documentation
6. Announce to users

## DNS Changes

When modifying DNS:
1. Update zone file
2. Increment zone serial
3. Test zone validation: `named-checkzone zsel.opole.pl zsel.opole.pl.zone`
4. Roll out gradually (not all servers at once)
5. Wait for TTL to expire
6. Verify global DNS propagation

## Monitoring & Alerts

After network changes:
- [ ] Verify monitoring data
- [ ] Check alert thresholds
- [ ] Test alert notifications
- [ ] Document any metric changes

## Documentation Updates

Update when you change:
- **VLAN mapping** → [vlans/VLAN-MAPPING.md](vlans/VLAN-MAPPING.md)
- **IP addressing** → [documentation/IP-ADDRESSING.md](documentation/IP-ADDRESSING.md)
- **Firewall rules** → [firewall/](firewall/)
- **Topology** → [documentation/TOPOLOGY.md](documentation/TOPOLOGY.md)

## Code Standards

### RouterOS Scripts
```routeros
# Use descriptive names
:local newVlanId 35;
:local newVlanName "Labs-Extended";

# Add helpful comments
# Create VLAN interface for lab network
/interface vlan add name=$newVlanName vlan-id=$newVlanId interface=ether1

# Test the configuration
/ip address add address=10.30.1.1/24 interface=$newVlanName
```

## Testing Network Changes

Test procedure:
1. [ ] Configuration syntax valid
2. [ ] Test from isolated subnet
3. [ ] Test from each VLAN
4. [ ] Test failover scenario
5. [ ] Test under load
6. [ ] Monitor for 24 hours post-change

## Security Review

Network changes reviewed for:
- [ ] Firewall rule clarity
- [ ] No rule conflicts
- [ ] Least privilege applied
- [ ] Logging enabled
- [ ] Rate limiting appropriate
- [ ] DDoS mitigation active

## Performance Impact

Large changes need performance testing:
1. Establish baseline metrics
2. Implement change
3. Monitor for 1 week
4. Document any impact
5. Adjust if needed

## Backup & Recovery

Before any change:
- [ ] Backup RouterOS config
- [ ] Export all VLAN configs
- [ ] Document current DNS records
- [ ] Test restore procedure
- [ ] Verify backup freshness

## Review Process

Reviewers will check:
- [ ] Change clearly documented
- [ ] Impact analysis complete
- [ ] Testing comprehensive
- [ ] Rollback plan detailed
- [ ] Security reviewed
- [ ] Performance impact assessed
- [ ] Team notifications included

## Reporting Issues

Found a network issue? Include:
- Affected VLAN/service
- When issue started
- Symptoms observed
- Impact on users
- Relevant logs
- Network topology context

## Questions?

- **Docs**: See [documentation/](documentation/)
- **Team**: network-team@zsel.opole.pl
- **Help**: Create an issue

---

Thank you for strengthening our network! 🌐
