# UniFi

## Discovery: DHCP Option 43

For L2 adoption you can set Option 43 in DHCP Server. To make it work in OpnSense, add Custom DHCP Option, set:

```
Number: 43
Type: String
Value: 01:04:0a:11:01:d2
```

Value is computed as follows:

```
01: sub option
04: payload length (IPv4 = 4)
0a:11:01:d2: static IPv4 as Hex
```

[Source and generator](https://tcpip.wtf/en/unifi-l3-adoption-with-dhcp-option-43-on-pfsense-mikrotik-and-others.htm)
