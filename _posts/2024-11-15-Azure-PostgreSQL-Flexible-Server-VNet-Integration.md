---
layout: post
title: Azure PostgreSQL Flexible Server - VNet Integration 
description: Azure PostgreSQL Flexible Server VNet Integration
category: blog
---


## VNet Intergration
> VNet injection is the VNet integration pattern for services whose architecture is based on dedicated resources that can be deployed (aka “injected”) into the instance owner’s VNet


### Subnet
- To deploy above dedicated resources, a delegtion subnet needed.
- To simplify system design and maintenance, psql instances with similar role could be grouped together. Make sure plan your subnet size, as it can not be changed after deployment. From MS document, one psql with HA will use 4 IPs.

### Peering 
If you need config a async replica for DR, VNet peering needed for these VNet pair. 

## Intra VNet Access
Within VNet, different subnets could be designed for different roles like application, management and etc. NSGs configured for aceess between these Subnets.

Here simulated psql client access from insider subnet (10.1.2.4)
    psqladmin@psqlvm-intra:~$ ifconfig -a
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.1.2.4  netmask 255.255.255.0  broadcast 10.1.2.255

Address Resoved via default linked private DNS zone
psql-core1.postgres.database.azure.com ->  a1b9c61e56fd.psql-core1.private.postgres.database.azure.com -> 10.1.1.4

    psqladmin@psqlvm-intra:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.4

    psqladmin@psqlvm-intra:~$ nslookup psql-core1-dr.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-dr.postgres.database.azure.com       canonical name = e2f2dd939144.psql-core1.private.postgres.database.azure.com.
    Name:   e2f2dd939144.psql-core1.private.postgres.database.azure.com
    Address: 10.2.1.4

    psqladmin@psqlvm-intra:~$ nslookup psql-core2.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core2.postgres.database.azure.com  canonical name = b6c43f34036d.psql-core2.private.postgres.database.azure.com.
    Name:   b6c43f34036d.psql-core2.private.postgres.database.azure.com
    Address: 10.1.1.7


## Extra VNet Access
Here simulated psql client access from outside subnet (10.3.1.4)

    psqladmin@psqlvm-inter:~$ ifconfig -a
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.3.1.4  netmask 255.255.255.0  broadcast 10.3.1.255

Default not work as network is not peering.
After VNet peering, can access using psql server IP.

    psqladmin@psqlvm-inter:~$ psql -h 10.1.1.4 -d postgres
    Password for user psqladmin:
    psql (12.20 (Ubuntu 12.20-0ubuntu0.20.04.1), server 16.4)
    WARNING: psql major version 12, server major version 16.
            Some psql features might not work.
    SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
    Type "help" for help.

    postgres=> \l

DNS name can not be resolved.
    psqladmin@psqlvm-inter:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    ** server can't find psql-core1.postgres.database.azure.com: NXDOMAIN

After manually linked private DNS zone with Server, it can be resolved.
![Private DNS Zone](/images/psql/private-dns-zone-vnet-link.png)


    psqladmin@psqlvm-inter:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.4

### Private DNS Zone 
From above inside and outside access testing, private DNS zone are vital for application connections, especially in HA scenario. 

![alt text](/images/psql/private-dns-zone-recordsets.png)


## HA/DR

### HA
![alt text](/images/psql/psql-ha1.png)

    psqladmin@psqlvm-inter:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.4

    psqladmin@psqlvm-inter:~$ nslookup psql-core1-dr.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-dr.postgres.database.azure.com       canonical name = e2f2dd939144.psql-core1.private.postgres.database.azure.com.
    Name:   e2f2dd939144.psql-core1.private.postgres.database.azure.com
    Address: 10.2.1.4


Triggered a planned failover between different AZ.
![alt text](/images/psql/psql-ha2.png)

    psqladmin@psqlvm-inter:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.4

    # HA failover

    psqladmin@psqlvm-inter:~$ nslookup psql-core1.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.5

After failover, a1b9c61e56fd's according IP changed to 10.1.1.5
![private-dns-zone-recordsets-ha](/images/psql/private-dns-zone-recordsets-ha.png)


### DR
![alt text](/images/psql/psql-dr1.png)

Virtual endpoints
These logical domains simplify interactions with the underlying servers, ensuring efficient routing and operation distribution.
- read write -> HA primary server
- read only  -> Read replica server


    psqladmin@psqlvm-inter:~$ nslookup psql-core1-fg.writer.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-fg.writer.postgres.database.azure.com        canonical name = psql-core1.postgres.database.azure.com.
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.4

    psqladmin@psqlvm-inter:~$ nslookup psql-core1-fg.reader.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-fg.reader.postgres.database.azure.com        canonical name = psql-core1-dr.postgres.database.azure.com.
    psql-core1-dr.postgres.database.azure.com       canonical name = e2f2dd939144.psql-core1.private.postgres.database.azure.com.
    Name:   e2f2dd939144.psql-core1.private.postgres.database.azure.com
    Address: 10.2.1.4

Promote read replica
- Action:
 - Promote to primary server
 - Promote to independent server and split replication

- Data sync:
Similar like HA failover, there are
 - Planned, RPO = 0
 - Forced, data loss.

![alt text](/images/psql/psql-dr-promote.png)


    psqladmin@psqlvm-inter:~$ nslookup psql-core1-fg.writer.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-fg.writer.postgres.database.azure.com        canonical name = psql-core1.postgres.database.azure.com.
    psql-core1.postgres.database.azure.com  canonical name = a1b9c61e56fd.psql-core1.private.postgres.database.azure.com.
    Name:   a1b9c61e56fd.psql-core1.private.postgres.database.azure.com
    Address: 10.1.1.5
    
    # Promote read replica
    
    psqladmin@psqlvm-inter:~$ nslookup psql-core1-fg.writer.postgres.database.azure.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    psql-core1-fg.writer.postgres.database.azure.com        canonical name = psql-core1-dr.postgres.database.azure.com.
    psql-core1-dr.postgres.database.azure.com       canonical name = e2f2dd939144.psql-core1.private.postgres.database.azure.com.
    Name:   e2f2dd939144.psql-core1.private.postgres.database.azure.com
    Address: 10.2.1.4

Be aware of HA on pre-primary will be removed, as high availability is not supported for replica server. 
![alt text](/images/psql/psql-dr-promote-oldprimary-ha.png)


[BeiYuu]:    http://beiyuu.com  "BeiYuu"
