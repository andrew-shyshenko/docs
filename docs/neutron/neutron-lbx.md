Note! While the LinuxBridge and Open vSwitch agents and drivers can coexist
in the same environment, they should not be installed and configured
simultaneously on the same host.

#####Configuring the DHCP agent to use LinuxBridge

Update the interface_driver confi guration option in the Neutron DHCP agent
configuration fi le at /etc/neutron/dhcp_agent.ini on the controller node to use
the LinuxBridge interface driver:
```
[DEFAULT]
...
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
```

#####ML2 confi guration options for LinuxBridge

Prior to ML2, the LinuxBridge plugin used its own confi guration fi le and
options. The [linux_bridge] and [vxlan] option blocks are moved to the ML2
confi guration fi le, and the most common options can be seen in the following code:
```
[linux_bridge]
...
physical_interface_mappings
[vxlan]
...
enable_vxlan
l2_population
local_ip
```

**Physical interface mappings**

The **physical_interface_mappings** configuration option describes the mapping
of an artifi cial interface name or label to a physical interface in the server. When
networks are created, they are associated with an interface label, such as physnet2 .
The physnet2 label is then mapped to a physical interface, such as eth2 , by the
physical_interface_mappings option. This mapping can be observed as follows:
```
physical_interface_mappings = physnet2:eth2
```
The chosen label must be consistent between all nodes in the environment. However,
the physical interface mapped to the label may be different. A difference in
mappings is often observed when one node maps physnet2 to a 1-Gbit interface and
another maps physnet2 to a 10-Gbit interface.
More than one interface mapping is allowed, and they can be added to the list using
a comma as the separator:
```
physical_interface_mappings = physnet1:eth1,physnet2:eth2
```
In this installation, the eth2 interface will be utilized as the physical network
interface, which means that any VLAN provided for use by tenants must traverse
eth2 . The physical switch port connected to eth2 must support 802.1q VLAN
tagging if VLAN networks are to be created by tenants.
Confi gure the LinuxBridge plugin to use physnet2 as the physical interface label
and eth2 as the physical network interface by updating the ML2 confi guration fi le
accordingly on all hosts, as follows:
```
[linux_bridge]
...
physical_interface_mappings = physnet2:eth2
```

**Enable VXLAN**

To enable support for VXLAN, the enable_vxlan confi guration option must be set
to true . Update the enable_vxlan confi guration option in the [vxlan] section of
the ML2 confi guration fi le accordingly on all hosts with the following code:
```
[vxlan]
...
enable_vxlan = true
```

**L2 population**

To enable support for the L2 population driver, the l2_population confi guration
option must be set to true . Update the l2_population confi guration option in the
[vxlan] section of the ML2 confi guration fi le accordingly on all hosts using the
following code:
```
[vxlan]
...
l2_population = true
```

**Local IP**

The local_ip confi guration option specifi es the local IP address on the node that
will be used to build the VXLAN overlay between hosts when enable_vxlan is set
to true .

Update the local_ip confi guration option in the [vxlan] section of the ML2
confi guration file accordingly on all hosts.
On each controller, compute node, use the specific ip-address:
```
[vxlan]
...
local_ip = 192.168.0.2
```

Change 192.168.0.2 to current ip address of specific node.

######Restarting services

If the OpenStack confi guration fi les have been modifi ed to use LinuxBridge as the
networking plugin, certain services must be restarted for the changes to take effect.
The following service should be restarted on all hosts in the environment:
```
# service neutron-linuxbridge-agent restart
```
Also, the following services should be restarted on the controller node:
```
# service nova-api restart
# service neutron-server restart
# service neutron-dhcp-agent restart
```
Then, the following service should be restarted on the compute nodes:
```
# service nova-compute restart
```

#####Verifying LinuxBridge agents

To verify that the LinuxBridge network agents on all nodes have properly checked
in, issue the neutron agent-list command on the controller node:
```
# neutron agent-list
+--------------------------------------+--------------------+-----------------------------------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host                                    | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+-----------------------------------------+-------------------+-------+----------------+---------------------------+
| 3466303f-a7df-4194-99eb-61aef2b7ec68 | DHCP agent         | node1-neutron-agents-container-b635f114 | nova              | :-)   | True           | neutron-dhcp-agent        |
| 4a971c63-3906-4e6c-8599-494c748c8a9a | Metadata agent     | node1-neutron-agents-container-b635f114 |                   | :-)   | True           | neutron-metadata-agent    |
| 57e13ad5-5e6e-4bd0-b532-228afaa7286e | Metering agent     | node1-neutron-agents-container-b635f114 |                   | :-)   | True           | neutron-metering-agent    |
| 5cf1c43f-9fea-4920-840c-04e594e0d43e | Metadata agent     | ubuntu                                  |                   | :-)   | True           | neutron-metadata-agent    |
| 868ae4c0-dc93-4346-9a64-18d492242546 | Linux Bridge agent | ubuntu                                  |                   | :-)   | True           | neutron-openvswitch-agent |
| c88a45d8-1c11-4eec-aa69-07bb1127421a | L3 agent           | node1-neutron-agents-container-b635f114 | nova              | :-)   | True           | neutron-l3-agent          |
| c9d868fd-fd7b-453f-b47d-ded962b85fa3 | L3 agent           | ubuntu                                  | nova              | :-)   | True           | neutron-l3-agent          |
+--------------------------------------+--------------------+-----------------------------------------+-------------------+-------+----------------+---------------------------+

```

The LinuxBridge agents on the controller and compute nodes should be visible in
the output with a smiley face under the alive column. If a node is not present or the
status is XXX , troubleshoot agent connectivity issues by observing the log messages
found in /var/log/neutron/neutron-linuxbridge-agent.log on the
respective host.