# Role Name

This role installs and configures `isc-dhcp-server` on Debian.
Currently it supports only DHCPv4 (IPv4).

## Requirements

Debian 10

## Role Variables

| Name                          |  Required/Default  | Description                                                                              |
| ----------------------------- | :----------------: | ---------------------------------------------------------------------------------------- |
| `isc_dhcp_server_interfaces`  | :heavy_check_mark: | List of IPv4 interfaces to listen on                                                     |
| `isc_dhcp_server_config_ipv4` |        `[]`        | List of top-layer configuration statements (excluding semicolon), see `man 5 dhcpd.conf` |
| `isc_dhcp_server_subnets`     |        `{}`        | Dict of [subnet declarations](#subnet-declarations)                                      |
| `isc_dhcp_server_hosts`       |        `{}`        | Dict of [host declarations](#host-declarations)                                          |

### Subnet Declarations

In a subnet declaration, the key is the router IP followed by the netmask in CIDR notation.
The value is a list of configuration statements (excluding semicolon) for that subnet.
There are certain defaults:

- If no `range` statement is given, then a range from network address plus 100 to broadcast address minus one will be configured.
- If no `option routers` statement is given, then the router IP given in the key will be configured.

Example:

```yaml
isc_dhcp_server_subnets:
  192.168.100.1/24: []
  192.168.200.1/24:
    - range 192.168.200.10 192.168.200.20
    - option routers 192.168.200.254
    - option all-subnets-local false
```

gets translated to:

```dhcpd.conf
subnet 192.168.100.0 netmask 255.255.255.0 {
  range 192.168.100.100 192.168.100.254;
  option routers 192.168.100.1;
}

subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.10 192.168.200.20;
  option routers 192.168.200.254;
  option all-subnets-local false;
}
```

### Host Declarations

In a host declaration, the key is a hostname.
The value is a list of configuration statements (excluding semicolon) for that host.
Example:

```yaml
isc_dhcp_server_hosts:
  examplehostname:
    - hardware ethernet 00:0a:95:9d:68:16
    - fixed-address 192.168.10.16
```

gets translated to:

```dhcpd.conf
host examplehostname {
  hardware ethernet 00:0a:95:9d:68:16;
  fixed-address 192.168.10.16;
}
```

The hostname is only used
[as per the isc-dhcp-server documentation](https://kb.isc.org/docs/isc-dhcp-41-manual-pages-dhcpdconf):

> The DHCP server determines the client’s hostname by first looking for a
> ddns-hostname configuration option, and using that if it is present. If no
> such option is present, the server looks for a valid hostname in the FQDN
> option sent by the client. If one is found, it is used; otherwise, if the
> client sent a host-name option, that is used. Otherwise, if there is a host
> declaration that applies to the client, the name from that declaration will be
> used. If none of these applies, the server will not have a hostname for the
> client, and will not be able to do a DNS update.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

## Author Information

- [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
- [Sebastian Hasler (haslersn)](https://github.com/haslersn) _sebastian.hasler at stuvus.uni-stuttgart.de_
