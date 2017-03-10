#### Swift quotas

##### Set Object Storage Quotas: Prerequisites

**There are currently two categories of quotas for Object Storage:**

    - Container quotas
    Limit the total size (in bytes) or number of objects that can be stored in a single container.

    - Account quotas
    Limit the total size (in bytes) that a user has available in the Object Storage service.

To take advantage of either container quotas or account quotas, your Object Storage proxy server must have container_quotas or account_quotas (or both) added to the [pipeline:main] pipeline. Each quota type also requires its own section in the proxy-server.conf file:

```
[pipeline:main]
pipeline = catch_errors [...] slo dlo account_quotas proxy-server

[filter:account_quotas]
use = egg:swift#account_quotas

[filter:container_quotas]
use = egg:swift#container_quotas
To view and update Object Storage quotas, use the swift command provided by the python-swiftclient package. Any user included in the project can view the quotas placed on their project. To update Object Storage quotas on a project, you must have the role of ResellerAdmin in the project that the quota is being applied to.
```  
  
  
##### Set Object Storage Quotas to ADMIN account

To view account quotas placed on a project:
```
$ swift stat
   Account: AUTH_b36ed2d326034beba0a9dd1fb19b70f9
Containers: 0
   Objects: 0
     Bytes: 0
X-Timestamp: 1351050521.29419
Content-Type: text/plain; charset=utf-8
Accept-Ranges: bytes
```

To apply or update account quotas on a project:
```
$ swift post -m quota-bytes:<bytes>
```
    
For example, to place a 5 GB quota on an account:
```
$ swift post -m quota-bytes:5368709120
```

To verify the quota, run the swift stat command again:
```
$ swift stat
   Account: AUTH_b36ed2d326034beba0a9dd1fb19b70f9
Containers: 0
   Objects: 0
     Bytes: 0
Meta Quota-Bytes: 5368709120
X-Timestamp: 1351541410.38328
Content-Type: text/plain; charset=utf-8
Accept-Ranges: bytes
```

##### Set Object Storage Quotas to TENANT account

Swift requires '--os-storage-url <storage_url>' to set quotas for account
To find out it for tenant from CLI create 'openrc_tenant' file with specific credentials for tenant
Run swift stat -v:
```commandline
$ source openrc_tenant
$ swift stat -v
                    StorageURL: http://172.29.236.11:8080/v1/AUTH_889bf752c015475fa9950311c832d2f4
                    Auth Token: gAAAAABYwse6ce7zZ_g9OvoUJSN2S0vgX-uceDB9z6jCdrm3E02JC61M9HV0wo7H4iPYaKBKf4sSOT7SCceNAevEXUumq50cWYc1cdx4_Lyb_1zXoHBcEDsCIa6xT8UsjeEDX2Hzq1grkHDt_rJGYc7kMexJlsA3JS5IbHNz927GDk-O3qx0RkA
                       Account: AUTH_889bf752c015475fa9950311c832d2f4
                    Containers: 4
                       Objects: 6
                         Bytes: 59478179
Containers in policy "default": 4
   Objects in policy "default": 6
     Bytes in policy "default": 59478179
              Meta Quota-Bytes: 2368709120
   X-Account-Project-Domain-Id: default
                 Accept-Ranges: bytes
                   X-Timestamp: 1489155361.80575
                    X-Trans-Id: tx68b8fd06492f4390ac425-0058c2c7ba
                  Content-Type: text/plain; charset=utf-8
```

Get from previous output 'StorageURL'.
For set quotas your user should have role 'ReselleAdmin'.
Thus, from CLI set admin environment variables and set quota 5GB for account:
```
$ source openrc
$ swift --os-storage-url http://172.29.236.11:8080/v1/AUTH_889bf752c015475fa9950311c832d2f4 post -m quota-bytes:5368709120
```
This command return nothing if operation was successful or some error.

Check quotas for tenant account:
```commandline
$ source openrc_tenant
$ swift stat -v
                    StorageURL: http://172.29.236.11:8080/v1/AUTH_889bf752c015475fa9950311c832d2f4
                    Auth Token: gAAAAABYwse6ce7zZ_g9OvoUJSN2S0vgX-uceDB9z6jCdrm3E02JC61M9HV0wo7H4iPYaKBKf4sSOT7SCceNAevEXUumq50cWYc1cdx4_Lyb_1zXoHBcEDsCIa6xT8UsjeEDX2Hzq1grkHDt_rJGYc7kMexJlsA3JS5IbHNz927GDk-O3qx0RkA
                       Account: AUTH_889bf752c015475fa9950311c832d2f4
                    Containers: 4
                       Objects: 6
                         Bytes: 59478179
Containers in policy "default": 4
   Objects in policy "default": 6
     Bytes in policy "default": 59478179
              Meta Quota-Bytes: 5368709120
   X-Account-Project-Domain-Id: default
                 Accept-Ranges: bytes
                   X-Timestamp: 1489155361.80575
                    X-Trans-Id: tx68b8fd06492f4390ac425-0058c2c7ba
                  Content-Type: text/plain; charset=utf-8
```

Note!
In web-interface Horizon doesn't inform user when he reach quota. Horizon only returns error 'Error: Unable to upload the object.'
For more information look at logs.

To remove quota (from admin openrc):
```commandline
$ swift --os-storage-url http://172.29.236.11:8080/v1/AUTH_889bf752c015475fa9950311c832d2f4 post -m quota-bytes:
```
Check changes (from tenant openrc_tenant)
```commandline
swift stat -v
                    StorageURL: http://172.29.236.11:8080/v1/AUTH_889bf752c015475fa9950311c832d2f4
                    Auth Token: gAAAAABYws0LOw1BJHVvlb59OiDWlkIIO-CVhgeIcPUD6nf71pzhokOKrdDAZXrsIiQj1vhbGnKvBy8WaRxYrH4Tfd-vpVtTMM_VBFom0uSHg8sA7nyJnTppk7QoacS6nJs6EjfQU47h7F4Efao2G_6Aew0gWqWF8V5A576eCwlcyYvepeEV7V0
                       Account: AUTH_889bf752c015475fa9950311c832d2f4
                    Containers: 5
                       Objects: 6
                         Bytes: 5947817984
Containers in policy "default": 5
   Objects in policy "default": 6
     Bytes in policy "default": 5947817984
   X-Account-Project-Domain-Id: default
                 Accept-Ranges: bytes
                   X-Timestamp: 1489155361.80575
                    X-Trans-Id: tx66fb451467114dd0b5982-0058c2cd0b
                  Content-Type: text/plain; charset=utf-8

```
The line 'Meta Quota-Bytes: 5368709120' disappeared - success.  


###### Links

1. https://docs.openstack.org/ops-guide/ops-quotas.html
2. https://docs.openstack.org/developer/swift/middleware.html#module-swift.common.middleware.account_quotas
3. https://docs.openstack.org/developer/swift/api/container_quotas.html
4. https://www.swiftstack.com/docs/admin/middleware/container_quotas.html
5. https://www.swiftstack.com/docs/admin/middleware/account_quotas.html