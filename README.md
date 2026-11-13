# CentOS DHCP server Ansible role

This role installs and configures a DHCP server for a lab network. It supports
PXE boot clients with vendor and hostname based address pools. The role
installs ISC dhcpd on EL9 and Fedora 43-45. It installs Kea DHCP on EL10 and
Fedora 42, because the ISC `dhcp-server` package is not available there. The
role manages the configuration file, checks the rendered configuration, and
enables the service.

## Warning

Use this role in a lab. The role sets the server as authoritative by default
(`centos_dhcpd_authoritative: true`). An authoritative server answers all DHCP
traffic on the network. It can disrupt an existing production network. Keep
`centos_dhcpd_dynamic_ip_range` and every pool range outside the ranges of the
other DHCP servers on the network. Set `centos_dhcpd_authoritative: false` to
limit the server to its declared subnets.

## Requirements

Control node: ansible-core 2.15 or later.

Supported targets:

| Distribution | Version | DHCP server | Package | Service |
| --- | --- | --- | --- | --- |
| EL (Rocky, AlmaLinux, CentOS, RHEL) | 9 | ISC dhcpd | `dhcp-server` | `dhcpd` |
| EL (Rocky, AlmaLinux, CentOS, RHEL) | 10 | Kea DHCP | `kea` | `kea-dhcp4` |
| Fedora | 42 | Kea DHCP | `kea` | `kea-dhcp4` |
| Fedora | 43, 44, 45 | ISC dhcpd | `dhcp-server` | `dhcpd` |

The ISC dhcpd package on Fedora is best-effort. The role fails with a clear
message on any other platform.

## Variables

The role checks every variable against `meta/argument_specs.yml` and fails
early when a required variable is missing.

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `centos_dhcpd_domain_name` | str | yes | | Local domain name that the role gives to DHCP clients. |
| `centos_dhcpd_domain_dns_servers` | list of str | yes | | DNS server addresses that the role gives to DHCP clients. |
| `centos_dhcpd_default_lease_time` | int | yes | | Lease time in seconds for a new lease. |
| `centos_dhcpd_max_lease_time` | int | yes | | Maximum lease time in seconds. |
| `centos_dhcpd_subnet_ip_address` | str | yes | | Network address of the served subnet. |
| `centos_dhcpd_netmask_ip_address` | str | yes | | Netmask of the served subnet. |
| `centos_dhcpd_dynamic_ip_range` | list of str | yes | | Start and end address of the dynamic IP range. |
| `centos_dhcpd_gateway_ip_address` | str | yes | | Gateway address that the role gives to DHCP clients. |
| `centos_dhcpd_broadcast_ip_address` | str | yes | | Broadcast address that the role gives to DHCP clients. |
| `centos_dhcpd_pxelinux_filename` | str | no | `pxelinux.0` | Boot file name that PXE clients download. |
| `centos_dhcpd_next_server` | str | no | `""` | TFTP server address for PXE clients. An empty value uses the primary IPv4 address of the server. |
| `centos_dhcpd_authoritative` | bool | no | `true` | Answer all DHCP traffic on the network. |
| `centos_dhcpd_log_facility` | str | no | `local7` | Syslog facility. ISC dhcpd only. |
| `centos_dhcpd_interfaces` | list of str | no | `["*"]` | Network interfaces. Kea DHCP only. |
| `centos_dhcpd_pools` | list of dict | no | see below | Static IP pools for PXE clients. |

Each pool entry uses these keys:

| Key | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `name` | str | yes | | Class name of this pool. |
| `match_by` | str | yes | | `mac` or `hostname`. Selects the match rule. |
| `match_value` | str | yes | | MAC address prefix such as `08:00:27`, or hostname prefix such as `pxe`. |
| `match_value` | str | yes | | MAC address prefix such as `08:00:27`, or hostname prefix such as `pxe`. Use lowercase hexadecimal. |
| `next_server` | str | no | `""` | TFTP server address for this pool. An empty value uses `centos_dhcpd_next_server`. |

Default pools:

```yaml
centos_dhcpd_pools:
  - name: VBox
    match_by: mac
    match_value: "08:00:27"
    range: ["192.168.1.40", "192.168.1.41"]
    next_server: ""
  - name: SuperMicro
    match_by: mac
    match_value: "00:25:90"
    range: ["192.168.1.42", "192.168.1.43"]
    next_server: ""
  - name: pxe
    match_by: hostname
    match_value: "pxe"
    range: ["192.168.1.44", "192.168.1.45"]
    next_server: ""
```

## Renamed variables in version 2

This release renames every role variable. Update your inventory and playbooks.

| Old variable | New variable |
| --- | --- |
| `DOMAIN_NAME` | `centos_dhcpd_domain_name` |
| `DOMAIN_DNS_SERVERS` (space-separated string) | `centos_dhcpd_domain_dns_servers` (list) |
| `DEFAULT_LEASE_TIME` | `centos_dhcpd_default_lease_time` |
| `MAX_LEASE_TIME` | `centos_dhcpd_max_lease_time` |
| `SUBNET_IP_ADDRESS` | `centos_dhcpd_subnet_ip_address` |
| `NETMASK_IP_ADDRESS` | `centos_dhcpd_netmask_ip_address` |
| `DHCP_DYNAMIC_IP_RANGE` ("start end" string) | `centos_dhcpd_dynamic_ip_range` (list of two items) |
| `GATEWAY_IP_ADDRESS` | `centos_dhcpd_gateway_ip_address` |
| `BROADCAST_IP_ADDRESS` | `centos_dhcpd_broadcast_ip_address` |

Other breaking changes:

- The package changed from `dhcp` to `dhcp-server` (ISC) or `kea` (Kea).
- EL7, EL8, Fedora 30, and Fedora 31 are no longer supported.
- EL10 and Fedora 42 use Kea, so they have no `/etc/dhcp/dhcpd.conf`.
- New variables: `centos_dhcpd_pxelinux_filename`, `centos_dhcpd_next_server`,
  `centos_dhcpd_authoritative`, `centos_dhcpd_log_facility`,
  `centos_dhcpd_interfaces`, `centos_dhcpd_pools`.

## Dependencies

None.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy install mariuszczyz.centos_dhcpd
```

With a `requirements.yml` file:

```yaml
# requirements.yml
- src: mariuszczyz.centos_dhcpd
```

```yaml
# requirements.yml, from the GitHub repository
- src: https://github.com/mariuszczyz/centos-dhcpd
```

Install the file:

```bash
ansible-galaxy install -r requirements.yml
```

Example playbook:

```yaml
---
- name: Configure DHCP server
  hosts: dhcp_servers
  become: true
  vars:
    centos_dhcpd_domain_name: lab.example
    centos_dhcpd_domain_dns_servers:
      - 1.1.1.1
      - 9.9.9.9
    centos_dhcpd_default_lease_time: 600
    centos_dhcpd_max_lease_time: 7200
    centos_dhcpd_subnet_ip_address: 192.168.122.0
    centos_dhcpd_netmask_ip_address: 255.255.255.0
    centos_dhcpd_dynamic_ip_range:
      - 192.168.122.100
      - 192.168.122.110
    centos_dhcpd_gateway_ip_address: 192.168.122.1
    centos_dhcpd_broadcast_ip_address: 192.168.122.255
  roles:
    - role: mariuszczyz.centos_dhcpd
```

Run it:

```bash
ansible-playbook -i hosts playbook.yml
```

## Testing

The role uses Molecule with the Podman driver. Install the plugin once:

```bash
python3 -m pip install --user 'molecule-plugins[podman]'
```

Run the full suite:

```bash
molecule test
```

The suite creates six systemd containers and runs converge, idempotence, and
verify on each one:

- `instance-rocky9`, `instance-alma9`, `instance-cs9` (EL9, ISC dhcpd)
- `instance-rocky10` (EL10, Kea)
- `instance-fedora42` (Fedora 42, Kea)
- `instance-fedora44` (Fedora 44, ISC dhcpd)

The verify step runs `dhcpd -t` on ISC instances and `kea-dhcp4 -t` plus
`python3 -m json.tool` on Kea instances, and checks that the service is
enabled.

## License

MIT

## Author

Mariusz Czyz, mariuszczyz.com
