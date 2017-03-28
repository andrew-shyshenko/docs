#### Adding Drives to Swift

Storing data on hard drives is the fundamental purpose of Swift, so adding and for‐
matting the drives is an important first step.

Swift replicates data across disks, nodes, zones, and regions, so there
is no need for the data redundancy provided by a RAID controller.
Parity-based RAID, such as RAID5, RAID6, and RAID10, harms the
performance of a cluster and does not provide any additional pro‐
tection because Swift is already providing data redundancy.

###### Format and label

You should label each device so you can keep an inventory of each device in the system
and properly mount the device. To add a label, use the -L option to the mkfs.xfs
command. For instance, the following assigns the label d1 to the device /dev/sdb and
the label d2 to the device /dev/sdc:

```commandline
mkfs.xfs -f -L d1 /dev/sdb
mkfs.xfs -f -L d2 /dev/sdc
```

###### Mounting drives

The next step is to tell the operating system to attach (mount) the new XFS filesystems
somewhere on the devices so that Swift can find them and start putting data on them.
Create a directory in /srv/node/ for each device as a place to mount the filesystem. We
suggest a simple approach to naming the directories, such as using each device’s label
(d1, d2, etc):
```
mkdir -p /srv/node/d1
mkdir -p /srv/node/d2
```
For each device (dev) you wish to mount, you will use the mount command:
```
// _mount -t xfs -o noatime,nodiratime,logbufs=8 -L_ <dev label> <dev dir>
```
Run the command for each device:
```
mount -t xfs -o noatime,nodiratime,logbufs=8 -L d1 /srv/node/d1
mount -t xfs -o noatime,nodiratime,logbufs=8 -L d2 /srv/node/d2
```
Next, give the swift user ownership to the directories:
```
chown -R swift:swift /srv/node
```
And add disks to /etc/fstab for auto-mounting after reboot.


#### Creating the Ring Builder Files

You need to create a minimum of three files: account.builder, container.builder, and
object.builder. You will also create a builder file for each user-defined storage policy that
you are implementing in your cluster.

As their names indicate, these are the files that are used to build the rings that the Swift
processes use for data storage and retrieval.

The builder files are some of the most critical components of the cluster, so we will take
a closer look at their creation.

**The create command**

Think of a builder file as a big database. It contains a record of all the storage devices in
the cluster and the values the ring-builder utility will use to create a ring.
The help menu for swift-ring-builder shows the format for the create command as:
```
swift-ring-builder (account|container|object|object-n).builder create <part_power> <replicas> <min_part_hours>
```
So the commands we will be running are:
```
swift-ring-builder <min_part_hours> account.builder create <part_power> <replicas> 
swift-ring-builder <min_part_hours> container.builder create <part_power> <replicas>
swift-ring-builder <min_part_hours> object.builder create <part_power> <replicas>
swift-ring-builder <min_part_hours> object-n.builder create <part_power> <replicas>
```
The three parameters the create command takes are:

- __part_power__: Determines the number of partitions created in the storage cluster
- __replicas__: Specifies how many replicas you would like stored in the cluster
- __min_part_hours__: Specifies the frequency at which a replica is allowed to be moved

These are critical parameters for a cluster, so let’s dig a bit more deeply into each of these
configuration settings before using them.

Note! Don’t change the partition power after the cluster is created. A change will result in the 
reshuffling of every single thing in the cluster. That much data in flight could create vulnerabilities, 
and the data will be inaccessible for some period of time before the system is repartitioned and the data is 
moved to where it should be.


###### Running the create command

Now we are ready to create the builder files.
The commands we will be running are:
```
swift-ring-builder account.builder create <part_power> <replicas> <min_part_hours>
swift-ring-builder container.builder create <part_power> <replicas> <min_part_hours>
swift-ring-builder object.builder create <part_power> <replicas> <min_part_hours>
swift-ring-builder object-n.builder create <part_power> <replicas> <min_part_hours>
```
Now that we understand more about the values needed for part_power , replicas , and
min_part_hours , please select your values and run the create command. Here for ac‐
count, container, and object we use a part_power value of 17, replicas value of 3, and
min_part_hours value of 1.

```commandline
cd /etc/swift
swift-ring-builder account.builder create 17 3 1
swift-ring-builder container.builder create 17 3 1
swift-ring-builder object.builder create 17 3 1
```

You’ll now see account.builder, container.builder, object.builder, object-1.builder, and
object-2.builder in the directory. There should also be a backup directory, appropriately
named backups.

Note! 
Don’t lose the builder files!
Be sure not to lose these builder (.builder) files! Keep regular back‐
ups of them.

###### Adding Devices to the Builder Files

Our next goal is to add the drives, their logical groupings (region, zone), and weight.
This part of the configuration serves two purposes:

- It marks failure boundaries for the cluster by placing each node into a region and
zone.
- It describes how much data Swift should put on each device in the cluster by giving
it a weight.

With the builder files created, we can now add devices to them. For each node, the
storage device in /srv/node will need an entry added to the builder file(s). If a drive is
meant to be dedicated storage for a certain data type, it does not have to be added to the
other ring builder files. For example, if you have a pair of SSDs that will handle all
account and container data, you would add those drives to the account builder file and
to the container builder file but not to the object builder file(s).
Entries for devices are added to each file with this format:
```commandline
swift-ring-builder add account.builder <region><zone>-<IP>:6002/<device> <weight>
swift-ring-builder add container.builder <region><zone>-<IP>:6002/<device> <weight>
swift-ring-builder add object.builder <region><zone>-<IP>:6002/<device> <weight>
```

Now that we’ve covered the parameters you need to specify when adding devices, we’ll
go ahead and add our devices to the builder files.
The following adds the first device ( d1 ) to the builder files:
```
swift-ring-builder account.builder add r1z1-127.0.0.1:6002/d1 100
swift-ring-builder container.builder add r1z1-127.0.0.1:6001/d1 100 
swift-ring-builder container.builder add r1z1-127.0.0.1:6001/d1 100
```
Then we add the next device ( d2 ) to the builder files:
```
swift-ring-builder account.builder add r1z1-127.0.0.1:6002/d2 100
swift-ring-builder container.builder add r1z1-127.0.0.1:6001/d2 100 
swift-ring-builder container.builder add r1z1-127.0.0.1:6001/d2 100
```
Continue this pattern for each device that is being added to the cluster.

###### Building the Rings

Once all the devices have been added, let’s create the ring files. The rebalance command
creates an actual ring file used by Swift to determine where data is placed. The command
will need to be run for each ring that you wish to create.

Once the rings are created, they need to be copied to the /etc/swift directory of every
node in the cluster. No other action will be needed after that, because Swift automatically
detects new ring data every 15 seconds.

Note!
When a node that was down comes back online, be sure to provide
any new ring files to it. If the node has a different (old) ring file, it
will think that data isn’t where it should be and will do its part to move
it back to where it’s “supposed” to be.

Here we run the commands to create the four rings:
```commandline
cd /etc/swift
swift-ring-builder account.builder rebalance
swift-ring-builder container.builder rebalance
swift-ring-builder object.builder rebalance
```

As the command runs, each device signs up for its share of the partitions. During the
build process, each device declares its relative weight in the system. The rebalancing
process takes all the partitions and assigns them to devices, making sure that each device
is subscribed according to its weight. That way a device with a weight of 200 gets twice
as many partitions as a device with a weight of 100.

This rebalancing process also takes into account recent changes to the cluster. The
builder files keep track of things such as when partitions were last moved and where
partitions are currently located. This ensures that partitions do not move around if
they’ve already been moved recently (based on what min_part_hours is set to) and that
partitions are not reassigned to new devices unnecessarily.

Once the rebalance is complete, you should see the following additional files in /etc/swift:

- account.ring.gz
- container.ring.gz
- object.ring.gz

Copy these files to the /etc/swift directory of every node in the cluster.