---
title: "How to Setup PostgreSQL Replication"
date: 2024-09-03 T17:26:30-04:00
categories:
  - blog
tags:
  - postgresql
toc: true
---

Setting up PostgreSQL replication ensures that your database remains highly available, providing data redundancy and improving reliability. This guide walks you through the steps to configure PostgreSQL replication between a master and standby node.

## Step 1: Create Replication Slots on the Master

On the master node, you need to create a replication slot for each standby node. This ensures that the WAL (Write-Ahead Logging) files are retained until the standby nodes have consumed them.

```sql
SELECT pg_create_physical_replication_slot('standby1', true);
```

You can view the available slots with the following queries:
```sql
SELECT slot_name, slot_type, active FROM pg_replication_slots;
SELECT * FROM pg_replication_slots;
```

PostgreSQL Configuration
Make sure the following parameters are set in your `postgresql.conf` file:

```
data_directory = '/disk1/postgres/main'
max_replication_slots = 10         # should be >= number of standby nodes
max_wal_senders = 10               # 0 = disabled, keep this more than max_replication_slots
wal_level = logical                # replica or logical suited for recovery needs
max_slot_wal_keep_size             # when standby is OFFLINE for a long time, use this or disable slots to avoid STORAGE getting exhausted on master
wal_keep_segments = 1000   # change according to your storage size
```

User Permissions
Ensure that the PostgreSQL or replication user you use with pg_basebackup has the required permissions set in `pg_hba.conf`.

Check for custom tablespaces.
If you have tablespaces with custom locations, you can find them with the following query:

```sql
SELECT spcname AS tablespace_name,
    pg_tablespace_location(oid) AS location
FROM pg_tablespace;
```

##  Step 2: Prepare the Standby Node
On the standby node, clean up the contents of the standby data directories:

```bash
# Example directories
rm -rf /disk_2/mytables/*
rm -rf /disk1/postgres/main/*
```

## Step 3: Initialize the Standby Node

On the standby node, initialize the data directory with the pg_basebackup command:
Use pg_basebackup to create a base backup from the master. This step will also generate /disk1/postgres/main/postgresql.auto.conf with the configurations needed to connect to the WAL files.
a. Without Custom Tablespaces
```bash
pg_basebackup -h <Master-Hostname> -p <db-port> -D /disk1/postgres/main -U postgres -R -v -P --wal-method=stream --slot <slot_name>
```

b. With Custom Tablespaces
```bash
export PGPASSWORD=mypasswd
pg_basebackup -h <Master-Hostname> -p <db-port> -D /disk1/postgres/main -U postgres \
-T /disk_2/mytables=/disk_2/mytables -R -v -P --wal-method=stream --slot <slot_name>
```

## Step 4: Start the Standby Node

Checking Useful Settings on the Standby Node
```sql
SELECT name, setting, source, sourcefile FROM pg_settings WHERE name='primary_conninfo';
SELECT name, setting, source, sourcefile FROM pg_settings WHERE name='primary_slot_name';
```

Additionally, you can inspect the configuration files directly:

```bash
cat /etc/postgresql/postgresql.auto.conf           
vim /etc/postgresql/postgresql.conf
vim /etc/postgresql/pg_hba.conf
```

## Step 5: Start the Standby Node

```bash
pg_ctl -D /disk1/postgres/main -l /disk1/postgres/main/logfile start
```

## Step X: Terminate/Decommission Standby or Revert CHANGES

If you need to decommission a standby node or if the standby will be offline for an extended period, delete any unused replication slots to prevent storage exhaustion on the master:
```sql
SELECT pg_drop_replication_slot('slot_name');
````  
