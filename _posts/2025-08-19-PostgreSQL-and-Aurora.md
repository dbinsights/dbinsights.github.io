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
- Identity
- Explain 
- Interpret and analysis
    - estimated vs acutal rows
    - most expensive cost 
- Tuning and verify

![Query processing](/images/psql/QPM-enhanced-Query-Processing-Flow.png)

Stats
- Table Stats
    - Partition stats
- Column Stats 
    - Data distribution
    - Column corelation
- System Stats
    - Seq_page_cost/Random_page_cost = 1/4
    - Scattered Read/Sequential Read = ((10ms+8*2ms)/8) / (10ms+2ms) = 0.27   <a href="https://www.apress.com/la/book/9781590596364">Jonathan Lewis</a>


Bind value type
- <a href="https://jonathanlewis.wordpress.com/2007/01/05/bind-variables/#comment-99416">Character columns 32, 128, 2000, or 4000 bytes </a>

- <a href="https://www.brentozar.com/archive/2025/01/i-feel-sorry-for-untrained-developers-using-entity-framework/">EF string properties to nvarchar(max)</a>

Bind variables
- <a href="https://github.com/dotnet/efcore/issues/13617#issuecomment-716052091">IN() list queries are not parameterized </a>
- <a href="https://github.com/prisma/prisma/issues/21648"> Prevent Prisma from reaching bind parameters limit</a>


<a href="https://www.linkedin.com/posts/jipeng-liu_azure-sql-managed-instance-1-excessive-activity-7226446481514278912-u8Bc?utm_source=share&utm_medium=member_desktop&rcm=ACoAACI4CSsBWsGS38S2UJ7lJ7pNPKzCYXJ24OA">Long compilation time issue</a>

### DB time model/Wait event
 
### Monitor
- APM
- Metrics
- Plan stability


![alt text](/images/psql/psql-dr-promote-oldprimary-ha.png)


[BeiYuu]:    http://beiyuu.com  "BeiYuu"
