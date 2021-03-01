# posgre_rep

- db-master 10.128.0.18
- db-slave 10.128.0.19

## 1. INSTALL POSTGRESQL

Master

`root@db-master:~# apt install pg-activity postgresql-contrib postgresql`

Slave

`root@db-slave:~# apt install pg-activity postgresql-contrib postgresql`



## 2. ADD DATA

Master

`postgres@db-master:~$ git clone https://github.com/morenoh149/postgresDBSamples.git`

`postgres@db-master:~$ cd postgresDBSamples/french-towns-communes-francaises/`

`postgres@db-master:~$ psql -d dz5 -f french-towns-communes-francaises.sql`



## 3. CONFIGURE REPLICATION

Master

`postgres@db-master:~$ echo "host        replication     all             10.128.0.19/32          trust" >> /etc/postgresql/12/main/pg_hba.conf`

`postgres@db-master:~$ psql -c "ALTER SYSTEM SET listen_addresses TO '*'";`

`root@db-master:~# systemctl restart postgresql.service`

Slave

`postgres@db-slave:~$ mkdir /var/lib/postgresql/12/orig`

`postgres@db-slave:~$ mv /var/lib/postgresql/12/main/* /var/lib/postgresql/12/orig/`

`postgres@db-slave:~$ pg_basebackup -h 10.128.0.18 -D /var/lib/postgresql/12/main/ -U replicator -P -v  -R -X stream -C -S pgstandby1`



## 4. CHECK REPLICATION

Master

`postgres@db-master:~$ psql -c "\x" -c "SELECT slot_name, slot_type, active_pid FROM pg_replication_slots;"`

```
Expanded display is on.
-[ RECORD 1 ]----------
slot_name  | pgstandby1
slot_type  | physical
active_pid | 6208
```

Slave

`postgres@db-slave:~$ psql -c "\x" -c "SELECT pid, status, slot_name, sender_host, conninfo FROM pg_stat_wal_receiver;"`

```
Expanded display is on.
-[ RECORD 1 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid         | 5246
status      | streaming
slot_name   | pgstandby1
sender_host | 10.128.0.18
conninfo    | user=replicator passfile=/var/lib/postgresql/.pgpass dbname=replication host=10.128.0.18 port=5432 fallback_application_name=12/main sslmode=prefer sslcompression=0 gssencmode=prefer krbsrvname=postgres target_session_attrs=any
```
