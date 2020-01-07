# Role Name

This role installs and configures `isc-dhcp-server` on Debian.


## Requirements

Debian 10


## Role Variables


| Name                                  |     Required/Default     | Description                                                                         |
| ------------------------------------- | :----------------------: | ----------------------------------------------------------------------------------- |
| `isc_dhcp_server_interfaces`          | :heavy_multiplication_x: | List of IPv4 interfaces to listen on                                                |
| `isc_dhcp_server_authoritative`       |         `False`          | Whether this DHCP is authoritative for its subnets                                  |
| `isc_dhcp_server_default_lease_time`  | :heavy_multiplication_x: | Default lease time to offer DHCP clients                                            |
| `isc_dhcp_server_max_lease_time`      | :heavy_multiplication_x: | Maximum lease time to allow DHCP clients                                            |
| `isc_dhcp_server_routers`             | :heavy_multiplication_x: | List of routers to advertise                                                        |
| `isc_dhcp_server_ntp_servers`         | :heavy_multiplication_x: | List of NTP servers to advertise                                                    |
| `isc_dhcp_server_domain_name_servers` | :heavy_multiplication_x: | List of name servers to advertise                                                   |
| `isc_dhcp_server_domain_search`       | :heavy_multiplication_x: | List of search domains to advertise                                                 |
| `isc_dhcp_server_other_flags`         | :heavy_multiplication_x: | List of other flags to set in the global config section                             |
| `isc_dhcp_server_other_options`       | :heavy_multiplication_x: | List of other options to set in the global config section using `option`            |
| `isc_dhcp_server_failover`            | :heavy_multiplication_x: | Dict to configure failover, see below                                               |
| `isc_dhcp_server_subnets`             |    :heavy_check_mark:    | Dict of subnets, see below                                                          |
| `isc_dhcp_server_hosts`               | :heavy_multiplication_x: | Dict of [host declarations](#host-declarations) so as to assign reserved addresses. |

### Host Declarations

In a host declaration, the key is a MAC address and the value is another dictionary containing the
keys `host` and `ip`.
Example:

```yaml
  "00:0a:95:9d:68:16":
    host: examplehostname
    ip: 192.168.10.16
```

The host with that MAC address will be statically assigned the given IP address.
The value of the `host` key is only used
[as per the isc-dhcp-server documentation](https://kb.isc.org/docs/isc-dhcp-41-manual-pages-dhcpdconf):

> The DHCP server determines the client’s hostname by first looking for a
> ddns-hostname configuration option, and using that if it is present. If no
> such option is present, the server looks for a valid hostname in the FQDN
> option sent by the client. If one is found, it is used; otherwise, if the
> client sent a host-name option, that is used. Otherwise, if there is a host
> declaration that applies to the client, the name from that declaration will be
> used. If none of these applies, the server will not have a hostname for the
> client, and will not be able to do a DNS update.

### Failover

DHCP failover allows to split the DHCP database between *exactly two hosts*.
Both hosts should have the same subnet as well the failover configs (excluding host-specific settings, of course).
I recommend reading the failover section in the [dhcpd.conf man page](https://linux.die.net/man/5/dhcpd.conf) before deploying this feature.
`isc_dhcp_server_failover` is a dict with the following options:

| Name                       | Required/Default                | Description                                                                                                                                              |
|----------------------------|:-------------------------------:|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `peer`                     | :heavy_check_mark:              | Hostname of the failover peer (if this is defined, failover is configured)                                                                               |
| `primary`                  | `False`                         | Whether this host is the primary one of both                                                                                                             |
| `address`                  | :heavy_check_mark:              | IP Address to listen on for failover communication                                                                                                       |
| `port`                     | :heavy_check_mark:              | Port to use for both communication directions (the man page recommends `647` or `847`                                                                    |
| `peer_address`             | :heavy_check_mark:              | IP address of the peer for failover communication                                                                                                        |
| `max_response_delay`       | :heavy_check_mark:              | Number of BNDUPD messages sent before an ACK must be received (`10` seems to work for the man page authors)                                              |
| `split`                    | :heavy_multiplication_x:        | Ratio of how many IP addresses should be managed by the primary (Integer between `0` and `255`. The greater the value, the larger the primary ownership) |
| `mclt`                     | :heavy_check_mark: (on primary) | Maximum client lead time (time that one peer may renew a lease before contacting the other peer)                                                         |
| `load_balance_max_seconds` | :heavy_multiplication_x:        | Cutoff in seconds after which load balancing gets disabled                                                                                               |
| `max_lease_misbalance`     | :heavy_multiplication_x:        | Misbalance in percent of both address pools, after which balancing is started                                                                            |
| `max_lease_ownership`      | :heavy_multiplication_x:        | This value permits a small (percentage) skew in the lease balance of a percentage of the total number of free state leases.                              |
| `min_balance`              | :heavy_multiplication_x:        | Minimum age of leases to consider in rebalancing                                                                                                         |
| `max_balance`              | :heavy_multiplication_x:        | Maximum age of leases to consider in rebalancing                                                                                                         |


### Subnets

`isc_dhcp_server_subnets` is a dict of subnets to configure.
You should configure every subnet that is attached to the DHCP server, but only set the range value for subnets you are authoritative for.
The key is the network address, the value of it is a dict with the following entries:

| Name                 | Required/Default         | Description                                                                                    |
|----------------------|:------------------------:|------------------------------------------------------------------------------------------------|
| `netmask`            | :heavy_check_mark:       | The netmask of this subnet (not in CIDR format)                                                |
| `range_begin`        | :heavy_multiplication_x: | First IP address to assign                                                                     |
| `range_end`          | :heavy_multiplication_x: | Last IP address to assign                                                                      |
| `routers`            | :heavy_multiplication_x: | List of routers to advertise                                                                   |
| `broadcast_address`  | :heavy_multiplication_x: | Broadcast IP address                                                                           |
| `default_lease_time` | :heavy_multiplication_x: | Default lease time to use in thie subnet (if not defined, the global value from above is used) |
| `max_lease_time`     | :heavy_multiplication_x: | Maximum lease time to use in thie subnet (if not defined, the global value from above is used) |
| `other_flags`        | :heavy_multiplication_x: | List of other flags to set in this subnet config section                                       |
| `other_options`      | :heavy_multiplication_x: | List of other options to set in this subnet config section using `option`                      |


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

- [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
