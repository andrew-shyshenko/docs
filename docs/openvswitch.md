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

1.  Why do OVS bridges have state UNKNOWN in command 'ip a'?
 
    Because OVS manages this interfaces, not Linux.
    

2.  Why does each bridge need to have a port and interface with the same name marked as type "internal"?
    If I attempt to delete them, for example, it says that the port does not exist. How else are they used?
    
```
e.g.

$ovs-vsctl show
e1bbbcb1-e20d-48e5-ae89-823c1a485625
    Bridge br-tun
        Port br-tun
            Interface br-tun
                type: internal
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
    Bridge br-int
        Port br-int
            Interface br-int
                type: internal
        Port "int-br-eth1"
            Interface "int-br-eth1"
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
    Bridge "br-eth0"
        Port "phy-br-eth0"
            Interface "phy-br-eth0"
        Port "eth0"
            Interface "eth0"
        Port "br-eth0"
            Interface "br-eth0"
                type: internal
```    

The internal interface and port in each bridge is both an implementation requirement and exists for historical reasons 
relating to the implementation of Linux bridging module.
This is referred to as the "local" interface and port. In newer releases of OpenVSwitch, this message has been changed to reflect this, e.g.:

```
$ovs-vsctl del-port br-eth1
ovs-vsctl: cannot delete port br-eth1 because it is the local port for bridge br-eth1 (deleting this port requires deleting the entire bridge)
```

The purpose is to hold the IP for the bridge itself (just like some physical bridges do). This is also useful in cases where a bridge has 
a physical interface that would normally have its own IP. Since assigning a port to an IP wouldn't happen in a physical bridge, 
assigning an IP to the physical interface would be incorrect, as packets would stop at the port and not be passed across the bridge.

3.  I created a bridge and added my Ethernet port to it, using commands like these:

```
   ovs-vsctl add-br br0
   ovs-vsctl add-port br0 eth0
```

and as soon as I ran the "add-port" command I lost all connectivity through eth0. Help!


A physical Ethernet device that is part of an Open vSwitch bridge should not have an IP address. If one does, then that IP address 
will not be fully functional.
You can restore functionality by moving the IP address to an Open vSwitch "internal" device, such as the network device 
named after the bridge itself. For example, assuming that eth0's IP address is 192.168.128.5, you could run the commands below to fix up the situation:

```
   ifconfig eth0 0.0.0.0
   ifconfig br0 192.168.128.5
```
(If your only connection to the machine running OVS is through the IP address in question, then you would want to run all of these commands 
on a single command line, or put them into a script.) If there were any additional routes assigned to eth0, then you would also want to use commands 
to adjust these routes to go through br0.

There is no compelling reason why Open vSwitch must work this way. However, this is the way that the Linux kernel bridge module has always worked, 
so it's a model that those accustomed to Linux bridging are already used to. Also, the model that most people expect is not implementable without 
kernel changes on all the versions of Linux that Open vSwitch supports.

Links
-----

1. [All OVS manuals](http://openvswitch.org/support/dist-docs/)
2. [Boot integration of the Openvswitch in Ubuntu](http://www.opencloudblog.com/?p=240)
