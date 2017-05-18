### Task description
- Gateway machine
  - DHCP server
    - gateway network is configured manually
    - app machine networks are configured via DHCP
  - DNS server
    - gateway machine has to hostname gateway.training.com
    - gateway hostname should resolved from each private subnet
    - other hostnames have to resolve the default DNS server from public network (Interface1)
  - interfaces
    - Interface1, public network with access to internet
    - Interface2, private network 1, IP: 192.168.10.50, SUBNET MASK: 192.168.10.0/26
    - Interface3, private network 2, IP: 10.0.0.45, SUBNET MASK: 10.0.0.0/26
  - routing & netfilter
    - machines from private network 1 and 2 can access to each other
    - machines from private network 2 can access to internet
    - restrict access from network 2 to google.com and logging all attempts to do that
  - rsyslog
    - receive logs from all nodes and store in /opt/logs/<ip-or-host>/<date>.log
  - SSH
    - write a script to generate new ssh keys and deploy them to the app nodes (from the cron job, every 15 minutes)
    - script should change the ssh-keys if there are no ssh connections
- Application machines (app1 and app2)
  - DHCP client
  - interfaces
    - all network settings are received from the DHCP server
    - app1 SUBNET 192.168.10.0/26
    - app2 SUBNET 10.0.0.0/26
  - here are events to store and send on log server
    - Input http requests (from iptables)
    - ssh authorization
    - gateway is not available (write a script to determine this event and run the script from the cron job; every minute)
  - configure logrotate to compress logs and delete logs older than one week

### HowIDidIt:
1. Up 3 VMs manually:
  - app1 with 1 internal network ("intnet1")
  - app2 with 1 internal network ("intnet2")
  - gateway with 1 nat nework and 2 internal networks ("intnet1" and "intnet2")
2. Install `isc-dhcp-server` (dhcp server) and `bind9` (dns server)
3. Added files from needed folders (called as VMs) to each VM
4. Restarted VMs (better) or restart services (isc-dhcp-server, networks, bind9, rsyslog)
5. Added cronjobs (`crontab /home/vagrant/<name>_cron`)

### Explanation:
 - On Apps:
   - added network interface, which get configuration from dhcp (`/etc/network/interfaces`)
   - Added pre up iptables rules (from file `/etc/iptables.rules`), which logged all incoming http traffic:  
```
-I INPUT -i eth0 -p tcp --dport 80 -j LOG --log-prefix "iptables: HTTP IN " --log-level 4
```
Used `--log-level 4` for separating this section of log from main syslog  
   - Configured logrotate for daily backuping and compressing. Also delete log backups, which older than 1 week (`/etc/logrotate.d/rsyslog`)
   - Checked if gateway available (using `/home/vagrant/ifavailable.sh`) every minute with cronjob `home/vagrant/gateway_avail_cron`
   - Configured rsyslog for deliverying logs (from cronjob, iptables and ssh activities) to gateway rsyslog (`/etc/rsyslog.d/loghost.conf`)
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
iptables -A FORWARD -i eth0 -o eth2 -m state --state RELATED,ESTABLISHED -j ACCEPT &&\
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT &&\
iptables -A FORWARD -i eth1 -o eth0 -j REJECT &&\
iptables -A FORWARD -i eth0 -o eth1 -j REJECT &&\
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
**Note:** 
 - Rejected connections for eth1, because otherwise they had accessing, too. 
 - ~~Didn't add rules for accepting traffic from eth2 (internal) to eth0 (nat), because it is allow by default~~ 
 - Added rules for accepting traffic from eth2 (internal) to eth0 (nat) securely (Packets ingressing from the public network (eth0) are accepted for forwarding out to the private network (eth2) if and only if the ingressing public packet is related to a conversation that was established by a host on the private network)
 - Enabled masquerading for nat interface
