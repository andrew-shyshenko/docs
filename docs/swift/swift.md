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


#### Working with Swift from shell

###### About Your Swift Cluster

Before you even try to access any data, you might want to get some information about
your Swift cluster. What version of Swift is it running? What values are configured for
its parameters, such as maximum object size? What features are enabled by middleware
in its pipeline? This information is all available via the Swift cluster info API, which can
be accessed with a simple GET request to the path /info under the Swift cluster’s base
URL. For example, if your Swift cluster’s base URL is http://swift.example.com, you can
get the cluster info with a GET request to the URL http://swift.example.com/info. The
cluster info API is public; no authentication is required.
The response to an info request will be a JSON dictionary such as the following:
```
{
    "tempauth": {
        "account_acls": true
    },
    "slo": {
        "max_manifest_segments": 1000,
        "min_segment_size": 1048576,
        "max_manifest_size": 2097152
    },
    "swift": {
        "max_file_size": 5368709122,
        "account_listing_limit": 10000,
        "max_meta_count": 90,
        "max_meta_value_length": 256,
        "container_listing_limit": 10000,
        "version": "1.12.0.37.g45feab5",
        "max_meta_name_length": 128,
        "max_object_name_length": 1024,
        "max_account_name_length": 256,
        "max_container_name_length": 256
    }
}
```
A Swift client application can retrieve and parse the cluster info, and use the results to
determine how to proceed based on the cluster’s advertised capabilities. This is one way
in which Swift implements the RESTful principle of discoverability.


###### Authentication

With a few exceptions (such as the info request described in the previous section), an
authentication token (“auth token”) is a prerequisite for nearly all Swift operations. From
a cold start, the first step in a Swift operation is to authenticate and receive an auth token.
Because Swift allows administrators to plug in their auth system of choice, the auth
URL (that is, the URL to which requests for authentication are sent) is separate from
the storage URL (which specifies where data is stored) of the user’s primary account. In
order to authenticate to Swift, you need to have the cluster’s auth URL—and, of course,
your username and password.
Your authentication request then looks like this:
```
curl -i -X GET -H 'X-Auth-User: myusername' -H 'X-Auth-Key: mysecretpassword' \
https://swift.example.com/auth/v1.0
```
In this example, https://swift.example.com/auth/v1.0 is the auth URL. Generally, Swift
traffic should happen over the encrypted HTTPS connection, in order to protect all
data on the wire (credentials and tokens for an auth request, or the stored content itself).
The auth system then produces a response. If there is no problem with the Swift instal‐
lation, and your username and password are correct, then the authentication response
code is 200 OK and the response headers will return with the auth token and the storage
URL of your username’s primary account, which will look like this:
```
X-Auth-Token: AUTH_tkdc764d39fd1c40c9a293cbea142b90d7
X-Storage-Url: https://swift.example.com/v1/AUTH_myusername
[other HTTP headers]
```
Auth tokens have a configurable expiration time, with a default of 24 hours. During that
period, this auth token will prove your identity to the auth system, which can then check
to see whether you are authorized to perform the operation you are requesting. If you
present an auth token that was previously working but you receive an HTTP 401 re‐
sponse, then your auth token has likely expired and you should re-authenticate to receive
a new token.
The storage URL returned by the auth response is the root of the storage area of the
user’s primary Swift account. Account is a bit of a confusing term. In the context of Swift,
think of an account as an area where data is stored. Like a bank account, a Swift account
Overview of the Swift APImay be owned by one person or co-owned by multiple people. A given user, such as
Alice or Bob, might have access to multiple Swift accounts, because the cluster might be
set up with one account (storage area) per project team, or per service, or any of several
other strategies. So the storage URL returned by the auth request is not necessarily the
only storage URL that the user may access.
It is even more important to think of Swift accounts as storage areas when considering
Swift’s data hierarchy. At Swift’s basic level, an account has containers and a container
has objects. Unlike a traditional filesystem, however, containers can’t be placed in other
containers, and objects can’t be placed directly in an account without a container. The
URL of every object in a Swift cluster looks like https://swift.example.com/v1/myaccount/
mycontainer/myobject, with exactly one account (storage area) name, one container
name, and one object name. So it makes sense to emphasize this:
**Swift users are identities. Swift accounts are storage areas.**


###### Code Samples

In the following examples, we will demonstrate basic use of the Swift features described
in this chapter. All examples depend on your storage URL, and require you to be au‐
thenticated, so we assume you have set the environment variables TOKEN and
STORAGE_URL , perhaps with the following shell commands:
```
USERNAME=your_Swift_account_name
KEY=your_Swift_account_key_or_password
AUTH_URL=your_Swift_cluster’s_authentication_URL
TOKEN=$(curl -s -i -H "x-auth-user: $USERNAME" -H "x-auth-key: $KEY" \
"$AUTH_URL" | perl -ane '/X-Auth-Token:/ and print $F[1];')
STORAGE_URL=$(curl -s -i -H "x-auth-user: $USERNAME" -H "x-auth-key: $KEY" \
"$AUTH_URL" | perl -ane '/X-Storage-Url:/ and print $F[1];')
```
If you are using a non-TempAuth-style auth system (one that does not take its input via
the X-Auth-User and X-Auth-Key headers), adjust the token generation command ac‐
cordingly.
The following examples use cURL from the command-line shell.


###### Swift Cluster Info

Swift’s Cluster Info API is a little unusual in two regards. First, it does not require a
token or other form of authentication. Second, it is not accessed via the auth URL or
any storage URL. The URL for a Swift cluster’s info can be constructed by appending /
info to the cluster’s root URL, e.g., https://swift.example.com. Since $STORAGE_URL is the
Code Samples | 95storage URL for an account (rather than a container or object), the Swift root URL can
be generated like this:
```
SWIFT_ROOT_URL=$(echo $STORAGE_URL | perl -pe 's!/[^/]+/[^/]+$!!')
curl -i -X GET "$SWIFT_ROOT_URL/info" ; echo
```
The output of the previous cURL command might look like this:
```
{
    "swift": {
        "max_file_size": 5368709122,
        "account_listing_limit": 10000,
        "max_meta_count": 90,
        "max_meta_value_length": 256,
        "container_listing_limit": 10000,
        "version": "1.11.0.55.gdb63240",
        "max_meta_name_length": 128,
        "max_object_name_length": 1024,
        "max_account_name_length": 256,
        "max_container_name_length": 256
    },
    "tempauth": {
        "account_acls": true
    }
}
```
Note that Swift’s Cluster Info API returns its results as a JSON dictionary. Note also that
the keys of the dictionary include “swift” (for info on core Swift parameters) as well as
names of middleware packages (e.g., “tempauth”). In this way, middleware packages can
indicate their own info without requiring a change in core Swift code.


###### Links

1. https://docs.openstack.org/ops-guide/ops-quotas.html
2. https://docs.openstack.org/developer/swift/middleware.html#module-swift.common.middleware.account_quotas
3. https://docs.openstack.org/developer/swift/api/container_quotas.html
4. https://www.swiftstack.com/docs/admin/middleware/container_quotas.html
5. https://www.swiftstack.com/docs/admin/middleware/account_quotas.html