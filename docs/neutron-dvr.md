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


#####FAQ

* How many **public** IP does DVR require without any FIP for VM?

    1 for snat router port in controller (network) node
    1 for fip router port in each compute node (this is not fip for VM) 
