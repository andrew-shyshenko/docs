#####FAQ

- What should I remember when I allocate **public** network in OpenStack in OVH?

    On all the blocks that OVH.com delivers, there are 5 IP addresses reserved by the OVH configuration and should never be used.
    These are the last 4 and the first of each block.
    For example, You can set 172.16.0.0/12 with the exception of IP listed below, you must not add IN ANY CASE the following IPs 
    as the interface on your machine:
    
```
172.16.0.0 => IP Network
172.31.0.252 => IP reserved for internal use OVH
172.31.0.253 => IP reserved for internal use OVH
172.31.0.254 => IP Gateway your vrack 
172.31.0.255 => IP Broadcast
```
    
   
    See more information here:
            
        - https://www.ovh.com/us/g582.configure_an_ip_address_on_a_virtual_machine
        - http://help.ovh.com/vrack
        - http://help.ovh.com/IpAlias