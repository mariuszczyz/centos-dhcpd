# CentOS 7 Common Initial Setup Tasks

This role installs and configured a very basic, single subnet local DHCP server. 

This can be used as a drop-in replacement for a hardware router DHCP when additional functionality is needed to allow for filename PXE BOOT booting on clients. 

## Requirements

None.

## Role Variables

`DOMAIN:` this is where the domain goes, example: example.org
`DOMAIN-DNS-SERVERS:` network DNS resolvers, example: 192.168.1.100, 192.168.1.101
`DEFAULT-LEASE-TIME:` how long before the DHCP client lease expires in seconds, example: 600;
`MAX-LEASE-TIME:` maximum time before the DHCP client lease expires in seconds, example: 7200;
`SUBNET:` local network subnet, example: 192.168.1.0
`NETMASK:` local network netmask, example: 255.255.255.0
`DHCP-IP-RANGE:` pool of IP addresses reserved for DHCP clients, example: 192.168.1.50 192.168.1.99
`GATEWAY:` local network gateway, example: 192.168.1.1
`BROADCAST-ADDRESS:` local network broadcasr address, example: 192.168.1.255

## Dependencies

## Example Playbook

Fetch this role from Ansible Galaxy:

`ansible-galaxy install mariuszczyz.centos-install-additional-packages`

In playbook.yml:

```bash
- hosts: servers
  roles:
    - { role: mariuszczyz.centos-install-additional-packages, tags: ['install-additional-packages'] }
```

## License

BSD

## Author Information

Author: Mariusz Czyz  

Date: 09/2018
