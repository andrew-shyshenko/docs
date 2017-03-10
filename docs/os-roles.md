#### Available Roles after OpenStack installation

```
admin
_member_
Member
heat_stack_owner
heat_stack_user
ResellerAdmin
swiftoperator
```

##### Keystone

`admin`
`_member_`
`Member`

Roles are custom-defined by the admin, there are no default roles or default role definition.

member
    A typical user
admin
    An administrative super user, which has full permissions across all projects and should be used with great care 
    The admin is global, not per project, so granting a user the admin role in any project gives the user administrative rights across the whole cloud.

Typical use is to only create administrative users in a single project, by convention the admin project, which is created by default during cloud setup. 
If your administrative users also use the cloud to launch and manage instances, it is strongly recommended that you use separate user accounts for 
administrative access and normal operations and that they be in distinct projects.

**Links:** https://docs.openstack.org/ops-guide/ops-users.html

_ member_ is the default role that is assigned to users when they are created. The other roles listed above seem to be from your devstack instance. 
Basically, we just have 2 default openstack roles that gets configured: admin and _ member_.
Depending on how you installed, certain tools or guides recommend using _member_ as the default role to define "membership" within a project.

However, by default Horizon expects the default role to be Member, so one can either update local_settings.py (usually in /etc/openstack_dashboard) 
to change the OPENSTACK_KEYSTONE_DEFAULT_ROLE to be "_member_", or create a "Member" role in Keystone for use in Horizon.


##### Swift

ResellerAdmin
swiftoperator

Users with the Keystone role defined in reseller_admin_role (ResellerAdmin by default) can operate on any account.
The reseller admin role has the ability to create and delete accounts (in Swift 'account' is not user identity, this is some abstract layer, namespace in storage).
To update Object Storage quotas on a project, you must have the role of ResellerAdmin in the project that the quota is being applied to.
The auth system sets the request environ reseller_request to True if a request is coming from a user with this role. This can be used by other middlewares.

Swiftoperator - user's role for Swift.

**Links:** https://docs.openstack.org/developer/swift/overview_auth.html

##### Heat

heat_stack_owner
heat_stack_user
