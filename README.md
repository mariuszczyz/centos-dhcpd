# CentOS & Fedora DHCP Server Installation and Configuration Ansible Role

This role configures a basic DHCP server which can be used as a regular primary DHCP service on a local network or as a temporary DHCP service used specifically for the purpose of allowing PXE boot clients to get temporary IP addresses while performing an unattended (or attended) CentOS & Fedora OS installation.

The current way it is configured in the `templates/dhcpd.conf.j2` allows for both functionalities listed above. With additional custom configuration changes it can be tweaked to fit any end user needs.

To make it behave properly and not interfere with other DHCP services running on the network it can serve IP addresses to clients based on either:

- full MAC address to limit per a single device
- partial vendor specific MAC address prefix to limit to a group of devices (example: VirtualBox)
- specific string contained in the hostname

Setting the dynamic IP range in `DHCP_DYNAMIC_IP_RANGE` variable and client pools outside of the current DHCP range on your local network or on its own seperate VLAN will eliminate any potential conflicts even if limiting by hostname or vendor MAC address.

Note: this functionality has only been tested in a lab environment, use at your own risk in production. Actually, don't ever use this in production.

## Requirements

None.

## Role Variables

```yaml
DOMAIN_NAME: local domain name for dhcp clients
DOMAIN_DNS_SERVERS: internal or public DNS servers, blank space seperated
DEFAULT_LEASE_TIME: DHCP lease time expiration in seconds
MAX_LEASE_TIME: max DHCP lease time expiration in seconds
SUBNET_IP_ADDRESS: subnet IP address of the local network
NETMASK_IP_ADDRESS: netmask IP address of the local network
DHCP_DYNAMIC_IP_RANGE: an range of IP address to serve to clients
GATEWAY_IP_ADDRESS: local network gateway IP address
BROADCAST_IP_ADDRESS: local network broadcast IP address
```

## Dependencies

None.

## Example Playbook

Fetch this role from Ansible Galaxy:

`ansible-galaxy install mariuszczyz.centos_dhcpd`

Include this role from Ansible Galaxy via requirement.yml

### Galaxy option

```yaml
# Install from Ansible Galaxy
- src: mariuszczyz.centos_dhcpd
```

### Github option

```yaml
# Install from Github repository
- src: https://github.com/mariuszczyz/centos-dhcpd
```

In playbook.yml:

```yaml
- hosts: servers
  user: YOUR USER
  become: True
  
  roles:
    - { role: mariuszczyz.centos_dhcpd, tags: ['centos_dhcpd'] }
```

Run it:

`ansible-playbook -i hosts playbook.yml`

## License

BSD

## Author Information

Author: Mariusz Czyz
Date: 12/2019
mariuszczyz.com
