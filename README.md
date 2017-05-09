<h1>Stacki Docker Pallet - Phase 2</h1>

***This is the Phase 2 stacki-docker pallet for Stacki 4.0.***

***If you have Stacki 3.2, go to the phase2_3.2 tag for the proper documentation and download links.***

Really, you should read the whole README because that nun in fifth grade told you to read all the directions on your phonix worksheet first. I'm not a nun, you're not in fifth grade, but reading the whole thing will help you understand what's happening here.

This is the open source stacki-docker pallet for Stacki based systems. There's nothing to pay for here. It's free, as in beer, and free, as in your nethers when skinny-dipping. Enjoy. (Either Docker, beer, or skinny-dipping. Tell us about the first, not the last.)

StackIQ creates application pallets in phases. Phase 1 means an application will run correctly at its basic usuable level and will be available and functional upon installation. Generally, there is little, if any, in the way of security; it's likely to follow the application's simple case documentation. We did that in Phase 1. If you want that, you can check out the phase1_3.2 tag but it contains only Docker software and no configuration for anything. This was before they created the Community Edition which was, like, last week.

Phase 2 means the application will install on initial installation on a set of machines securely, working for more complex cases and examples, and more flexible. It's probably production ready. It's more, it's better, but it probably needs some further work to make it more flexible for your use case.

Phase 3 will definitely be production ready, will allow for greater security and flexibility; however, that flexibility will require you do more stuff to get it the way you want it. Phase 3 for stacki-docker will be done on stacki 4.0. Phase 2 has been ported to work with Stacki 4.0.

All that being said, this is phase2 of the docker pallet. It runs Docker Community Edition (17.03.1), has docker swarm mode baked in (installs and configure swarm mode automatically thankyouverymuch) and is SSL/TLS endpoint terminated. If you are interested in just Docker and Docker Swarm Mode, this is the pallet and the instructions you should follow.

Please note, we assume either your frontend or your backend nodes have access to the internet. This will work for a local docker registry, (local to your company or to the cluster itself) but I haven't tested that yet so I'm not going to give those instructions. Ask how if this is important to you.

If your frontend has internet access, we'll open the firewall so the backend nodes can get the images they need for the demo. Whether your are able to do this at your site dependent on network policy. Please consult with the people who tell you "no" most often before you do this, or if you prefer forgiveness, go that way. 

If you don't need to run the demo, then you don't have to open the firewall, but you have to have a way to get images to your cluster so you may very well have too. Again, Phase 3 should accommodate more use cases, especially with regard to remote and local registries.

***Warning: Don't do any of this if you are using stacki-docker
with the stacki-kubernetes pallet. Folllow those docs not these.***

This is the tl;dr. It's actually kinda long for a tl;dr but it's shorter than the full set of instructions in which I give you far more details than you want.

<h2>tl;dr</h2>

Do the following on the stacki frontend, unless it says "do this on the backend nodes." 

Download isos:
```
# cd /export
# mkdir isos
# cd isos
# wget https://s3.amazonaws.com/stacki/public/os/centos/7/CentOS-7-x86_64-Everything-1611.iso
# wget https://s3.amazonaws.com/stacki/public/os/centos/7/CentOS-Updates-7.3-7.x.x86_64.disk1.iso
# wget https://s3.amazonaws.com/stacki/public/pallets/4.0/open-source/stacki-docker-17.03.1-7.x_phase2.x86_64.disk1.iso

If you want monitoring:
# wget https://s3.amazonaws.com/stacki/public/pallets/4.0/open-source/stacki-prometheus-1.6.1-7.x.x86_64.disk1.iso
```

Add/enable/disable pallets:
```
# stack add pallet *.iso
# stack list pallet
# stack disable pallet os
# stack enable pallet CentOS CentOS-Updates stacki-docker stacki-prometheus
```

Now add hosts. You have a hosts file right? Prep a host spreadsheet if you don't first.
```
# stack load hostfile file=hosts.csv 
```

You need an attributes spreadsheet:
```
# yum clean all
# yum install stacki-docker-spreadsheets
cd /opt/stack/share/examples/spreadsheets/docker
cat docker-attrs-swarm.csv 
```

I highly suggest editing this in Google Sheets or Excel. Easier to see and make changes.

Change the docker.swarm.master_ip info. The 10.1.255.254 should be the ip of
whatever your docker swarm master will be.

Change the default docker.swarm.overlay_network to something else if you wish, along 
with the docker.swarm.network_name if you don't like my incredibly imaginative "testnet."

Swarm is default enabled. And again, ***don't run the pallet if you're using
kubernetes.***

"docker.swarm.demo" is default enabled. Set it to false if you don't want a whole
bunch of containers right off the bat.

Change the hosts to match your host names. You need one master and two secondary_managers. Everything else can be a node.
You're smart enough to edit the file. 

If you want detailed info on settings, see below.
You're the one following the tl;dr. Suck it up cowboy.

Load the attrs:
```
stack load attrfile file=docker-attrs-swarm.csv 
```

Run the pallet:
```
stack run pallet stacki-docker | bash
```

It worked if the listener is listening:
```
systemctl status stacki-listener
```

Add partitions. This works for me. You may want to do something else.
The "mkfsoptions" is a requirement for the overlay fs no matter what partition you put it on.
```
# stack load storage partition file=docker-partitions.csv 
```

If you want monitoring, use the stacki-prometheus pallet, AFTER
you've done everything above.
```
stack run pallet stacki-prometheus | bash

systemctl is-active prometheus
```

Install your hosts:
```
# stack set host boot backend action=install
# stack set host attr backend attr=nukedisks value=true
```

Now power cycle. When they come up, if you've used stacki prometheus,
you'll have access at: 
http://frontend.IP:9090 for prometheus
http://frontend.IP:3000 for grafana

<h2>Detailed Explanation</h2>

The stacki-docker pallet deploys Docker Community edition 17.03.1, with a flag to start Docker Swarm Mode. Without swarm mode, Docker runs docker daemons with TLS certs on every backend and is availble to accept Docker images. In swarm mode, a designated manager, designated by you along with secondary managers and workers, automatically register themselves to the cluster at boot time, with no intervention from you. 

A demo of NGINX runs by default in Swarm mode because I do a snotload of demos. 

If you don't want this, set that flag to "false." See below in spreadsheet section.

The OS is CentOS 7.3 with updates if you downloaded them. You want to. The pallet provides a 4.10 kernel from elrepo because we are using overlay fs for /var/lib/docker. Overlay is only in preview for CentOS/RHEL 7.2/3 so I've added a modern kernel here. I imagine this helps with cgroups too. (I tossed that out there like I understand those. I don't so don't ask me anything about it.)

<h3>Software</h3>
You can't make an omelet without killing some chickens so get some software.

Download isos:
/export should be the largest partition on your frontend. If it's not, you probably messed something up during the frontend install. Pick the largest partition then. 

```
# cd /export
```

Make a directory to store the isos and then cd to it:
```
# mkdir isos
# cd isos
```

Download some software:
```
# wget https://s3.amazonaws.com/stacki/public/os/centos/7/CentOS-7-x86_64-Everything-1611.iso
# wget https://s3.amazonaws.com/stacki/public/os/centos/7/CentOS-Updates-7.3-7.x.x86_64.disk1.iso
# wget https://s3.amazonaws.com/stacki/public/pallets/4.0/open-source/stacki-docker-17.03.1-7.x_phase2.x86_64.disk1.iso
```

If you want monitoring too, get the prometheus pallet:
```
# wget https://s3.amazonaws.com/stacki/public/pallets/4.0/open-source/stacki-prometheus-1.6.1-7.x.x86_64.disk1.iso
```

If you want details on how to install monitoring got to the [stacki-prometheus](https://github.com/StackIQ/stacki-prometheus) page. Do that after you've done this.

<h3>Frontend Setup</h3>
The following steps are pretty standard for adding an application pallet. A pallet tends to have configuration and software that needs to run on the frontend before installing backend nodes. It's different than a cart in this respect, which only has configuration for backend nodes. 

stacki-docker is a even a little trickier because the order in which you do things matters to get the proper result. If you screw-up a step, though, you can always redo them without having to start at the absolute beginning.


Add/enable/disable pallets:

```
# stack add pallet *.iso
# stack list pallet
```

Looks like this:
```
[root@stackdock isos]# ls
CentOS-7-x86_64-Everything-1611.iso      stacki-docker-17.01.1-7.x_phase2.x86_64.disk1.iso
CentOS-Updates-7.3-7.x.x86_64.disk1.iso  stacki-prometheus-1.6.1-7.x.x86_64.disk1.iso

[root@stackdock isos]# stack add pallet *.iso
Copying "CentOS" (7,x86_64) pallet ...
Copying CentOS-Updates to pallets ...2349528 blocks
Copying stacki-docker to pallets ...322405 blocks
Copying stacki-prometheus to pallets ...112373 blocks

[root@stackdock isos]# stack list pallet

NAME               VERSION                  RELEASE    ARCH   OS     BOXES
os:                7.2                      7.x        x86_64 redhat default
stacki:            4.0_20170414_c4aff2a     7.x        x86_64 redhat default
CentOS:            7                        ---------- x86_64 redhat -------
CentOS-Updates:    7.3                      7.x        x86_64 redhat -------
stacki-docker:     17.03.1                  7.x_phase2 x86_64 redhat -------
stacki-prometheus: 1.6.1                    7.x        x86_64 redhat -------
```

You should see the newly added CentOS, CentOS-Updates, stacki-docker, and stacki-prometheus if you're using it. They will have dashes in the "BOXES" column which means they aren't active. We mostly don't enable things by default, unless I tell you I did, in which case you have to undo the enable. 

You can't have two OS pallets. The "os" pallet is a minimal CentOS 7.2 pallet that is designed for the absolute minimal install. We assume there'll be a need for other things, so we add the CentOS and CentOS-Updates pallets. Which means you no longer need the "os" pallet. Disable it. If you don't disable it, weird things happen. (Usually involving failed backend installs but sometimes involving frogs in party hats.) 

Disable the "os" pallet.

```
[root@stackdock ~]# stack disable pallet os
Cleaning repos: stacki-4.0
Cleaning up everything

[root@stackdock ~]# stack list pallet
NAME               VERSION RELEASE    ARCH   OS     BOXES
os:                7.2     7.x        x86_64 redhat -------
stacki:            4.0_20170414_c4aff2a     7.x        x86_64 redhat default
CentOS:            7                        ---------- x86_64 redhat -------
CentOS-Updates:    7.3                      7.x        x86_64 redhat -------
stacki-docker:     17.03.1                  7.x_phase2 x86_64 redhat -------
stacki-prometheus: 1.6.1                    7.x        x86_64 redhat -------
```

Now we're going to enable the pallets we want to use. Once they are enabled, they are listed in the /etc/yum.repos.d/stacki.repo file and availble via yum to the frontend and all the backends.
```
[root@stackdock ~]#  stack enable pallet CentOS CentOS-Updates stacki-docker stacki-prometheus
Cleaning repos: CentOS-7 CentOS-Updates-7.3 stacki-3.2 stacki-docker-17.03.0 stacki-prometheus-1.0
Cleaning up everything

[root@stackdock ~]# stack list pallet
NAME              VERSION              RELEASE    ARCH   OS     BOXES
os:               7.2                  7.x        x86_64 redhat -------
CentOS            7                    7.x        x86_64 redhat default
stacki            4.0_20170414_c4aff2a 7.x        x86_64 redhat default
CentOS-Updates    4.0_20170414_c4aff2a 7.x        x86_64 redhat default
stacki-docker     17.03.1              7.x_phase2 x86_64 redhat default
stacki-prometheus 1.6.1                7.x        x86_64 redhat default
```

We are not quite done with frontend set-up yet. We need to pre-populate some attributes, key/value pairs, that have to be set before we can run the stacki-docker pallet to set up the configuration. 

<h3>Spreadsheets</h3>

Install the spreadsheets examples for docker. These will be in /opt/stack/share/examples/spreadsheets/docker . They're CSV files. They are easier to see and change if you open them in Google Docs or Excel. Then export back to CSV and load them.

They should be installed. If that directory doesn't exist, yum install them.
```
[root@stackdock ~]# yum install stacki-docker-spreadsheets -y
```

The spreadsheet we are first concerned with contains only the global attributes needed to run the stacki-pallet. There are other ways to do this, but going step by step gives you a feel for what needs to be done in toto.


```
[root@stackdock ~]# cd /opt/stack/share/examples/spreadsheets/docker/

[root@stackdock ]# cat stacki-docker-4.0-global-docker-attrs-swarm.csv

target,docker.registry.external,docker.registry.local,docker.swarm,docker.swarm.demo,docker.swarm.manager,docker.swarm.manager_ip,docker.swarm.node,docker.swarm.overlay_network,docker.swarm.overlay_network_name,docker.swarm.secondary_manager,docker.experimental,docker.swarm.mode
global,True,False,True,True,False,10.1.255.254,True,172.16.10.0/24,testnet,False,True,True
```

Here's a prettier view of it:

| target | docker.registry.external | docker.registry.local | docker.swarm | docker.swarm.demo | docker.swarm.manager | docker.swarm.manager_ip | docker.swarm.node | docker.swarm.overlay_network | docker.swarm.overlay_network_name | docker.swarm.secondary_manager | docker.experimental | docker.swarm.mode | 
|--------|--------------------------|-----------------------|--------------|-------------------|----------------------|-------------------------|-------------------|------------------------------|-----------------------------------|--------------------------------|---------------------|-------------------| 
| global | True                     | False                 | True         | True              | False                | 10.1.255.254            | True              | 172.16.10.0/24               | testnet                           | False                          | True                | True              | 

I'll give a brief explanation here:

"Attributes" or key/value pairs, allow us to customize the install of a bunch of nodes or individual nodes depending how they've been set. This allows us to enable or disable functionality for a set of hosts or individual hosts without having to generate multiple kickstart files. They can be used for configuration settings, or as conditionals in Kickstart files to fire off or not fire off a configuration. 

Attributes allow us flexibility; however, it comes at the cost of complexity. More attributes mean more knobs to turn and buttons to push. It makes for a steeper learning curve. (I would put forth the proposition that even with the added complexity of attributes, Stacki is still simpler to wrap your head around than Cobbler/MaaS/Satellite/Spacewalk/Foreman or, deity forbid, Ironic. (Which, if you've ever tried to use it, turns out to be a play on it's own name.)

So let's go through this a little more completely so you know what you're getting.

Line 1, the "target" line, is the header line. The targets are the "key" part in the attributes pair. 

Line 2, the "global" line, represents the values for each key in the targets line. 

So attributes = key/value pair. Target = key, global = value. One key, one value and this particular file sets these keys needed to configure Docker at a global level. 

You can set key/value pairs at different levels: there's the "global" level pictured above, the "appliance" level which applies only to appliances of a certain kind, (You only have a backend appliance right now so this is not valid at this point.) and a "host" level which applies only to a specific host. This allows us to change the default global setting for individual hosts or groups of hosts. Values are hierarchical and the last one wins. "host" is the lowest level so if an attribute is set at the "host" level, that's the value that's used. Otherwise, the default "global" value is used.

You'll see two types of values for attributes: booleans and strings. I generally use booleans to turn on or off features. If a feature is turned on, the service might require further configuration, in which case there will be an attribute that sets a value to modify the configuration based on site requirements.

With boolean attributes, True, true, yes, Yes, 1 all evaluate to true "true," and any value of False, false, no, No, NO, 0, evaluate to true "false."

We'll go through each of these values and tell you why it exists, what it will do at the current default setting, and why you might want to change it. This will be valuable when you add backend hosts below and whant to change the role they play in the Docker set-up.

```
key: docker experimental
value: True
description: 
Turn on experimental features in Docker CE. If using Prometheus, the experiemental feature is needed to get metrics from docker containers, so default is True. (Those metrics are on port 9323 which is the requested port to Prometheus for obtaining this metric information.) If you don't want metrics, set it to False.

key: docker.registry external
value:True
description: This says whether or not the Docker registry is reachable 
via the Interwebz. At this point I'm assuming this it true. Either all 
of your backend nodes can reach it, or your frontend can. If only your 
frontend can, then set-up the firewall forwarding configuration below.

key: docker.registry local
value:False
description: This is if we are serving a local registry. 
This is not complete yet. It will likely be a fix in 
the next point release, assuming there is one.

key: docker.swarm
value:True
description: This is default True because I've been demoing the 
auto-deploy of Docker Swarm mode. If you are not going to use 
Docker Swarm, then set this to False.

key: docker.swarm.demo
value:True
description: Again, default True because I'm doing demos, like, all 
the f*ing time. This creates three NGINX replicas in the Docker Swarm. 
So if you're not using Swarm, definitely set this to False.

key: docker.swarm.manager
value:False
description: If you are going to use Swarm, then you need a manager. 
By default it's set to False because I don't want all my machines to 
think they're a manager. I'll define one machine as my manager in 
a host attributes file, which means by default, everything else won't be.

key: docker.swarm.manager_ip
value:10.1.255.254
description: I use IPs a lot because it guarantees a bunch of things. 
Once you know which node is going to be the manager, put it's IP here. 
Presumably you aren't doing all of this via discovery. If you are, we should talk. 

key: docker.swarm.node
value:True
description: By default, all nodes are going to be Swarm workers, 
unless we set this to False, which we will for the manager and 
secondary managers. But globally it's true, because this means the 
least amount of typing for us later.

key: docker.swarm.overlay_network
value:172.16.10.0/24
description: Default network for the Docker containers is this one. 
It's arbitrary. Do you have your own? Put it here in network/cidr 
notation. Otherwise go with it. It will only be used if you you're 
using Swarm mode.

key: docker.swarm overlay_network_name
value:testnet
description: Only applies if you're using swarm. Every network 
needs a name or it will feel left out and not a part of the it-crowd. 
You can probably be more imaginative that this. Presumably you're 
testing this before you throw it into production, so "testnet" might 
be fine for the moment.

key: docker.swarm.secondary_manager
value:False
description: Swarm needs secondary managers for the raft algorithm. 
Raft algorithms work best with an odd number of nodes. You could do 
only one manager if you're using a storage backend to save the info, 
but I have not done that. So we need two more secondary_managers for 
a total of 3. The secondary manager role is for a limited number of 
machines so it's False by default. We'll set it to True for a couple 
of nodes in the hosts attributes spreadsheet below. 

key: docker.experimental
value:True
description: Turns on experimental features in the docker engine. Leave it on.

key: docker.swarm.mode
value: True
description: We assumed that's what you wanted. If you only want to have 
docker engine running on backend nodes. Then set this to False.
```

So edit this file appropriately. At minimum you'll have to change the 
docker.swarm.manager_ip and then load the spreadsheet to the database 
and then run the pallet to set-up the frontend to install backends properly.

```
[root@stackdock examples]# stack load attrfile file=stacki-docker-4.0-global-docker-attrs-swarm.csv
/export/stack/spreadsheets/RCS/global-docker-attrs-swarm.csv,v  <--  /export/stack/spreadsheets/global-docker-attrs-swarm.csv
file is unchanged; reverting to previous revision 1.1
done
/export/stack/spreadsheets/RCS/global-docker-attrs-swarm.csv,v  -->  /export/stack/spreadsheets/global-docker-attrs-swarm.csv
revision 1.1 (locked)
done
```
Now we want to get the configuration on the frontend to make the install of the backends work.

```
To see the kickstart script that's generated and run on the fronted:

# stack run pallet stacki-docker 

Now run it for real:

# stack run pallet stacki-docker | bash
```

<h3>Open the firewall</h3>

We are using iptables and not firewalld. Firewall rules are held in the database. 
If you are going to use an external registry, (you have to at the moment) and your backend nodes have no access to the internet, set up the frontend to foward traffic from the backend to the internet. 

You can see the global rules with:

```
# stack list firewall
```

And per host rules with:

```
# stack list host firewall <hostname>
```

If you have outside access, this likely means you have two interfaces on the frontend. One that installs backend machines on a private subnet, and one that has access to the outside world. In the demo example I have been using, that's what I have.

```
[root@stackdock ~]# stack list network
NETWORK  ADDRESS     MASK        GATEWAY      MTU   ZONE   DNS   PXE
private: 10.1.0.0    255.255.0.0 10.1.1.1     1500  local  False True
public:  192.168.0.0 255.255.0.0 192.168.10.1 1500  public False False
```

So I want to allow FORWARDING and MASQUERADING from the outside world to/from the backend nodes. Here's a script that will do that:

```
#!/bin/bash
HOST=`hostname -s`

/opt/stack/bin/stack add host firewall ${HOST} output-network=${1} table=nat rulename=MASQUERADE service="all" protocol="all" action="MASQUERADE" chain="POSTROUTING"

/opt/stack/bin/stack add host firewall ${HOST} network=${1} output-network=private table=filter rulename=FORWARD_PUB service="all" protocol="all" action="ACCEPT" chain="FORWARD"

/opt/stack/bin/stack add host firewall ${HOST} network=private output-network=public table=filter rulename=FORWARD_PRIV service="all" protocol="all" action="ACCEPT" chain="FORWARD"

echo "net.ipv4.ip_forward=1" > /etc/sysctl.conf
sysctl -p /etc/sysctl.conf

stack sync host firewall ${HOST} restart=true
```

Run it like this:

```
chmod 755 fixfw.sh (or whatever you named it)

./fixfw.sh <name of public network from>
```

Since my public network's name is public I would do it like this:
```
./fixfw.sh public
```
The firewall should restart, and you should be able to ping outside services from a backend.


<h3>Backend Setup</h3>

This pallet assumes that you have a host spreadsheet file. If you don't, you should either build one or use discovery to install backend machines with just the basic stacki/os pallets. Then you'll have hosts and you can set up the attributes spreadsheet.

This is what my demo-hosts.csv looks like:

| NAME        | INTERFACE HOSTNAME | DEFAULT | APPLIANCE | RACK | RANK | IP           | MAC               | INTERFACE | NETWORK | CHANNEL | OPTIONS | VLAN | 
|-------------|--------------------|---------|-----------|------|------|--------------|-------------------|-----------|---------|---------|---------|------| 
| backend-0-0 |                    | True    | backend   | 0    | 0    | 10.1.255.254 | c8:1f:66:cb:e7:43 | em1       | private |         |         |      | 
| backend-0-1 |                    | True    | backend   | 0    | 1    | 10.1.255.253 | c8:1f:66:cb:33:74 | em1       | private |         |         |      | 
| backend-0-2 |                    | True    | backend   | 0    | 2    | 10.1.255.252 | c8:1f:66:cb:e5:7d | em1       | private |         |         |      | 
| backend-0-3 |                    | True    | backend   | 0    | 3    | 10.1.255.251 | c8:1f:66:cb:37:75 | em1       | private |         |         |      | 
| backend-0-4 |                    | True    | backend   | 0    | 4    | 10.1.255.250 | c8:1f:66:cd:d3:c0 | em1       | private |         |         |      | 

I'm going to load that:

```
[root@stackdock ~]# stack load hostfile file=hosts.csv
/export/stack/spreadsheets/RCS/hosts.csv,v  <--  /export/stack/spreadsheets/hosts.csv
initial revision: 1.1
done
/export/stack/spreadsheets/RCS/hosts.csv,v  -->  /export/stack/spreadsheets/hosts.csv
revision 1.1 (locked)
done
```
And now I have hosts I can install:
```
[root@stackdock ~]# stack list host
HOST         RACK RANK CPUS APPLIANCE BOX     ENVIRONMENT RUNACTION INSTALLACTION
stackdock:   0    0    1    frontend  default ----------- os        install
backend-0-0: 0    0    1    backend   default ----------- os        install
backend-0-1: 0    1    1    backend   default ----------- os        install
backend-0-2: 0    2    1    backend   default ----------- os        install
backend-0-3: 0    3    1    backend   default ----------- os        install
backend-0-4: 0    4    1    backend   default ----------- os        install
```

When these do install, if there are other network interfaces, these will be discovered and added to the database. Those can then be plumbed and started by either reinstalling or syncing the network. If you are doing bonding/vlaning/other network thingies, get on the Slack channel or Google Groups and ask the ways in which you can make this work for your site.

So now we have hosts, and I want to apply the docker configuration to those hosts. This is my attributes file with host specifications. It's from the stacki-docker-4.0-hosts-docker-attrs-swarm.csv file in the examples directory you are probably still sitting in.


| target      | docker.registry.external | docker.registry.local | docker.swarm | docker.swarm.demo | docker.swarm.manager | docker.swarm.manager_ip | docker.swarm.node | docker.swarm.overlay_network | docker.swarm.overlay_network_name | docker.swarm.secondary_manager | docker.experimental | docker.swarm.mode | 
|-------------|--------------------------|-----------------------|--------------|-------------------|----------------------|-------------------------|-------------------|------------------------------|-----------------------------------|--------------------------------|---------------------|-------------------| 
| backend-0-0 |                          |                       |              |                   | True                 |                         | True              |                              |                                   |                                |                     |                   | 
| backend-0-1 |                          |                       |              |                   |                      |                         | False             |                              |                                   | True                           |                     |                   | 
| backend-0-2 |                          |                       |              |                   |                      |                         | False             |                              |                                   | True                           |                     |                   | 
| backend-0-3 |                          |                       |              |                   |                      |                         |                   |                              |                                   |                                |                     |                   | 
| backend-0-4 |                          |                       |              |                   |                      |                         |                   |                              |                                   |                                |                     |                   | 

These are the same attributes as headers as in the first atttribute spreadsheet we loaded above. In reality, I usually just combine these into one file. The example is stacki-docker-4.0-docker-attrs-swarm.csv It looks like this:

| target      | docker.registry.external | docker.registry.local | docker.swarm | docker.swarm.demo | docker.swarm.manager | docker.swarm.manager_ip | docker.swarm.node | docker.swarm.overlay_network | docker.swarm.overlay_network_name | docker.swarm.secondary_manager | docker.experimental | docker.swarm.mode | 
|-------------|--------------------------|-----------------------|--------------|-------------------|----------------------|-------------------------|-------------------|------------------------------|-----------------------------------|--------------------------------|---------------------|-------------------| 
| global      | True                     | False                 | True         | True              | False                | 10.1.255.254            | True              | 172.16.10.0/24               | testnet                           | False                          | True                | True              | 
| backend-0-0 |                          |                       |              |                   | True                 |                         | True              |                              |                                   |                                |                     |                   | 
| backend-0-1 |                          |                       |              |                   |                      |                         | False             |                              |                                   | True                           |                     |                   | 
| backend-0-2 |                          |                       |              |                   |                      |                         | False             |                              |                                   | True                           |                     |                   | 
| backend-0-3 |                          |                       |              |                   |                      |                         |                   |                              |                                   |                                |                     |                   | 
| backend-0-4 |                          |                       |              |                   |                      |                         |                   |                              |                                   |                                |                     |                   | 


This is the same host attrs file with the "global" line. Or the same global file with the host lines. It's easier in terms of configuring, but harder in terms of explanation. If you use the combined file, just make sure you set the proper values for your attributes in the global line before loading. 

The attributes are the same as the above explanation so I won't explain them again. I'll just explain what we are changing.

In this particular file, we're going to make backend-0-0 our master, and backend-0-1 and backend-0-2 as secondary managers for docker swarm mode. This provides the consensus machines for the raft algorithm. Remember, the default global attribute will be set to the value in the global line for any host that does not change it on the host line in the file. 

To get the config I want: 
docker.swarm.manager is going to be set to True for backend-0-0 and it's default False for everything else.
docker.swarm.node is True by default and I'm going to set it to False for backend-0-[0-2] because they are my managers. 
docker.swarm.secondary_manager is False by default, and I'm setting it to True for backend-0-[1-2]
docker.swarm.manager_ip is 10.1.255.254 and that's at a global level. It matches backend-0-0 from my host csv file I just recently loaded.

It's easiest to put this in an Excel or Google Spreadsheet and edit your values there. Then export back as CSV and load. The file itself will look like this, but commas are hard to keep track of in vi.

```
target,docker.registry.external,docker.registry.local,docker.swarm,docker.swarm.demo,docker.swarm.manager,docker.swarm.manager_ip,docker.swarm.node,docker.swarm.overlay_network,docker.swarm.overlay_network_name,docker.swarm.secondary_manager,docker.experimental,docker.swarm.mode
global,True,False,True,True,False,10.1.255.254,True,172.16.10.0/24,testnet,False,True,True
backend-0-0,,,,,True,,True,,,,
backend-0-1,,,,,,,False,,,True,
backend-0-2,,,,,,,False,,,True,
backend-0-3,,,,,,,,,,,
backend-0-4,,,,,,,,,,,
```

So let's load it:

```
[root@stackdock examples]# stack load attrfile file=stacki-docker-4.0-host-docker-attrs-swarm.csv
/export/stack/spreadsheets/RCS/host-docker-attrs-swarm.csv,v  <--  /export/stack/spreadsheets/host-docker-attrs-swarm.csv
initial revision: 1.1
done
/export/stack/spreadsheets/RCS/host-docker-attrs-swarm.csv,v  -->  /export/stack/spreadsheets/host-docker-attrs-swarm.csv
revision 1.1 (locked)
done
```

We also want to add a partitioning scheme. This is the one I've been using. Even if you don't use this, whatever you put /var/lib/docker in, you'll want the "--mkfsoptions -n fsytpe=1" to properly support the overlay fs Docker is using.

My partitions.csv looks like this:
```
Name,Device,Mountpoint,Size,Type,Options
backend,sda,biosboot,1,biosboot,
,sda,/boot,1024,xfs,
,sda,/,16000,xfs,
,sda,/var,26000,xfs,
,sda,swap,8192,swap,
,sda,/var/lib/docker,0,xfs,--grow --mkfsoptions="-n ftype=1"
```

| Name    | Device | Mountpoint      | Size  | Type     | Options                           | 
|---------|--------|-----------------|-------|----------|-----------------------------------| 
| backend | sda    | biosboot        | 1     | biosboot |                                   | 
|         | sda    | /boot           | 1024  | xfs      |                                   | 
|         | sda    | /               | 16000 | xfs      |                                   | 
|         | sda    | /var            | 26000 | xfs      |                                   | 
|         | sda    | swap            | 8192  | swap     |                                   | 
|         | sda    | /var/lib/docker | 0     | xfs      | --grow --mkfsoptions="-n ftype=1" | 

So let's load that:

```
[root@stackdock examples]# stack load storage partition file=docker-partitions.csv
/export/stack/spreadsheets/RCS/docker-partitions.csv,v  <--  /export/stack/spreadsheets/docker-partitions.csv
initial revision: 1.1
done
/export/stack/spreadsheets/RCS/docker-partitions.csv,v  -->  /export/stack/spreadsheets/docker-partitions.csv
revision 1.1 (locked)
done
```

All that looks good. Hopefully we didn't screw anything up. NOW we want to run the stacki-docker pallet.

```
[root@stackdock examples]# stack run pallet stacki-docker | bash
```
Should output something like this:
```
Package stacki-docker-spreadsheets-4.0-7.x_phase2.noarch already installed and latest version
Nothing to do
RCS file: /opt/stack/sbin/RCS/stacki-listener,v
done
RCS file: /usr/lib/systemd/system/RCS/stacki-listener.service,v
done
Created symlink from /etc/systemd/system/multi-user.target.wants/stacki-listener.service to /usr/lib/systemd/system/stacki-listener.service.
```

And your stacki-listener should be listening:

```
[root@stackdock examples]# systemctl is-active stacki-listener
active
```
(The stacki-listener is the magic to automatically join managers and nodes to the Docker Swarm when it's run in swarm mode. It just listens for a worker token and a manager token, which are generated by the swarm manager on first boot. The other nodes listen and wait until they get a "worker:Token" pair and then they join the cluster. This means you don't have to run any commands to get a node to join the swarm. You'll appreciate this if you're as lazy as I am. The stacki-listener is just python xml-rpc. I would like to rewrite it in Go but you could also use the Salt event bus or redis if you want to go that route.)

<h3>Monitoring</h3>

If you want monitoring, the stacki-prometheus pallet should be run at this point, after running the stacki-docker pallet and adding machines. The directions are [here](https://github.com/StackIQ/stacki-prometheus/blob/master/README.md). 

When run with the docker pallet, there will be some default docker dashboards along with machine metrics exporting to prometheus and viewable in Grafana. You're welcome. 

<h3>Install backends</h3>

Damn, that took forever. 

I sum-up:
    
    - isos are installed
    - hosts are in the database
    - attrfile is correct and loaded
    - partitions are loaded
    - stacki-docker has been run



Let's install:
```
[root@stackdock]# stack set host boot backend action=install
[root@stackdock]# stack set host attr backend attr=nukedisks value=true
```
Now power cycle whatever way you can. 

The first command sets the nodes to install the next boot, assuming your nodes are set to PXE first. (You're running a cluster - why aren't they PXE first? [ Oh! Because you're working for a paranoid bank.])

Setting the "nukedisks" command will reformat and repartition our disks which is what we want. This is no brownfield - you're getting these under your complete control. 

When these nodes come up, if you have enabled docker.swarm.mode and docker.swarm, you should have a bunch of nodes available in docker swarm mode. If you have not enabled these two attributes, docker will be installed on these nodes and running, ready to pull from the default registry, which I think is hub.docker.io or whatever. 

I have some ideas for how to handle registries. My assumpution is people want external access (hub.docker.io), internal access to a site registry, or a registry local to the cluster, i.e. docker run registry on a given node. If there is some other method you want to see for registries let me know on Slack or on Google Groups, and I'll add it to the next update of the stacki-docker pallet.

Thanks, good luck, come to us with questions. 


