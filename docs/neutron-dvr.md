#####Overview

Neutron Distributed Virtual Router implements the L3 Routers across the Compute Nodes, so that tenants intra VM communication will occur without hiting the Network Node. (East-West Routing)

Also Neutron Distributed Virtual Router implements the Floating IP namespace on every Compute Node where the VMs are located. In this case the VMs with FloatingIPs can forward the traffic to the External Network without reaching the Network Node. (North-South Routing)

But Neutron Distributed Virtual Router provides the legacy SNAT behavior for the default SNAT for all private VMs. SNAT service is not distributed, it is centralized and the service node will host the service. And this is bottleneck.

######Comparing routers

```
+=================================+==================================+
|      Legacy Router              |            DVR                   |
+=================================+==================================+
| NETWORK node provides:          | COMPUTE node provides:           |
|   * IP forwarding:              |   * IP forwarding for local VMS: |
|      - inter-subnet traffic     |      - inter-subnet traffic      |
|        between VMs              |        between VMs               |
|      - Floating-ip traffic      |      - Floating-ip traffic       |
|        between VMs and external |        between VMs and external  |
|      - default SNAT traffic     |                                  |
|        from VM to external      |                                  |
|   * Metadata Agent:             |   * Metadata Agent:              |
|      - access to Nova metadata  |      - access to Nova metadata   |
|        service                  |        service                   |
|                                 |                                  |
| ISSUES:                         | ADVANTAGES:                      |
|   - Performance bottleneck      |   - Bypass Network node improves |
|                                 |     performance                  |
|   - Scalability limitation      |   - Scales with Compute farm     |
|   - Single point of failure     |   - Limited failure domain       |
|                                 |     (per compute node)           |
|                                 |                                  |
|                                 | LIMITATION:                      |
|                                 |   - Dafaul SNAT function is      |
|                                 |     still centralized            |
+---------------------------------+----------------------------------+
```

TODO(as): add pictures with traffic flows

######Configure Distributed Virtual Router (DVR)

1) Edit the ovs_neutron_plugin.ini file to change enable_distributed_routing to True:
```
enable_distributed_routing = True
l2_population = True
```
2) Edit the /etc/neutron/neutron.conf file to set the base MAC address that the DVR system uses for unique MAC allocation with the dvr_base_mac setting:
```
dvr_base_mac = fa:16:3f:00:00:00
```
3) Edit the /etc/neutron/neutron.conf file to set router_distributed to True.
```
router_distributed = True
```
4) Edit the l3_agent.ini file to set agent_mode to dvr on compute nodes for multi-node deployments:
```
agent_mode = dvr
```
On the Network node, configure dvr_snat on the distributed router: 
```
agent_mode = dvr_snat
```
5) When using a separate networking host, set agent_mode to dvr_snat. Use dvr_snat for Devstack or other single-host deployments also.
```
[ml2]
mechanism_drivers = openvswitch,l2population
```
6) In the [agent] section of the ml2_conf.ini file, set these configuration options to these values:
```
[agent]
l2_population = True
tunnel_types = vxlan
enable_distributed_routing = True
```
7) Restart the OVS L2 agent.
```
service neutron-openvswitch-agent restart
```

Note!
It is not currently possible to convert an existing non-distributed router to DVR. The router should instead be deleted and re-created as DVR


##### DVR requirements

- You must use the ML2 plug-in for Open vSwitch (OVS) to enable DVR.
- Be sure that your firewall or security groups allows UDP traffic over the VLAN, GRE, or VXLAN port to pass between the compute hosts.


##### DVR limitations

1) Distributed virtual router configurations work with the Open vSwitch Modular Layer 2 driver
2) In order to enable true north-south bandwidth between hypervisors (compute nodes), you must use public IP addresses for every compute node and enable floating IPs.
3) For now, based on the current neutron design and architecture, DHCP cannot become distributed across compute nodes. 
TODO(as): check 3rd point

##### Links

1) http://docs.openstack.org/liberty/networking-guide/scenario-dvr-ovs.html

##### FAQ

* How many **public** IP does DVR require without any FIP for VM?

    1 for snat router port in controller (network) node
    1 for fip router port in each compute node (this is not fip for VM) 
