# Switchover-And-Switchback
```

```

## Switchover

- When to perform :-
    1. Maintainance on the primary server.
           - Patch / upgrade
           - Reboot Matchine
           - Change Hardware or migrade to differen disk/VM.
    3. Load balancing
    4. Testing HA setup

## Steps (Manual using Streaming Replication): 
```
Assume:
  Primary = server1
  standby = server2

--- On Primary (server1): Ensure all wal files are shipped and applied on standby.

SELECT pg_switch_wal();

---On standby Promote (server2):

pg_ctl promote -D /var/lib/pgsql/16/data or SELECT pg_promote();

--- On Old Primary (server1) Stop the services:-

pg_ctl stop -D /var/lib/pgsql/16/data

```
#### Configure as new standby old primary (server1):
```
1. Clear or rename old data directory.
2. Use pg_basebackup from new primary (server2).

pg_basebackup -h server2 -D /var/lib/pgsql/16/data -U replication_user -P -R

-- Start services of new standby(server1)--
pg_ctl start -D /var/lib/pgsql/16/data
```
---
---

## Switchback :- Returning the original primary back to its primary role after its recovered or maintained.
```
--- Steps (Same as Switchover but in reverse):
Now:
server2 is primary
server1 is standby

```
### Verify all parameters on both Standby and Primary servers before the pg_basebackup:

# Primary Server Parameters

| Parameter                 | Description |
|---------------------------|-------------|
| **wal_level**             | Sets the amount of information written to WAL. Must be `replica` or `logical` for replication. |
| **max_wal_senders**       | Number of replication (WAL sender) processes to allow. |
| **wal_keep_size**         | Minimum size of WAL files to retain to support slower standbys. |
| **max_replication_slots** | Max number of replication slots to retain WAL for standbys. |
| **archive_mode**          | Enables archiving of WAL files to a configured archive. |
| **archive_command**       | Command to run to archive a WAL segment file. |
| **wal_sender_timeout**    | Time to wait before terminating unresponsive standby connections. |
| **synchronous_standby_names** | Specifies standby servers considered as synchronous for replication. |
| **synchronous_commit**    | Controls whether commit waits for acknowledgment from standby. |

---

# Standby Server Parameters

| Parameter                 | Description |
|---------------------------|-------------|
| **primary_conninfo**      | Connection string to connect to the primary server. Includes host, port, user, etc. |
| **primary_slot_name**     | The name of the replication slot to use on the primary. |
| **standby.signal**        | File that activates standby mode in PostgreSQL 12+. |
| **restore_command**       | Command to fetch archived WAL files during recovery. |
| **wal_receiver_timeout**  | Timeout if standby doesn’t receive WAL data from primary in time. |
| **wal_receiver_status_interval** | Interval at which standby sends feedback to primary. |
| **recovery_min_apply_delay** | Introduces an artificial delay before applying WAL files. |
| **hot_standby_feedback**  | Sends feedback to prevent tuple cleanup on primary that standby still needs. |
| **recovery_target_***     | Used for Point-In-Time Recovery (e.g., `recovery_target_time`, `recovery_target_lsn`). |

### Important Notes :
```
1. Monitor WAL replay log before promoting.
2. Ensure replication users have required permissions.
3. Automate with scripts or use toolslike repmgr , patroni for safer transactions
4. Always back up pg_hba.conf postgresql.conf , and repliction slot.
```




