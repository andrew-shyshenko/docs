#### Available Roles after OpenStack installation

- `admin`
- `_member_`
- `Member`
- `heat_stack_owner`
- `heat_stack_user`
- `ResellerAdmin`
- `swiftoperator`



##### Keystone

- **`admin`** - an administrative super user, which has full permissions across all projects and should be used with great care

- **`_ member_`** is the default role that is assigned to users when they are created (default role in Keystone)

- **`member`** - a typical user (some old components could require this role, for example, Horizon older Grizlly release)

The admin is global, not per project, so granting a user the admin role in any project gives the user administrative rights across the whole cloud.

Typical use is to only create administrative users in a single project, by convention the admin project, which is created by default during cloud setup. 
If your administrative users also use the cloud to launch and manage instances, it is strongly recommended that you use separate user accounts for 
administrative access and normal operations and that they be in distinct projects.

Basically, we just have 2 default openstack roles that gets configured: admin and _ member_.
Depending on how you installed, certain tools or guides recommend using _ member_ as the default role to define "membership" within a project.

However, by default Horizon expects the default role to be Member, so one can either update local_settings.py (usually in /etc/openstack_dashboard) 
to change the OPENSTACK_KEYSTONE_DEFAULT_ROLE to be "_member_", or create a "Member" role in Keystone for use in Horizon.

The end result is many older environments still have both roles unless action is taken to manually remove the "Member" one 
and update the Horizon default to the new value, and I should note it probably wouldn't even be that surprising 
if there are still some deployment tools creating new environments and using a "Member" role - setting horizon to match - 
in this fashion instead of the "_member_" built-in.
 
**Links:** 
- https://docs.openstack.org/ops-guide/ops-users.html
- https://lists.gt.net/openstack/dev/38640


##### Swift

- `ResellerAdmin`

- `swiftoperator`


Users with the Keystone role defined in reseller_admin_role (ResellerAdmin by default) can operate on any account.
The **reseller admin** role has the ability to create and delete accounts (in Swift 'account' is not user identity, this is some abstract layer, namespace in storage).
To update Object Storage quotas on a project, you must have the role of ResellerAdmin in the project that the quota is being applied to.
The auth system sets the request environ reseller_request to True if a request is coming from a user with this role. This can be used by other middlewares.

**Swiftoperator** - user's role for Swift.

**Links:** 
- https://docs.openstack.org/developer/swift/overview_auth.html


##### Heat

- `heat_stack_owner`
- `heat_stack_user`

The Heat service automatically assigns the **heat_stack_user** role to users that it creates during stack deployment. 
This role has restricted access to some API calls as users created by Heat inside a stack are only for that stack.

If you wish for a user to be able to manage different stacks (for example, delete and add resources to many running stacks), 
then you must assign the **heat_stack_owner** role to this user. The heat_stack_owner role has much fewer restrictions 
on which API calls it can make to Heat.

Unfortunately, the **heat_stack_user** and **heat_stack_owner** roles will cause conflict with each other, so if you have a user 
with the heat_stack_owner role, you must not assign the heat_stack_user role to it, to avoid this conflict.


**Links:** 
- http://ibm-blue-box-help.github.io/help-documentation/heat/Getting_Started_With_Heat/
- http://hardysteven.blogspot.com/2014/04/heat-auth-model-updates-part-2-stack.html