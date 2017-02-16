Neutron DVR. North/South packet flow (from VM to world)
=======================================================

Environment
^^^^^^^^^^^

- 3 controllers
- 2 computes
- DVR enabled
- public_ip range: 172.30.89.2 - 172.30.89.125
- floating_ip range: 172.30.89.126 - 172.30.89.254
- internal ip pool in router: 10.0.111.0/24
- compute nodes without public ips.

.. code::

    Controllers                                  Computes

    node-13                                         node-12
    +--------+                                  +-----------+
    | br-ex  |                                  |           |
    | 172.30.89.6                               |           |
    |        |                                  |           |
    +--------+                                  |           |
                                                |           |
    node-14                                     |           |
    +--------+                                  |           |
    | br-ex  |                                  +-----------+
    | 172.30.89.4
    |        |
    +--------+                                      node-16
                                                +-----------+
    node-15                                     |           |
    +--------+                                  |           |
    | br-ex  |                                  |           |
    | 172.30.89.5                               |           |
    |        |                                  |           |
    +--------+                                  |           |
                                                +-----------+


Before creating floating ip (compute node-12):
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code::

    root@node-12:~# ip netns
    qrouter-517ba029-f811-4af8-9cfd-6cef0babd1d1
    fip-cb1f1abb-443c-49df-b863-ba4cebf10d18


.. code::

    root@node-12:~# ip netns exec fip-cb1f1abb-443c-49df-b863-ba4cebf10d18 ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    22: fg-73de653f-c4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether fa:16:3e:6f:c7:c5 brd ff:ff:ff:ff:ff:ff
        inet 172.30.89.129/17 brd 172.30.127.255 scope global fg-73de653f-c4
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe6f:c7c5/64 scope link
           valid_lft forever preferred_lft forever


.. code::

    root@node-12:~# ip netns exec qrouter-517ba029-f811-4af8-9cfd-6cef0babd1d1 ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    47: qr-22378d26-ea: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether fa:16:3e:b0:bc:f0 brd ff:ff:ff:ff:ff:ff
        inet 10.0.111.1/24 brd 10.0.111.255 scope global qr-22378d26-ea
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:feb0:bcf0/64 scope link
           valid_lft forever preferred_lft forever


Notice!
~~~~~~~

On another compute node we have the same interface with the same ip and mac!

.. code::

    root@node-16:~# ip netns exec qrouter-517ba029-f811-4af8-9cfd-6cef0babd1d1 ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    60: qr-22378d26-ea: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether fa:16:3e:b0:bc:f0 brd ff:ff:ff:ff:ff:ff
        inet 10.0.111.1/24 brd 10.0.111.255 scope global qr-22378d26-ea
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:feb0:bcf0/64 scope link
           valid_lft forever preferred_lft forever


After VM with floating floating ip (compute node-12):
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Associate fip 172.30.89.140 (local VM's ip 10.0.111.9)

.. code::

    root@node-12:~# ip netns exec fip-cb1f1abb-443c-49df-b863-ba4cebf10d18 ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    3: fpr-517ba029-f: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 6a:41:cd:d4:c5:c8 brd ff:ff:ff:ff:ff:ff
        inet 169.254.106.115/31 scope global fpr-517ba029-f
           valid_lft forever preferred_lft forever
        inet6 fe80::6841:cdff:fed4:c5c8/64 scope link
           valid_lft forever preferred_lft forever
    22: fg-73de653f-c4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether fa:16:3e:6f:c7:c5 brd ff:ff:ff:ff:ff:ff
        inet 172.30.89.129/17 brd 172.30.127.255 scope global fg-73de653f-c4
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe6f:c7c5/64 scope link
           valid_lft forever preferred_lft forever


.. code::

    root@node-12:~# ip netns exec qrouter-517ba029-f811-4af8-9cfd-6cef0babd1d1 ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: rfp-517ba029-f: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 16:e4:28:ad:c7:62 brd ff:ff:ff:ff:ff:ff
        inet 169.254.106.114/31 scope global rfp-517ba029-f
           valid_lft forever preferred_lft forever
        inet 172.30.89.140/32 brd 172.30.89.140 scope global rfp-517ba029-f
           valid_lft forever preferred_lft forever
        inet6 fe80::14e4:28ff:fead:c762/64 scope link
           valid_lft forever preferred_lft forever
    47: qr-22378d26-ea: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
        link/ether fa:16:3e:b0:bc:f0 brd ff:ff:ff:ff:ff:ff
        inet 10.0.111.1/24 brd 10.0.111.255 scope global qr-22378d26-ea
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:feb0:bcf0/64 scope link
           valid_lft forever preferred_lft forever


So we see new interfaces in namespaces: "fpr" and "rfp"


Information from neutron cli and mysql-database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code::

    mysql> select * from dvr_host_macs;
    +-----------------------+-------------------+
    | host                  | mac_address       |
    +-----------------------+-------------------+
    | node-16.example.local | fa:16:3f:72:23:87 |
    | node-14.example.local | fa:16:3f:96:84:70 |
    | node-12.example.local | fa:16:3f:ac:96:0e |
    | node-15.example.local | fa:16:3f:cb:12:c9 |
    | node-13.example.local | fa:16:3f:ec:2f:d4 |
    +-----------------------+-------------------+
    5 rows in set (0.00 sec)


.. code::

    mysql> select * from ml2_dvr_port_bindings;
    +--------------------------------------+-----------------------+--------------------------------------+----------+------------------------------------------------+-----------+---------+--------+
    | port_id                              | host                  | router_id                            | vif_type | vif_details                                    | vnic_type | profile | status |
    +--------------------------------------+-----------------------+--------------------------------------+----------+------------------------------------------------+-----------+---------+--------+
    | 22378d26-ea9f-454b-afba-c79096da4877 | node-12.example.local | 517ba029-f811-4af8-9cfd-6cef0babd1d1 | ovs      | {"port_filter": true, "ovs_hybrid_plug": true} | normal    |         | ACTIVE |
    | 22378d26-ea9f-454b-afba-c79096da4877 | node-13.example.local | NULL                                 | ovs      | {"port_filter": true, "ovs_hybrid_plug": true} | normal    |         | ACTIVE |
    | 22378d26-ea9f-454b-afba-c79096da4877 | node-14.example.local | 517ba029-f811-4af8-9cfd-6cef0babd1d1 | ovs      | {"port_filter": true, "ovs_hybrid_plug": true} | normal    |         | ACTIVE |
    | 22378d26-ea9f-454b-afba-c79096da4877 | node-15.example.local | 517ba029-f811-4af8-9cfd-6cef0babd1d1 | ovs      | {"port_filter": true, "ovs_hybrid_plug": true} | normal    |         | ACTIVE |
    | 22378d26-ea9f-454b-afba-c79096da4877 | node-16.example.local | 517ba029-f811-4af8-9cfd-6cef0babd1d1 | ovs      | {"port_filter": true, "ovs_hybrid_plug": true} | normal    |         | ACTIVE |
    +--------------------------------------+-----------------------+--------------------------------------+----------+------------------------------------------------+-----------+---------+--------+



.. code::

    root@node-14:~# neutron router-list
    +--------------------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+
    | id                                   | name     | external_gateway_info                                                                                                                                                                     | distributed | ha    |
    +--------------------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+
    | 517ba029-f811-4af8-9cfd-6cef0babd1d1 | router04 | {"network_id": "cb1f1abb-443c-49df-b863-ba4cebf10d18", "enable_snat": true, "external_fixed_ips": [{"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.126"}]} | True        | False |
    +--------------------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+-------+


.. code::

    root@node-14:~# neutron l3-agent-list-hosting-router 517ba029-f811-4af8-9cfd-6cef0babd1d1
    +--------------------------------------+-----------------------+----------------+-------+----------+
    | id                                   | host                  | admin_state_up | alive | ha_state |
    +--------------------------------------+-----------------------+----------------+-------+----------+
    | 2c544bff-8934-4bbe-b045-6bb39f1813e7 | node-14.example.local | True           | :-)   |          |
    +--------------------------------------+-----------------------+----------------+-------+----------+


.. code::

    root@node-14:~# neutron router-port-list 517ba029-f811-4af8-9cfd-6cef0babd1d1
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+
    | id                                   | name | mac_address       | fixed_ips                                                                            |
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+
    | 22378d26-ea9f-454b-afba-c79096da4877 |      | fa:16:3e:b0:bc:f0 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.1"}    |
    | 784e8801-08ba-4509-b708-e7eebf58274e |      | fa:16:3e:a1:c3:68 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.2"}    |
    | cf417cb0-5342-4426-afa0-d6cf03647bd2 |      | fa:16:3e:eb:e6:79 | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.126"} |
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+

.. code::

    root@node-14:~# neutron subnet-list
    +--------------------------------------+----------------------------+---------------+----------------------------------------------------+
    | id                                   | name                       | cidr          | allocation_pools                                   |
    +--------------------------------------+----------------------------+---------------+----------------------------------------------------+
    | 1f06f938-b005-4be2-af8d-2447cfda8716 | admin_floating_net__subnet | 172.30.0.0/17 | {"start": "172.30.89.126", "end": "172.30.89.254"} |
    | 7978f97e-a2b3-4457-b9c6-8620cc953332 | admin_internal_net__subnet | 10.0.111.0/24 | {"start": "10.0.111.2", "end": "10.0.111.254"}     |
    +--------------------------------------+----------------------------+---------------+----------------------------------------------------+


.. code::

    root@node-14:~# neutron floatingip-list
    +--------------------------------------+------------------+---------------------+--------------------------------------+
    | id                                   | fixed_ip_address | floating_ip_address | port_id                              |
    +--------------------------------------+------------------+---------------------+--------------------------------------+
    | 5ea07d36-5d76-4594-a98d-75d5fcf45cf5 | 10.0.111.9       | 172.30.89.140       | a7884fb1-3573-4cdc-a48e-1f37c95ace81 | vm on node-12
    | 7e2f82cd-df1f-4aa5-bdb4-1f087df720ef | 10.0.111.10      | 172.30.89.139       | ddb1b5af-f1ad-436a-b751-1df48d870229 | vm on node-16
    +--------------------------------------+------------------+---------------------+--------------------------------------+


.. code::

    root@node-14:~# neutron port-list
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+
    | id                                   | name | mac_address       | fixed_ips                                                                            |
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+
    | 22378d26-ea9f-454b-afba-c79096da4877 |      | fa:16:3e:b0:bc:f0 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.1"}    | on controllers and computes 'ip netns exec qrouter-517ba029-f811-4af8-9cfd-6cef0babd1d1 ip a'
    | 3e0cce18-f497-4288-b538-1dacd842a230 |      | fa:16:3e:d6:34:ac | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.139"} | vm on node-16
    | 539a8281-16f6-425a-b656-81595fe87de8 |      | fa:16:3e:00:04:01 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.11"}   | on controller-1 'ip netns exec qdhcp-a6632dff-6612-412e-90c7-935156af71a9 ip a'
    | 73de653f-c4b7-41aa-adbf-478fc9ba5cd3 |      | fa:16:3e:6f:c7:c5 | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.129"} | neutron_agent_gateway on node-12
    | 784e8801-08ba-4509-b708-e7eebf58274e |      | fa:16:3e:a1:c3:68 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.2"}    | on controller 'ip netns exec snat-517ba029-f811-4af8-9cfd-6cef0babd1d1 ip a'
    | 8d3ff3f5-af3f-4c94-8c3e-dde2c5ff04b2 |      | fa:16:3e:d7:bb:d1 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.4"}    | on controller-2 'ip netns exec qdhcp-a6632dff-6612-412e-90c7-935156af71a9 ip a'
    | a7884fb1-3573-4cdc-a48e-1f37c95ace81 |      | fa:16:3e:66:15:3c | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.9"}    | vm on node-12
    | cf417cb0-5342-4426-afa0-d6cf03647bd2 |      | fa:16:3e:eb:e6:79 | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.126"} | virt_router
    | ddb1b5af-f1ad-436a-b751-1df48d870229 |      | fa:16:3e:54:1f:a3 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.10"}   | vm on node-16
    | e2da35d6-a1b1-4bb1-99fb-d7225341579f |      | fa:16:3e:a0:dd:f9 | {"subnet_id": "7978f97e-a2b3-4457-b9c6-8620cc953332", "ip_address": "10.0.111.3"}    | ? reserved_dhcp_port (inactive)
    | e81e3941-8336-434e-980c-fc0a5d3a7d5b |      | fa:16:3e:bc:eb:65 | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.131"} | neutron_agent_gateway on node-16
    | f1140802-f094-4380-9170-8646e1c4f22f |      | fa:16:3e:71:21:c6 | {"subnet_id": "1f06f938-b005-4be2-af8d-2447cfda8716", "ip_address": "172.30.89.140"} | vm on node-12
    +--------------------------------------+------+-------------------+--------------------------------------------------------------------------------------+


.. code::

    mysql> select * from ports;
    +----------------------------------+--------------------------------------+------+--------------------------------------+-------------------+----------------+--------+-------------------------------------------------------------------------------+--------------------------------------+----------+------------------+
    | tenant_id                        | id                                   | name | network_id                           | mac_address       | admin_state_up | status | device_id                                                                     | device_owner                         | dns_name | standard_attr_id |
    +----------------------------------+--------------------------------------+------+--------------------------------------+-------------------+----------------+--------+-------------------------------------------------------------------------------+--------------------------------------+----------+------------------+
    | 3f1a1ad039a548d0a14e2af29d073ea7 | 22378d26-ea9f-454b-afba-c79096da4877 |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:b0:bc:f0 |              1 | ACTIVE | 517ba029-f811-4af8-9cfd-6cef0babd1d1                                          | network:router_interface_distributed | NULL     |               35 |
    |                                  | 3e0cce18-f497-4288-b538-1dacd842a230 |      | cb1f1abb-443c-49df-b863-ba4cebf10d18 | fa:16:3e:d6:34:ac |              1 | N/A    | 7e2f82cd-df1f-4aa5-bdb4-1f087df720ef                                          | network:floatingip                   | NULL     |              157 |
    | 3f1a1ad039a548d0a14e2af29d073ea7 | 539a8281-16f6-425a-b656-81595fe87de8 |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:00:04:01 |              1 | ACTIVE | dhcpc274a333-337f-545f-b9d3-8ef831729124-a6632dff-6612-412e-90c7-935156af71a9 | network:dhcp                         | NULL     |              156 |
    |                                  | 73de653f-c4b7-41aa-adbf-478fc9ba5cd3 |      | cb1f1abb-443c-49df-b863-ba4cebf10d18 | fa:16:3e:6f:c7:c5 |              1 | ACTIVE | c706e875-6e71-48e5-a6df-0b3e46902e22                                          | network:floatingip_agent_gateway     | NULL     |              110 |
    |                                  | 784e8801-08ba-4509-b708-e7eebf58274e |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:a1:c3:68 |              1 | ACTIVE | 517ba029-f811-4af8-9cfd-6cef0babd1d1                                          | network:router_centralized_snat      | NULL     |               38 |
    | 3f1a1ad039a548d0a14e2af29d073ea7 | 8d3ff3f5-af3f-4c94-8c3e-dde2c5ff04b2 |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:d7:bb:d1 |              1 | ACTIVE | dhcp06aaf457-6456-5c27-99ab-b3bc774dc821-a6632dff-6612-412e-90c7-935156af71a9 | network:dhcp                         | NULL     |               44 |
    | 3f1a1ad039a548d0a14e2af29d073ea7 | a7884fb1-3573-4cdc-a48e-1f37c95ace81 |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:66:15:3c |              1 | ACTIVE | 8c7cdd3c-9274-41ca-bc35-97616b957962                                          | compute:nova                         | NULL     |              152 |
    |                                  | cf417cb0-5342-4426-afa0-d6cf03647bd2 |      | cb1f1abb-443c-49df-b863-ba4cebf10d18 | fa:16:3e:eb:e6:79 |              1 | ACTIVE | 517ba029-f811-4af8-9cfd-6cef0babd1d1                                          | network:router_gateway               | NULL     |               32 |
    | 3f1a1ad039a548d0a14e2af29d073ea7 | ddb1b5af-f1ad-436a-b751-1df48d870229 |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:54:1f:a3 |              1 | ACTIVE | dd88b998-5bcd-46ac-ab52-399fe1fb5af6                                          | compute:nova                         | NULL     |              154 |
    | 3f1a1ad039a548d0a14e2af29d073ea7 | e2da35d6-a1b1-4bb1-99fb-d7225341579f |      | a6632dff-6612-412e-90c7-935156af71a9 | fa:16:3e:a0:dd:f9 |              1 | BUILD  | reserved_dhcp_port                                                            | network:dhcp                         | NULL     |               41 |
    |                                  | e81e3941-8336-434e-980c-fc0a5d3a7d5b |      | cb1f1abb-443c-49df-b863-ba4cebf10d18 | fa:16:3e:bc:eb:65 |              1 | ACTIVE | 6c3f5125-41a0-4fc1-bc93-b7eac27b37a7                                          | network:floatingip_agent_gateway     | NULL     |              140 |
    |                                  | f1140802-f094-4380-9170-8646e1c4f22f |      | cb1f1abb-443c-49df-b863-ba4cebf10d18 | fa:16:3e:71:21:c6 |              1 | N/A    | 5ea07d36-5d76-4594-a98d-75d5fcf45cf5                                          | network:floatingip                   | NULL     |              161 |
    +----------------------------------+--------------------------------------+------+--------------------------------------+-------------------+----------------+--------+-------------------------------------------------------------------------------+--------------------------------------+----------+------------------+
    12 rows in set (0.00 sec)


We see from table that DVR requires 2 floating ip: for gateway in floating net and for agent on each compute node.


Find out with VM's macs with floating ip
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Find mac of VM1 (fip 172.30.89.140, local ip 10.0.111.9)

.. code::

    $ arping -I br-public -c 1 172.30.89.140
    ARPING 172.30.89.140
    42 bytes from fa:16:3e:6f:c7:c5 (172.30.89.140): index=0 time=771.461 msec


We see this is mac of neutron_agent_gateway on node-12 with ip 172.30.89.129.
If we create new Vm in the node-12, arping shows the same mac.


Find mac with tcpdump. From Vm make 'ping 172.30.0.11'. On 172.30.0.11 run 'tcpdump -e -i any icmp'.
Get mac 'fa:16:3e:6f:c7:c5'


Find route from VM to internet

.. code::

    $ traceroute 8.8.8.8
    traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 46 byte packets
    1  10.0.111.1 (10.0.111.1)  1.645 ms  0.065 ms  0.979 ms
    2  169.254.106.115 (169.254.106.115)  1.003 ms  1.179 ms  1.214 ms
    3  172.30.0.1 (172.30.0.1)  0.944 ms  0.088 ms  1.013 ms


As we see traffic goes through fpr-517ba029-f (169.254.106.115, in namespace fip-cb1f1abb-443c-49df-b863-ba4cebf10d18 in compute node-12)

From official documentation (http://docs.openstack.org/mitaka/networking-guide/scenario-dvr-ovs.html)

.. figure:: ../images/scenario-dvr-flowns2.png
   :alt: traffic_flow
   :align: center


The following steps involve a packet inbound from the external network to an instance on compute node 1:

    The external interface forwards the packet to the Open vSwitch external bridge br-ex. The packet contains destination IP address F1.
    The Open vSwitch external bridge br-ex forwards the packet to the fg interface (1) in the floating IP namespace fip. The fg interface responds to any ARP requests for the instance floating IP address F1.
    The floating IP namespace fip routes the packet (2) to the distributed router namespace qrouter using DVR internal IP addresses DA1 and DA2. The fpr interface (3) contains DVR internal IP address DA1 and the rfp interface (4) contains DVR internal IP address DA2.
    The floating IP namespace fip forwards the packet to the rfp interface (5) in the distributed router namespace qrouter. The rfp interface also contains the instance floating IP address F1.
    The iptables service (6) in the distributed router namespace qrouter performs DNAT on the packet using the destination IP address. The qr interface (7) contains the project network gateway IP address TG.
    The distributed router namespace qrouter forwards the packet to the Open vSwitch integration bridge br-int.
    The Open vSwitch integration bridge br-int forwards the packet to the Linux bridge qbr.
    Security group rules (8) on the Linux bridge qbr handle firewalling and state tracking for the packet.
    The Linux bridge qbr forwards the packet to the instance tap interface (9).

The following steps involve a packet outbound from an instance on compute node 1 to the external network:

    The instance 1 tap interface (9) forwards the packet to the Linux bridge qbr. The packet contains destination MAC address TG1 because the destination resides on another network.
    Security group rules (8) on the Linux bridge qbr handle state tracking for the packet.
    The Linux bridge qbr forwards the packet to the Open vSwitch integration bridge br-int.
    The Open vSwitch integration bridge br-int forwards the packet to the qr interface (7) in the distributed router namespace qrouter. The qr interface contains the project network gateway IP address TG.
    The iptables service (6) performs SNAT on the packet using the rfp interface (5) as the source IP address. The rfp interface contains the instance floating IP address F1.
    The distributed router namespace qrouter (2) routes the packet to the floating IP namespace fip using DVR internal IP addresses DA1 and DA2. The rfp interface (4) contains DVR internal IP address DA2 and the fpr interface (3) contains DVR internal IP address DA1.
    The fg interface (1) in the floating IP namespace fip forwards the packet to the Open vSwitch external bridge br-ex. The fg interface contains the project router external IP address TE.
    The Open vSwitch external bridge br-ex forwards the packet to the external network via the external interface.


Links
^^^^^

- http://docs.openstack.org/mitaka/networking-guide/
- https://assafmuller.com/category/dvr/
- https://www.openstack.org/assets/presentation-media/Openstack-kilo-summit-DVR-Architecture-20141030-Master-submitted-to-openstack.pdf
- https://platform9.com/support/distributed-virtual-routing-dvr-neutron/