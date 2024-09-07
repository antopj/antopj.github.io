---
title: "Monitoring PostgreSQL Replication"
date: 2024-09-06 T14:45:00-04:00
categories:
  - blog
tags:
  - postgresql
toc: true
toc_label: "On this Page"
---

PostgreSQL replication is a critical component of database high availability and disaster recovery. Ensuring that replication is running smoothly and identifying any issues early is essential to maintaining database reliability.

Replication lag is one of the most crucial metrics to monitor. Lag refers to the delay between the primary and the standby server. A significant lag could indicate network issues, resource constraints, or slow I/O or busy standby/master Node, or locks in the standby node.

In this post, I will walk you through different methods and tools to monitor PostgreSQL replication effectively.

## Prerequisites

Before you begin monitoring PostgreSQL replication, ensure that:
- You have a PostgreSQL master-slave (primary-secondary) replication setup.
- You have appropriate permissions to access both the primary and replica servers.
- Monitoring tools like `psql`, `pg_stat_replication`, and `pg_stat_wal_receiver` are enabled.

## MASTER

- Slots Info
```sql
SELECT * FROM pg_replication_slots;
```

- Replication Slots & Lag in Bytes
```sql
SELECT slot_name,
  pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS replicationSlotLag,
  pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), confirmed_flush_lsn)) AS confirmedLag,
  active
FROM pg_replication_slots;
```

- Check if all walreceiver processes are active or not
```sql
SELECT * FROM pg_stat_replication;
```

- Important fields in pg_stat_replication to see connected clients
```sql
SELECT pid, application_name, client_addr, client_hostname, state, sync_state, replay_lag FROM pg_stat_replication;
```

- Check Lag stats
```sql
SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn, write_lag, flush_lag, replay_lag
FROM pg_stat_replication;
```

- Traffic Source (Based on Application Name)
```sql
SELECT application_name, count(*) FROM pg_stat_activity GROUP BY application_name;
```

- Check WAL File Numbers (Ensure They Change Over Time)
```sql
SELECT pg_walfile_name(restart_lsn) FROM pg_replication_slots;
```

- Check WAL Receive & Apply Stats (Check Current Time)
```sql
SELECT application_name, client_addr, client_port, state, sent_lsn, write_lsn, flush_lsn, replay_lsn, write_lag, flush_lag, replay_lag, sync_state,
(extract(epoch from (now() - reply_time)))::int FROM pg_stat_replication;
```


## STANDBY / SLAVE

Important: Replication Lag can be divided into two parts:
1. `WAL Receive Lag`: The time it takes for the standby to receive the WAL file from the primary.
2. `WAL Apply Lag`: The time it takes for the standby to apply the received WAL file to the replica.
      - These are the issues are at the Standby side. `du -sch * /<data-dir>/main` will show increasing disk usage over time. Once the disk usage is saturated, the standby will stop receiving new WAL files from the primary, causing a backlog in primary/Master node and will accumulate WAL files in the WAL log of Master node.

- Check Hosts Resource Usage
```bash
top -cd1
free -mh
grep "out of" /var/log/syslog
du -sch * /<data-dir>/main     # If Standby is not ab
```

- WAL Receive & Apply Stats (Check Current Time)
```sql
SELECT pg_is_in_recovery(), pg_is_wal_replay_paused(), pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp(), now()-pg_last_xact_replay_timestamp() AS ReplicaLag;
``` 

- Check Disk Utilization (Performance) Using iostat
```bash
iostat -dx /dev/vda
iostat -x -d -m 1 /dev/vda /dev/vdb /dev/vdc /dev/vdd
iostat -x -d -m 1  /dev/vda /dev/vdb /dev/vdc /dev/vdd | awk '{printf "%-12s %-10s %-10s %-10s %-10s %-10s\n", $1, $2, $3, $8, $9, $21}'
```
- Check Lag in Bytes
```sql
SELECT
  pg_is_in_recovery() AS is_standby,
  pg_last_wal_receive_lsn() AS receive,
  pg_last_wal_replay_lsn() AS replay,
  pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() AS synced,
  (
   EXTRACT(EPOCH FROM now()) -
   EXTRACT(EPOCH FROM pg_last_xact_replay_timestamp())
  )::int AS lag;
```

- Check Long-Running Queries
```sql
SELECT
  pid, datname, client_addr, client_port,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '15 minutes'
  AND state = 'active'
ORDER BY duration DESC;
```

- Cancel/Terminate a Specific Query Using PID
```sql
SELECT pg_terminate_backend(<PID>);
```

- Kill All Long-Running Queries Older Than 15 Minutes
```sql
SELECT pid, pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '15 minutes'
```

- Currently Running Queries
```sql
SELECT query, state, usename, datname, now() - backend_start AS execution_time
FROM pg_stat_activity
ORDER BY execution_time DESC;
```

- WAL Receive Stats
```sql
SELECT pid, slot_name, status, receive_start_lsn, written_lsn, flushed_lsn, last_msg_receipt_time FROM pg_stat_wal_receiver;
```
