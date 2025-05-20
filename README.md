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
#### Configure as new strandby old primary (server1):
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

### Important Notes :
```
1. Monitor WAL replay log before promoting.
2. Ensure replication users have required permissions.
3. Automate with scripts or use toolslike repmgr , patroni for safer transactions
4. Always back up pg_hba.conf postgresql.conf , and repliction slot.
```




