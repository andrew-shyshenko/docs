##### FAQ

**Q: Can I associate specific storage node to specific tenant?**  

**A: Yes.**
For such solution you should: 
1) Create LVM Volume Group with specific name ('hdd_for_tenant1', e.g.)
2) In OSA deploy additional Storage node(node3) with such parameters:
```buildoutcfg
storage_hosts:
  node2:
    ip: 172.29.236.12
    container_vars:
      cinder_backends:
        limit_container_types: cinder_volume
        hdd:
          volume_group: 'hdd'
          volume_driver: cinder.volume.drivers.lvm.LVMVolumeDriver
          volume_backend_name: 'LVM_hdd'
          iscsi_ip_address: 172.29.244.12
  node3:
    ip: 172.29.236.13
    container_vars:
      cinder_backends:
        limit_container_types: cinder_volume
        hdd_for_tenant1:
          volume_group: 'hdd_for_tenant1'
          volume_driver: cinder.volume.drivers.lvm.LVMVolumeDriver
          volume_backend_name: 'LVM_hdd_for_tenant1'
          iscsi_ip_address: 172.29.244.13
   ...
```
3) As result in Cinder config /etc/cinder/cinder.conf you should find:
```
...
enabled_backends=hdd,hdd_for_tenant1
# All given backend(s)
[hdd]
iscsi_ip_address=172.29.244.12
volume_driver=cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name=LVM_hdd
volume_group=hdd

[hdd_for_tenant1]
iscsi_ip_address=172.29.244.13
volume_driver=cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name=LVM_hdd_for_tenant1
volume_group=hdd_for_tenant1
...

```
4) Create volume type
```
$ cinder type-create <volume type name>
$ cinder type-key <volume type name> set volume_backend_name=<backend name>
```
Working examples below:
```
$ cinder type-create --is-public false tenant1
$ cinder type-key tenant1 set volume_backend_name=LVM_hdd_for_tenant1
```
Adds volume type access for the given project
```
$ cinder type-access-add --volume-type <volume_type> --project-id <project_id>
$ cinder type-access-add --volume-type tenant1 --project-id 23520427-cdcb-41b6-bc28-f927531d5bef
```
Print access information about the given volume type
```
$ cinder type-access-list --volume-type <volume_type>
$ cinder type-access-list --volume-type tenant1
```

Links: https://docs.openstack.org/cli-reference/cinder.html

5) Restrict permissions via quotas:
Check quotas:
```
$ cinder quota-show <tenant_ID>
$ cinder quota-show 0ece405bde4b412fb689a6b072f2744a
```
Update quotas:
```
$ cinder quota-update --volumes <volume_count> --volume-type <volume_type_name> <tenant_ID>
$ cinder quota-update --volumes 100 --volume-type hdd_for_tenant1 0ece405bde4b412fb689a6b072f2744a
```
Links: https://docs.openstack.org/admin-guide/blockstorage-multi-backend.html

**Limitations:**
 
1) All user can see all types (backends in our case) of Cinder volumes if you use public cinder types (which is default).

2) When user creates in Horizon volume with 'no volume type', Cinder creates volume on first available backend 
(omitting all restrictions, quotas by 'type')  



