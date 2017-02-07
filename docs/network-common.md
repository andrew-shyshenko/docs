#####ARP

#####DHCP

#####VLAN

#####GRE

#####Namespaces

![Packet encapsulation](img/gre-vxlan.png)

#####MTU


#####Interfaces

OpenStack operates with such interfaces: 

- Physical interfaces
- Tap devices
- VLAN interfaces
- VXLAN interfaces
- Linux bridges
- Virtual Ethernet cables
- OVS bridges
- OVS patch ports

A **physical interface** represents an interface on the host that is plugged into physical
network hardware. Physical interfaces are often labeled eth0 , eth1 , em0 , em1 , and so
on, and vary depending on the host operating system.

A **tap interface** is created and used by a hypervisor, such as QEMU/KVM, to
connect the guest operating system in a virtual machine instance to the host. These
virtual interfaces on the host correspond to a network interface inside the guest
instance. An Ethernet frame sent to the tap device on the host is received by the guest
operating system, and the frames received from a guest operating system are injected
into the host network stack.

A **VLAN interface** can be created using iproute2 commands or the traditional vlan
utility and 8021q kernel module. A VLAN interface is often labeled ethX.<vlan>
and is associated with its respective physical interface, ethX. Linux supports 
802.1q VLAN tagging through the use of virtual VLAN interfaces.

A **VXLAN interface** is a virtual interface that is used to encapsulate and forward
traffi c based on the parameters confi gured during the creation of an interface, such
as a VXLAN Network Identifi er (VNI) and VXLAN Tunnel End Point (VTEP). The
function of a VTEP is to encapsulate the virtual machine instance traffi c within an IP
header across an IP network. Traffi c is segregated from other VXLAN traffi c using an
ID provided by the VNI. The instances themselves are unaware of the outer network
topology providing connectivity between VTEPs.

A **Linux bridge** is a virtual interface that connects multiple network interfaces. In
Neutron, a bridge usually includes a physical interface and one or more virtual or
tap interfaces. Linux bridges are a form of virtual switches.

Virtual Ethernet, or **veth**, cables are virtual interfaces that mimic network patch cables. An Ethernet
frame sent to one end of a veth cable is received by the other end, much like a real
network patch cable. Neutron also makes use of veth cables to make connections
between various network resources, including namespaces and bridges.

An **OVS bridge** behaves like a physical switch, only one that is virtualized. Neutron
connects the interfaces used by DHCP or router namespaces and instance tap
interfaces to OVS bridge ports. The ports themselves can be configured much like a
physical switch port. Open vSwitch maintains information about connected devices,
including MAC addresses and interface statistics.

Open vSwitch has a built-in port type that mimics the behavior of a Linux veth cable,
but it is optimized for use with OVS bridges. When connecting two Open vSwitch
bridges, a port on each switch is reserved as a **patch port**. Patch ports are configured
with a peer name that corresponds to the patch port on the other switch.

**Note!** 
Open vSwitch patch ports are used to connect Open vSwitch bridges to each other,
while Linux veth interfaces are used to connect Open vSwitch bridges to Linux
bridges or Linux bridges to other Linux bridges.

#####Configuring the bridge interface

In this instruction, the eth1 physical network interface will be utilized for bridging
purposes. On the controller and compute nodes, configure the eth1 interface within
the /etc/network/interfaces/eth1.cfg file, as follows:
```
    auto eth1
    iface eth1 inet manual
```    
Close and save the file and bring the interface up with the following command:
```
# ip link set dev eth1 up
or 
# ifup eth1
```
Confirm that the interface is in an UP state using the ip link show dev eth1:
```sh
# link show dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
```


#####TUN/TAP

1. TUN/TAP provides packet reception and transmission for user space programs. 
It can be seen as a simple Point-to-Point or Ethernet device, which,
instead of receiving packets from physical media, receives them from 
user space program and instead of sending packets via physical media 
writes them to the user space program. 

2. What is TUN/TAP driver used for?
As mentioned above, main purpose of TUN/TAP driver is tunneling. 
It is used by VTun (http://vtun.sourceforge.net).

Another interesting application using TUN/TAP is pipsecd
(http://perso.enst.fr/~beyssac/pipsec/), a userspace IPSec
implementation that can use complete kernel routing (unlike FreeS/WAN).

3. How does Virtual network device actually work ? 
Virtual network device can be viewed as a simple Point-to-Point or
Ethernet device, which instead of receiving packets from a physical 
media, receives them from user space program and instead of sending 
packets via physical media sends them to the user space program. 

Let's say that you configured IPX on the tap0, then whenever 
the kernel sends an IPX packet to tap0, it is passed to the application
(VTun for example). The application encrypts, compresses and sends it to 
the other side over TCP or UDP. The application on the other side decompresses
and decrypts the data received and writes the packet to the TAP device, 
the kernel handles the packet like it came from real physical device.

4. What is the difference between TUN driver and TAP driver?
TUN works with IP frames. TAP works with Ethernet frames.

This means that you have to read/write IP packets when you are using tun and
ethernet frames when using tap.

5. What is the difference between BPF and TUN/TAP driver?
BPF is an advanced packet filter. It can be attached to existing
network interface. It does not provide a virtual network interface.
A TUN/TAP driver does provide a virtual network interface and it is possible
to attach BPF to this interface.

6. Does TAP driver support kernel Ethernet bridging?
Yes. Linux and FreeBSD drivers support Ethernet bridging. 


#####FAQ

1. How many VLANs can I create?
 
    You can create 4094 because only 12 bits are used for VLANs in 802.1q, 
    so you can only use VLANs from 0-4095: 4096 different VLANs minus 2 (0 and 4095 are reserved).
    
    For more information read: https://en.wikipedia.org/wiki/IEEE_802.1Q#Frame_format

2. How many GRE tunnels can I create?
    
    65 535...

3. Can I bind two DHCP-servers to one interface?

    No.
    
4. How to change name of interface in Ubuntu 16.04 from ens3 to eth0?

    Edit /etc/default/grub:
    
```sh
# sed -i 's/^GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"/' /etc/default/grub
# update-grub
# sed -i 's/ens3/eth0/' /etc/network/interfaces
```

4. Can I add 2 bridges to 1 interface?

    No.
    
5. Can I add 1 interface to 2 bridges?

    No.
    
5. Can I add 2 interfaces to 1 bridge?

    Yes.
    
6. How to allow traffic through host?

    Enable forwarding on the host:
```
# echo 1 > /proc/sys/net/ipv4/ip_forward
```
To make this permanent add in /etc/sysctl.conf the line `net.ipv4.ip_forward = 1` and immediately enable with `sysctl -p /etc/sysctl.conf` 