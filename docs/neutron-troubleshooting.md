
- grep TRACE in /var/log/neutron/*

```
# grep -E -i "error|trace" /var/log/neutron/openvswitch-agent.log
# grep -E -i "error|trace" /var/log/neutron/server.log
```

- tcpdump -i eth0 -n ip proto gre

- tcpdump -envi br-int

- tcpdump -i eth0 -n arp or icmp

- tcpdump -i any -n icmp

- ip netns e qrouter-UUID tcpdump - i qr-662sdf64 icmp

For linux bridge, use the following:
```
brctl show – shows the configuration of the bridges on the machine
brctl show <bridge name> – shows the configuration for specific bridge
```

For openvswitch:
```
ovs-vsctl show - shows the configuration of the bridges on the machine
ovs-ofctl show – shows datapaths
ovs-ofctl dump-flows – dump all the flows installed on the machine
ovs-ofctl dump-flows br-tun – dump all the flows on br-tun
ovs-ofctl dump-flows br-tun table=21 – dump all the flows on br-tun in specific table
```


######FAQ

1. Can I use DVR with Linux Bridge?

    No. You can use DVR only with OVS. This is technical limitationof Linux bridge.
    
    