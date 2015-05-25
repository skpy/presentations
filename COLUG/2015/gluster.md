# Gluster

[Central Ohio Linux User Group](http://colug.net/)  
Scott Merrill  
skippy@skippy.net  
[@smerrill](https://twitter.com/smerrill)  
https://github.com/skpy/presentations/blob/master/COLUG/2015/gluster.md



> GlusterFS is a scalable network filesystem. Using common off-the-shelf hardware, you can create large, distributed storage solutions for media streaming, data analysis, and other data- and bandwidth-intensive tasks. GlusterFS is free and open source software. 



## Features
* scale out storage
* data distribution, replication, striping
* POSIX filesystem
* geo-replication
* native NFS v3
* optional NFS v4 (via [NFS-Ganesha](https://github.com/nfs-ganesha/nfs-ganesha))



## Terminology
* **Trusted Storage Pool**:  Gluster servers participate in a trusted storage pool
* **Volumes**: GlusterFS volumes live within a trusted storage pool 
* **Bricks**: A volume is built from bricks. Bricks are simply directories on the GlusterFS servers



## Volume Types
* **Distributed**: distributes files throughout the bricks in the volume
* **Replicated**: replicates files across bricks in the volume
* **Striped**: stripes data across bricks in the volume


* **Distributed Striped**: stripe data across two or more nodes in the cluster
* **Distributed Replicated**: distributes files across replicated bricks in the volume
* **Distributed Striped Replicated**: distributes striped data across replicated bricks in the cluster
* **Striped Replicated**: stripes data across replicated bricks in the cluster



## Replicated Volume
![](https://raw.githubusercontent.com/gluster/glusterfs/master/doc/admin-guide/en-US/images/Replicated_Volume.png)



## Distributed Volume
![](https://github.com/gluster/glusterfs/raw/master/doc/admin-guide/en-US/images/Distributed_Volume.png)



## DHT
GlusterFS uses a [distributed hash table](http://en.wikipedia.org/wiki/Distributed_hash_table) to compute where data will be written.

Any read or write operation can be satisfied programmatically by computing the hash to find the brick(s) handling that file.


## DHT
DHT operates **solely** on the file name and number of bricks in the volume!

It is possible for the DHT hash to resolve a file write operation to a brick that has no more free space!


## DHT: renames
[Renaming files](http://blog.vorona.ca/the-way-gluster-fs-handles-renaming-the-file.html) == a new DHT hash
* may mean new hash lives on different bricks!
* may mean moving large amounts of data between bricks!


## DHT: renames
GlusterFS leaves original file alone and creates a small pointer file at the new hash location
* pointer tells GlusterFS to use the original file
* avoids need to shuffle large amounts of data
* means at least two I/O operations to resolve!


## DHT: misses
[DHT misses are expensive](http://joejulian.name/blog/dht-misses-are-expensive/)



## Bricks
* require xattr support
  * xfs or ext4
* LVM recommended
  * allows for transparent expansion


## Brick layout
Create and mount a logical volume, and then create a directory underneath that for the actual brick
```
$ sudo mkdir -p /bricks/foo
$ sudo mount /dev/mapper/bricks-foo /bricks/foo
$ sudo mkdir /bricks/foo/brick
```
Ensures that no writes will occur to the mountpoint if the mount fails



## Volumes
* can be expanded or contracted by changing number of bricks
* changing the number of bricks changes the DHT!
* may require **rebalance** operation: time and CPU expensive


## Volume Options
Many [options](https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_managing_volumes.md#tuning-options) available to fine-tune volumes
* enable / disable NFS
* accept / reject IP addresses
* timeout
* self-heal
* quorum
* log levels
* and much, much more!



## Native Protocol
* POSIX filesystem via FUSE client
* communicates with all servers in the volume
* replicated volumes perform simultaneous writes to each of the bricks in the replica set
* attempt to communicate with all bricks hosting requested file(s); eventually communicate with first brick to respond


### Performance
> be wary of comparing apples to orchards. Picking a basket of apples to feed 20 people may be the fastest and most efficient way to feed them, but when you have 2000, it's better to just send them in to the orchard to pick their own.  Sure, each individuals performance will be degraded, but the overall task is vastly more performant.



## Network
* Gluster daemon: 24007/tcp
* Bricks: one TCP port per brick, starting at 49152
* built-in NFS: 38465/tcp, 38466/tcp and 38467/tcp



## SSL
Default protocol is plaintext / insecure

SSL is [available](https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_ssl.md)



## Install
Clients and servers should all be running the same version.

Ubuntu seems to release a little slower. Go figure.


### CentOS
Requires [EPEL](https://fedoraproject.org/wiki/EPEL). 
```
$ sudo rpm -ivh http://mirror.us.leaseweb.net/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
$ sudo wget -O /etc/yum.repos.d/gluster.repo http://download.gluster.org/pub/gluster/glusterfs/3.6/3.6.2/CentOS/glusterfs-epel.repo
$ sudo sed -i 's#LATEST/EPEL.repo#3.6/3.6.2/CentOS#' /etc/yum.repos.d/gluster.repo
$ sudo yum repolist
$ sudo yum install glusterfs-server
```


### Debian
```
$ wget -qO - http://download.gluster.org/pub/gluster/glusterfs/3.6/3.6.2/Debian/wheezy/pubkey.gpg | sudo apt-key add -
$ echo deb http://download.gluster.org/pub/gluster/glusterfs/3.6/3.6.2/Debian/wheezy/apt/ wheezy main | sudo tee /etc/apt/sources.list.d/gluster.list
$ sudo apt-get update
$ sudo apt-get install glusterfs-server
```


### Ubuntu
Requires addition of a [PPA](https://launchpad.net/~gluster).
```
$ sudo apt-get install python-software-properties
$ sudo add-apt-repository ppa:gluster/glusterfs-3.6
$ sudo apt-get update
$ sudo apt-get install gusterfs-server
```



## Probe Peers
```
$ sudo gluster peer probe 192.168.99.121
peer probe: success.
$ sudo gluster peer status
Number of Peers: 1

Hostname: 192.168.99.121
Uuid: e00edca3-1592-4cbc-8318-d25dbfa709c3
State: Peer in Cluster (Connected)
```



## Create Distribute Volume
Be sure to create and mount brick LVs first!
```
$ sudo gluster volume create dist1 \
  192.168.99.111:/bricks/dist1/brick \
  192.168.99.121:/bricks/dist1/brick
volume create: dist1: success: please start the volume to access data
$ sudo gluster volume status dist1
Volume dist1 is not started
```
Remember to use sub-directory, rather than mountpoint, for brick!


```
$ sudo gluster volume info dist1
Volume Name: dist1
Type: Distribute
Volume ID: c034f152-0714-4144-9349-1be507deb034
Status: Created
Number of Bricks: 2
Transport-type: tcp
Bricks:
Brick1: 192.168.99.111:/bricks/dist1/brick
Brick2: 192.168.99.121:/bricks/dist1/brick
```


```
$ sudo gluster volume start dist1
volume start: dist1: success
$ sudo gluster volume status dist1
Status of volume: dist1
Gluster process           Port  Online  Pid
------------------------------------------------------------------------------
Brick 192.168.99.111:/bricks/dist1/brick      49152 Y 4616
Brick 192.168.99.121:/bricks/dist1/brick      49152 Y 8009
NFS Server on localhost         N/A N N/A
NFS Server on 192.168.99.121        N/A N N/A

Task Status of Volume dist1
------------------------------------------------------------------------------
There are no active volume tasks
```



## Create Replicated Volume
```
$ sudo gluster volume create repl1 replica 2 \
  192.168.99.111:/bricks/repl1/brick 
  192.168.99.121:/bricks/repl1/brick
volume create: repl1: success: please start the volume to access data
$ sudo gluster volume start repl1
volume start: repl1: success
```
Remember to use sub-directory, rather than mountpoint, for brick!


```
$ sudo gluster volume info repl1

Volume Name: repl1
Type: Replicate
Volume ID: b624c0ec-9522-4f2b-b424-cfe6da784a7c
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 192.168.99.111:/bricks/repl1/brick
Brick2: 192.168.99.121:/bricks/repl1/brick
```


```
$ sudo gluster volume status repl1
Status of volume: repl1
Gluster process           Port  Online  Pid
------------------------------------------------------------------------------
Brick 192.168.99.111:/bricks/repl1/brick      49153 Y 4807
Brick 192.168.99.121:/bricks/repl1/brick      49153 Y 8050
NFS Server on localhost         2049  Y 4819
Self-heal Daemon on localhost       N/A Y 4826
NFS Server on 192.168.99.121        2049  Y 8064
Self-heal Daemon on 192.168.99.121      N/A Y 8069

Task Status of Volume repl1
------------------------------------------------------------------------------
There are no active volume tasks
```



## FUSE client
Point at any brick, client will find all other bricks
```
$ sudo mount -t glusterfs 192.168.99.121:/dist1 /mnt
$ sudo touch /mnt/one.txt
$ ls /bricks/dist1/brick # repeat on both servers
$ sudo umount /mnt
```


```
$ sudo mount -t glusterfs 192.168.99.121:/repl1 /mnt
$ sudo touch /mnt/two.txt
$ ls /bricks/repl1/brick # repeat on both servers
$ sudo umount /mnt
```



## Other fun stuff
* stop a brick, confirm volume still works
* trigger self-heal on replicated volume
* add / remove bricks and rebalance data



## Questions?
