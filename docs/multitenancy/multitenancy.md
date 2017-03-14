### OpenStack Multi-Tenant Isolation (Nova and Cinder)

<p><strong>The quick wrap up of the required steps are:</strong></p>

<ul>
<li>Add additional Nova Scheduling filters</li>
<li>Configure and enable multiple-storage back ends for Cinder</li>
<li>Create volume type(s)</li>
<li>Create the new tenant(s)</li>
<li>Create new host aggregate for tenant</li>
<li>Add compute nodes to the host aggregate</li>
<li>Update host aggregate metadata to include tenant filter</li>
<li>Create custome flavor to include tenant filter</li>
<li>Apply volume type quotas</li>
</ul>

<hr />

<h4 id="addadditionalnovaschedulingfilters">Add additional Nova Scheduling filters</h4>

<p>Add the additional Nova Scheduling filters to /etc/nova/nova.conf (new filters in bold).  This filter tells Nova to adhere to the host aggregate filter we will apply below.</p>

<p>scheduler_default_filters = <strong>AggregateInstanceExtraSpecsFilter,AggregateMultiTenancyIsolation,</strong> <br />
RetryFilter,AvailabilityZoneFilter,RamFilter,ComputeFilter, <br />
ComputeCapabilitiesFilter,ImagePropertiesFilter, <br />
ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,AggregateCoreFilter</p>

<hr />

<h4 id="configureandenablemultiplestoragebackends">Configure and enable multiple-storage back ends</h4>

<p>To enable a multiple-storage back ends, you must set the enabled_backends flag in the cinder.conf file.  Below is an example of the block to add to the cinder.conf file.</p>

<pre><code>enabled_backends=US-lvm,UK-lvm
[US-lvm]
volume_driver=cinder.volume.drivers.lvm.LVMISCSIDriver
volume_backend_name=LVM_iSCSI
volume_group=cinder-volumes

[UK-lvm]
volume_driver=cinder.volume.drivers.lvm.LVMISCSIDriver
volume_backend_name=LVM_iSCSI_2
volume_group=cinder-volumes2
</code></pre>

<hr />

<h4 id="createvolumetype">Create volume type</h4>

<pre><code>$ cinder type-create &lt;volume type name&gt;

$ cinder type-key &lt;volume type name&gt; set volume_backend_name=&lt;backend name&gt;
</code></pre>

<p>Working examples below:</p>

<pre><code>$ cinder type-create netapp_US
$ cinder type-key netapp_US set volume_backend_name=LVM_iSCSI
</code></pre>

<hr />

<h4 id="createtenantandhostaggregate">Create tenant and host aggregate</h4>

<p>Create new tenant:</p>

<pre><code>$ keystone tenant-create --name=&lt;name&gt; --description="&lt;tenant_description&gt;"
</code></pre>

<p><em>Take note of the ID of the tenant just created.</em></p>

<p>Create new aggregate for new tenant:</p>

<pre><code>$ nova aggregate-create &lt;name&gt;
</code></pre>

<p><em>Take note of the ID of the aggregate just created.</em></p>

<hr />

<h4 id="addhoststonewhostaggregate">Add hosts to new host aggregate</h4>

<p>Add hosts to aggregate, one command for each host.  Use the ID from the aggregate just created above.</p>

<pre><code>$ nova aggregate-add-host &lt;aggregate_name&gt; &lt;host_name&gt;
</code></pre>

<p>Example:</p>

<pre><code>$ nova aggregate-add-host &lt;aggregate_name&gt; compute01
$ nova aggregate-add-host &lt;aggregate_name&gt; compute02
$ nova aggregate-add-host &lt;aggregate_name&gt; compute03
</code></pre>

<hr />

<h4 id="updatehostaggregatemetadata">Update host aggregate metadata</h4>

<p>This step adds the tenant ID filter to the host aggregate, which will tell Nova Scheduler to restrict access to this host aggregate based on the tenant ID supplied.</p>

<p>Update aggregate metadata to include tenant ID filter:</p>

<pre><code>$ nova aggregate-set-metadata &lt;aggregate_ID&gt; filter_tenant_id=&lt;tenant_ID&gt;
</code></pre>

<hr />

<h4 id="createnewflavortoincludetenantidfilter">Create new flavor to include tenant ID filter</h4>

<pre><code>$ nova flavor-create &lt;flavor name&gt; &lt;id&gt; &lt;ram&gt; &lt;disk&gt; &lt;vcpus&gt;

$ nova flavor-key &lt;flavor name&gt; set filter_tenant_id=&lt;tenant ID&gt;
</code></pre>

<hr />

<h4 id="applytenantquotaforvolumetypes">Apply tenant quota for volume types</h4>

<p>New within the Icehouse release is the ability to define block storage based volume types using Cinder.  These volume types can be tied to separatly defined Cinder backends.  The process on how to do that can be found here: <a href="/web/20160407124442/https://wiki.openstack.org/wiki/Cinder-multi-backend">https://wiki.openstack.org/wiki/Cinder-multi-backend</a></p>

<p>Update default tenant quota to allow and/or restrict a particular volume type.</p>

<p>To allow access to a volume type:</p>

<pre><code>$ cinder quota-update --volumes 100 --volume-type netapp_US &lt;tenant_ID&gt;
</code></pre>

<p>To restrict access to a volume type:</p>

<pre><code>$ cinder quota-update --volumes 0 --volume-type netapp_US &lt;tenant_ID&gt;
</code></pre>

<hr />

<blockquote>
  <p>Per the OpenStack blueprint on Multi-Tenant Isolation…
  If a compute node does not belong to any aggregate, all tenants can create instances on it. <br />
  Also, if a compute node belongs to an aggregate that does not have defined the metadata filtertenantid, all tenants can create instances on it.</p>
</blockquote>

<p>In other words, if you apply the Multi-Tenant Isolation filter, you must have all compute nodes in an host aggregate with the filtertenantid metadata defined.</p>