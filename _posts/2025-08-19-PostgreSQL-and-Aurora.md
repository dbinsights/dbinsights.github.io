---
layout: post
title: PostgreSQL and Aurora 
description: 
category: blog
---
## Deployment
### HA/DR
![Azure PostgreSQL HA](/images/psql/Azure-PSQL-concepts-zone-redundant-high-availability-architecture.png)

![AWS RDS PostgreSQL HA](/images/psql/RDS-PostgreSQL-Multi-AZ-architecture-with-one-standby-and-two-standbys.png)

### Infra Deployment
Terraform vs Pulumi 
- Desired State Configurate
- Code-State File-Infra
- Apply


*Tech stack*

- Mature and widely adopted and extensive provider support.

    - CMK support for Geo-backup enabled Azure PostgreSQL flexible server
    https://github.com/pulumi/pulumi-azure-native/issues/4014

### DB Deployment
Prisma vs EF 
- Code First
- DSC
- Code-Migrations-DB
- Apply

Issues

- table/column rename

    - EF: Need specific seting, otherwise recreate table may happen
    https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.migrations.migrationbuilder.renamecolumn?view=efcore-9.0

    - https://github.com/prisma/prisma/issues/7710

- Column order
    - https://dba.stackexchange.com/questions/215037/ssdt-drop-and-recreate-tables-when-nothing-has-changed

    - https://github.com/prisma/prisma/issues/3623


*Code First vs DB First*

GitLab converts the existing Ruby schema from schema.rb to structure.sql
https://gitlab.com/gitlab-org/gitlab/-/merge_requests/22808




## Optimization

### Fine-tuning
- Table stats
    - Partition stats
- Column stats 
    - Data distribution
    - Column corelation
- System stats
    - Seq/Random
    - Sequtial read/ Scatter read 

### DB time model/ wait event
- 
### Monitor
- APM
- Metrics
- 


![alt text](/images/psql/psql-dr-promote-oldprimary-ha.png)


[BeiYuu]:    http://beiyuu.com  "BeiYuu"
