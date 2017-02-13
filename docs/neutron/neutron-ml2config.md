Before you can consume the Neutron API and build networking resources, a
networking plugin must be defi ned and confi gured. The remainder of this chapter
is dedicated to providing instructions on installing and confi guring the ML2 plugin
and LinuxBridge or Open vSwitch drivers and respective network agents.
Prior to the ML2 plugin and a common database schema, the LinuxBridge and Open
vSwitch plugins could not easily interoperate with one another. When using the
ML2 plugin, it is possible to use both the LinuxBridge and Open vSwitch drivers
simultaneously within an environment but on different hosts. Some agents, such
as the L3 and DHCP agents, require a network driver to be defi ned as part of their
confi guration. These changes will be highlighted as part of the confi guration outlined
in this chapter.

#####ML2 plugin confi guration options

The ML2 plugin was installed in the previous chapter, and its confi guration fi le
located at /etc/neutron/plugins/ml2/ml2_conf.ini must be confi gured before
Neutron networking services can be used.
The ml2_conf.ini fi le is broken into confi guration blocks and contains the following
commonly used options:
```
[ml2]
type drivers
mechanism drivers
tenant_network_types
[ml2_type_flat]
flat_networks
[ml2_type_vlan]
network_vlan_ranges
[ml2_type_gre]
tunnel_id_ranges
[ml2_type_vxlan]
vni_ranges
[securitygroup]
firewall_driver
enable_security_group
enable_ipset
```

#####Type drivers

Type drivers describe the type of networks that can be created and implemented
by mechanism drivers. Type drivers included with the ML2 plugin include local ,
flat , vlan , gre , and vxlan . Not all mechanism drivers can implement all types
of networks, however. The Open vSwitch driver can support all fi ve, but the
LinuxBridge driver lacks support for GRE networks.
Update the ML2 confi guration fi le on all hosts and add the following type drivers:
```
[ml2]
...
type_drivers = local,flat,vlan,gre,vxlan
```

#####Mechanism drivers

Mechanism drivers are responsible for implementing the networks described by the
type driver. Mechanism drivers included with the ML2 plugin are linuxbridge ,
openvswitch , and l2population .
Update the ML2 confi guration fi le on all hosts and add the following mechanism
drivers.
For LinuxBridge, you can use the following code:
```
[ml2]
...
mechanism_drivers = linuxbridge,l2population
```
For Open vSwitch, you can use the following code:
```
[ml2]
...
mechanism_drivers = openvswitch,l2population
```

#####Tenant network types

The tenant_network_types confi guration option describes the type of networks
that a tenant can create. When using the Open vSwitch driver, the supported tenant
network types are flat , vlan , local , gre , vxlan , and none . The LinuxBridge driver
supports the same type drivers, with the exception of gre .
The confi guration option takes values in an ordered list, such as vxlan,vlan . In this
example, when a tenant creates a network, Neutron will automatically provision a
VXLAN network. When all available VXLAN VNI's are allocated, Neutron allocates
a network of the next type in the list. In this case, a VLAN network would be
allocated. When all available networks are allocated, tenants can no longer create
networks.

Update the ML2 confi guration fi le on all hosts and add the following tenant network
types to the [ml2] section:
```
[ml2]
...
tenant_network_types = vlan,vxlan
```
If at any time you wish to change the value of tenant_network_types , edit the
plugin confi guration fi le accordingly on all nodes and restart the neutron-server
service.

#####Tenant Networks Configuration

######Flat

In the following example, the physnet1 interface is configured to support a flat
network. Multiple interfaces can be defi ned using a comma-separated list, as follows:
```
flat_networks = physnet1,physnet2
```
Note! 'physnet1' is only lable of real phisical interface. It maps to interface in 
'openvswitch_agent.ini' or 'linuxbridge_agent.ini'  

######VLAN

Update the ML2 configuration file on all hosts and add the following network VLAN
ranges configuration to the [ml2_type_vlan] section:
```
[ml2_type_vlan]
...
network_vlan_ranges = physnet2:30:33
```
In the following example, VLANs 30 through 33 are available for tenant network
allocation:
`network_vlan_ranges = physnet2:30:33`
Noncontiguous VLANs can be allocated using a comma-separated list, as follows:
`network_vlan_ranges = physnet2:30:33,physnet2:51:55`
In this installation, the physnet2 provider label is used with VLANs 30 through 33
available for tenant allocation.

When the number of the available VLAN IDs reaches zero, tenants
will no longer be able to create VLAN networks.

######GRE

When GRE networks are created, each network is assigned a unique segmentation
ID that is used to encapsulate traffic.

The tunnel_id_ranges confi guration option is a comma-separated list of ID ranges
that are available for tenant network allocation when tunnel_type is set to gre.

The tunnel_id_ranges option supports noncontiguous IDs using a comma-
separated list as follows:
```
tunnel_id_ranges = 1:1000,2000:2500
```

######VXLAN

When VXLAN networks are created, each network is assigned a unique
segmentation ID, which is used to encapsulate traffic.

Update the ML2 confi guration fi le on all hosts and add the following VNI range to
the [ml2_type_vxlan] section:
```
[ml2_type_vxlan]
...
vni_ranges = 1:1000
```

The vni_ranges option supports noncontiguous IDs using a comma-separated list.

######Firewall driver
The firewall_driver confi guration option instructs Neutron to use a particular
fi rewall driver for security group functionality. There may be different fi rewall
drivers confi gured based on the mechanism driver in use.
Update the ML2 confi guration fi le on all hosts and defi ne the appropriate firewall_
driver confi guration option in the [securitygroup] section.
For LinuxBridge, use the following code:
```
[securitygroup]
...
firewall_driver = neutron.agent.linux.iptables_firewall.
IptablesFirewallDriver
```
For Open vSwitch, run the following code:
```
[securitygroup]
...
firewall_driver = neutron.agent.linux.iptables_firewall.
OVSHybridIptablesFirewallDriver
```
If you do not want to use a fi rewall and want to disable the application of the
security group rules, set firewall_driver to neutron.agent.firewall.
NoopFirewallDriver .

######Enable security group

The enable_security_group confi guration option instructs Neutron to enable or
disable the security group API. The option is set to true by default.
If at any time this confi guration option is updated, you must restart the neutron-
server service for the changes to take effect.

######Enable ipset

The enable_ipset confi guration option instructs Neutron to enable or disable the
ipset extension for iptables that allows for the creation of fi rewall rules that match
entire sets of addresses at once. The use of ipsets makes lookups very effi cient
compared to the traditional linear lookups. The option is set to true by default.

Note! If at any time this configuration option is updated, you must restart the neutron-
server service for the changes to take effect.