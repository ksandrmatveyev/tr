# step2
### HowToIDid:
1. Up 3 VMs manually:
  - app1 with 1 internal network ("intnet1")
  - app2 with 1 internal network ("intnet2")
  - gateway with 1 nat nework and 2 internal networks ("intnet1" and "intnet2")
2. Install `isc-dhcp-server` (dhcp server) and `bind9` (dns server)
3. Add files from needed folders (called as VMs) to each VM
4. Restart VMs (better)
5. Run `iptables.sh` on Gateway VM

### Explanation:
 - On Gateway VM:
   - defined 2 internal network interfaces (/etc/network/interfaces):
     - static eth1 (192.168.10.50/26)
     - static eth2 (10.0.0.45/26)
   - after installing `isc-dhcp-server`
     - enabled serving DHCP requests on eth1 and eth2 (/etc/default/isc-dhcp-server)
     - configured dhcpd configuration (/etc/dhcp/dhcpd.conf):
       - define subnet 192.168.10.0/26 with router and dns as 192.168.10.50 (eth1)
       - define subnet 10.0.0.0/26 with router and dns as 10.0.0.45 (eth2)
       - domain name for both `training.com`
   - after installing `bind9`
     - added forwarders, if our dns can't give responce (/etc/bind/named.conf.options)
     - aded listen-on for 2 internal networks (/etc/bind/named.conf.options)
     - define 2 zones (`forward` and `reverse or arpa`) in /etc/bind/named.conf.local
     - added conf file for forward zone, where defined SOA, NS and A records for `gateway.training.com` domain
     - added conf file for reverse zone, where defined SOA, NS and PTR records for `gateway.training.com` domain
   - enable ip forwarding (added /etc/sysctl.d/60-ipforward.conf)
   - added iptables rules for accessing to internet via nat interface (eth0) of gateway for machines in 10.0.0.0/26 (eth2) only (our App2 VM):
```
#!/bin/bash
sudo iptables -A FORWARD -i eth1 -o eth0 -j REJECT &&\
sudo iptables -A FORWARD -i eth0 -o eth1 -j REJECT &&\
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
**Note:** 
 - Rejected connections for eth1, because otherwise they had accessing, too. 
 - Didn't add rules for accepting traffic from eth2 (internal) to eth0 (nat), because it is allow by default
 - Enabled masquerading for nat interface

 - On App1:
   - added network interface, which get configuration from dhcp (/etc/network/interfaces)
 - On App2:
   - added network interface, which get configuration from dhcp (/etc/network/interfaces)

### Tree view
+---app1
|   \---etc
|       \---network
+---app2
|   \---etc
|       \---network
\---gateway
    \---etc
        +---bind
        +---default
        +---dhcp
        +---network
        \---sysctl.d
