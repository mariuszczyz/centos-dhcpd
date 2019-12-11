# CentOS & Fedora DHCP Server Installation and Configuration Ansible Role

This role configures a basic DHCP server which can be used as:

- regular primary DHCP server on a local network
- or a temporary DHCP service used specifically for the purpose of allowing
PXE boot clients to get temporary IP addresses while performing
an unattended (or attended) CentOS & Fedora OS installation. Or other tasks
that require PXE boot capabilities.

The current way it is configured in the `templates/dhcpd.conf.j2`
allows for both functionalities listed above.
With additional custom configuration changes it can be tweaked to
fit any end user needs.

To make it behave properly and not interfere with other DHCP services
running on the network it can serve IP addresses to clients
based on either:

- full MAC address to limit per a single device
- partial vendor specific MAC address prefix to limit to a group
  of devices (example: VirtualBox)
- specific string contained in the hostname

Setting the dynamic IP range in `DHCP_DYNAMIC_IP_RANGE` variable and
client pools outside of the current DHCP range on your local network
or on its own seperate VLAN will eliminate any potential conflicts
even if limiting by hostname or vendor MAC address.

Note: this functionality has only been tested in a lab environment,
use at your own risk in production. Actually, don't ever use this in production.

I have successfully used this in a lab environments with no conflicts with the
primary DHCP service as long as the configuration is done properly and limits on
MAC addresses are set carefully. Limiting the DHCP range and monitoring logs for
ongoing DHCP leases helps in detecting issues quickly as well.

The worst case scenario is one of the existing clients will renew its IP address
with this DHCP server. If all other network settings are the same, no issues should
arrise.

Useful monitoring:

`journalctl -f` - watch the system log in real time, this is where client/server
DHCP renewal messages will be logged.

`watch -n1 "cat /var/lib/dhcpd/dhcpd.leases | grep lease` - watch current list of
active DHCP leases served by this server

## Requirements

None.

## Role Variables

Add and customize the following role variables in one of the following
locations:

Recommended:

- host_vars/{{ HOSTNAME }}.yml
- group_vars/{{ GROUPNAME }}.yml

Optional:

- {{ roles_path }}/mariuszczyz.centos-dhcpd/defaults/main.yml

Repplace `{{ HOSTNAME }}` and `{{ GROUPNAME }}` with appropriate
inventory names.

It's recommended to add all required variables to `hosts_vars` and
`group_vars`. This way they will not get overwritten next time the
original role is updated.

| Variable | Comment | Example |
| -------- | ------- | ------- |
| DOMAIN_NAME | local domain name for dhcp clients | local.localdomain |
| DOMAIN_DNS_SERVERS | internal or public DNS servers, blank space seperated | 1.1.1.1 9.9.9.9 |
| DEFAULT_LEASE_TIME | DHCP lease time expiration in seconds | 600 |
| MAX_LEASE_TIME |     max DHCP lease time expiration in seconds | 7200 |
| SUBNET_IP_ADDRESS | subnet IP address of the local network | 192.168.0.0 |
| NETMASK_IP_ADDRESS | netmask IP address of the local network | 255.255.255.0 |
| DHCP_DYNAMIC_IP_RANGE | an range of IP address to serve to clients | 192.168.0.100 192.168.0.150 |
| GATEWAY_IP_ADDRESS | local network gateway IP address | 192.168.0.1 |
| BROADCAST_IP_ADDRESS | local network broadcast IP address | 192.168.1.255 |
|  |

## Dependencies

None.

## Example Playbook

### Manual

Fetch this role from Ansible Galaxy manually:

`ansible-galaxy install mariuszczyz.centos_dhcpd`

### Not Manual

#### Galaxy

Or include this role from Ansible Galaxy via `requirements.yml`

```yaml
# requirements.yml
# Install from Ansible Galaxy
- src: mariuszczyz.centos_dhcpd
```

#### Github option

```yaml
# requirements.yml
# Install from Github repository
- src: https://www.github.com/mariuszczyz/centos_dhcpd
```

Then run this to install all dependencies from Ansible Galaxy:

`ansible-galaxy install -r requirements.yml`

### Run it

If you want to run this role individually create a new file:
`playbook.yml` (name it however you wish btw) with the following content:

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
