├───app1
│   └───etc
│       └───network
│               interfaces
│
├───app2
│   └───etc
│       └───network
│               interfaces
│
└───gateway
    │   iptables.sh
    │
    └───etc
        ├───bind
        │       forward.bind
        │       named.conf.local
        │       named.conf.options
        │       reverse.bind
        │
        ├───default
        │       isc-dhcp-server
        │
        ├───dhcp
        │       dhcpd.conf
        │
        ├───network
        │       interfaces
        │
        └───sysctl.d
                60-ipforward.conf
