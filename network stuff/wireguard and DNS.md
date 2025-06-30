DNS and Wireguard is strange.

This is how it behaves when using a pretty basic config (IPv4 tunnel and IPv4 interface address).



| DNS server   |  search domain  |  allowed IPs |  can resolve local IPv4? |  can resolve local IPv6? | IPv4 internet is reachable?   | IPv6 internet is reachable? | remote DNS can resolve vm1.company.local  | remote DNS can resolve vm1 |
| ------------ | -------------- | ------------ | ------------------------ | ------------------------ | ----------------------------- | --------------------------- | ----------------------------------------- | ------------------------------- | 
| 10.0.10.1     |  set           | 10.0.10.0/24 |   no                     | no                       | Yes (not using tunnel)        | Yes (not using tunnel)      |  yes                                       | no                               |
| 10.0.10.1        |  not set       | 10.0.10.0/24 |   no                     | no                       | Yes (not using tunnel)        | Yes (not using tunnel)      |  yes                                       | no                               |
| not set      |  not set       | 10.0.10.0/24 |   Yes                     | Yes                       | Yes (not using tunnel)       | Yes (not using tunnel)      |  no                                       | no                               |
| 10.0.10.1      |  set           | 0.0.0.0/0    |   no                     | no                       | Yes ( using tunnel)           | no                         |  yes                                       | yes                               |
|  10.0.10.1       |  not set       | 0.0.0.0/0    |   no                     | no                       | Yes ( using tunnel)           | no                          |  yes                                       | no                               |
| not set      |  not set       | 0.0.0.0/0    |   no                     | no                       | no       | no                        |  no                                       | no                               |


Conclusions:
- There is no way to get DNS working on both sides. Either only your home network or only your remote network can do DNS.
- The search domain setting does nothing if you specify the allowed IPs, only if you set it to 0.0.0.0/0
