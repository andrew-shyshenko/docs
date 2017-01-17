<h1>Open vSwitch</h1>

---

Overview
--------



---

Install
-------

On Ubuntu:

```sh
# apt install openvswitch-common openvswitch-switch -y
```

---

Configure interfaces
--------------------

All configurations items, which are necessary to set up the Openvswitch, can be defined in /etc/network/interfaces or in separate files in /etc/network/interfaces.d/.

**Define bridge**

```sh
# ovs-vsctl add-br br-vlan
```

or in /etc/network/interfaces.d/br-vlan.cfg

```sh
auto br-vlan
allow-ovs br-vlan
iface br-vlan inet manual
  ovs_type OVSBridge
```

The configurations lines are:

* _auto_ is necessary to allow an automatic start (Ubuntu 16.04 using systemd)
* _allow-ovs_ is the marker for an Openvswitch bridge. The bridge name follows, here vmbr-int
* _iface_ is the well known /etc/network/interfaces identifier to start an interface configuration
* ovs_type defines the interface type for ovs. OVSBridge identifies an Openvswitch bridge


**Define untagged interface**

```sh
allow-br-vlan eth1
iface eth1 inet manual
    ovs_bridge br-vlan
    ovs_type OVSPort
```

or in command line:

```
# ovs-vsctl add-port br-vlan eth1
```

**Define interface with VLAN**

```sh
allow-br-mgmt eth1.236
iface eth1.236 inet manual
    ovs_bridge br-mgmt
    ovs_type OVSPort
```

**Bring up interface**

```# ifup eth1.236```

**Check vlan**

`# cat /proc/net/vlan/config`

Result:

```sh
# cat /proc/net/vlan/config
VLAN Dev name	 | VLAN ID
Name-Type: VLAN_NAME_TYPE_RAW_PLUS_VID_NO_PAD
eth1.236   | 236  | eth1
```


**Define a L2 port (untagged)**

Note! The next step attaching a port to bridge is NOT seen by Linux

In command line:

```
# ovs-vsctl add-port br-vlan l2port
```

or in /etc/network/interfaces:

```sh
# create an Openvswitch bridge
auto br-vlan
allow-ovs br-vlan
iface br-vlan inet manual
  ovs_type OVSBridge
  ovs_ports l2port

# create an untagged ovs port in vlan 240
allow-br-vlan l2port
iface l2port inet manual
  ovs_bridge br-vlan
  ovs_type OVSPort
  ovs_options tag=240
```

The configuration lines are:

* _allow-[name of the bridge to attach the port]_ defines the name of the bridge, to which the port should be attached to. In our example this is vmbr-int. The name of the port follows (here l2port)
* _iface_ is the well known /etc/network/interfaces identifier to start an interface configuration — set the config to manual
* _ovs-bridge_ holds the name of the bridge to which the port should be attached to. In our example this is vmbr-int.
* _ovs_type_ defines the interface type for ovs. OVSPort identifies an Openvswitch port not seen by the Linux Operating system
* _ovs_options_ defines additional options for the port – here define an untagged port, which is member of Vlan 444. If this option is not set, a port may use all vlan tags on the interfaces, because all ports transport vlan tags by default.
* _ovs_ports_ holds the list of all ports which should be attached to the bridge


**Define a L3 interface**

In /etc/network/interfaces:

```sh
# create an untagged ovs port in vlan 240
allow-br-vlan l3port
iface l3port inet static
  ovs_bridge br-vlan
  ovs_type OVSIntPort
  ovs_options tag=240
  address 172.29.240.254
  netmask 255.255.255.0
```


**Define a L2 port (tagged/trunking)**

In /etc/network/interfaces:

```sh
# create an tagged ovs port
allow-br-int l2taggedport
iface l2taggedport inet manual
  ovs_bridge br-int
  ovs_type OVSPort
  ovs_options trunks=236,240,244
```

---

Useful commands
---------------

```sh
ovs-appctl: for querying and controlling Open vSwitch daemon
	logging stuff
	-V

ovs-dpctl: Open vSwitch datapath management utility
	add-dp DP
	dump-dps
	dump-flows DP

ovs-ofctl: OpenFlow switch management utility
	show SWITCH
	dump-flows SWITCH
	add-flow SWITCH FLOW
	mod-port SWITCH IFACE ACT (hmm?)

ovs-vsctl: ovs-vswitchd management utility
	list-br
	add-br br0
	add-port br0 eth0
	list-ifaces br0
	emer-reset
	-> various database commands (list, get, set, add, remove etc.
```

FAQ
---

1. Why do OVS bridges have state UNKNOWN in command 'ip a'?
 
    Because OVS manages this interfaces, not Linux.
    

Links
-----

1. [All OVS manuals](http://openvswitch.org/support/dist-docs/)
2. [Boot integration of the Openvswitch in Ubuntu](http://www.opencloudblog.com/?p=240)
