# posgre_rep

=========================
db-master - 10.128.0.18
db-slave - 10.128.0.19
=========================

1. INSTALL POSTGRESQL
Master
root@db-master:~# apt install pg-activity postgresql-contrib postgresql

Slave
root@db-slave:~# apt install pg-activity postgresql-contrib postgresql

2. ADD DATA
Master
postgres@db-master:~$ git clone https://github.com/morenoh149/postgresDBSamples.git
postgres@db-master:~$ cd postgresDBSamples/french-towns-communes-francaises/
postgres@db-master:~$ psql -d dz5 -f french-towns-communes-francaises.sql

postgres=# \c dz5;
You are now connected to database "dz5" as user "postgres".

dz5=# select * from regions;
 id | code | capital |            name
----+------+---------+----------------------------
  1 | 01   | 97105   | Guadeloupe
  2 | 02   | 97209   | Martinique
  3 | 03   | 97302   | Guyane
  4 | 04   | 97411   | La Réunion
  5 | 11   | 75056   | Île-de-France
  6 | 21   | 51108   | Champagne-Ardenne
  7 | 22   | 80021   | Picardie
  8 | 23   | 76540   | Haute-Normandie
  9 | 24   | 45234   | Centre
 10 | 25   | 14118   | Basse-Normandie
 11 | 26   | 21231   | Bourgogne
 12 | 31   | 59350   | Nord-Pas-de-Calais
 13 | 41   | 57463   | Lorraine
 14 | 42   | 67482   | Alsace
 15 | 43   | 25056   | Franche-Comté
 16 | 52   | 44109   | Pays de la Loire
 17 | 53   | 35238   | Bretagne
 18 | 54   | 86194   | Poitou-Charentes
 19 | 72   | 33063   | Aquitaine
 20 | 73   | 31555   | Midi-Pyrénées
 21 | 74   | 87085   | Limousin
 22 | 82   | 69123   | Rhône-Alpes
 23 | 83   | 63113   | Auvergne
 24 | 91   | 34172   | Languedoc-Roussillon
 25 | 93   | 13055   | Provence-Alpes-Côte d'Azur
 26 | 94   | 2A004   | Corse
(26 rows)

dz5=# \q

3. CONFIGURE REPLICATION
Master
postgres@db-master:~$ echo "host	replication	all		10.128.0.19/32		trust" >> /etc/postgresql/12/main/pg_hba.conf
postgres@db-master:~$ psql -c "ALTER SYSTEM SET listen_addresses TO '*'";
root@db-master:~# systemctl restart postgresql.service

Slave
postgres@db-slave:~$ mkdir /var/lib/postgresql/12/orig
postgres@db-slave:~$ mv /var/lib/postgresql/12/main/* /var/lib/postgresql/12/orig/
postgres@db-slave:~$ pg_basebackup -h 10.128.0.18 -D /var/lib/postgresql/12/main/ -U replicator -P -v  -R -X stream -C -S pgstandby1

4. CHECK REPLICATION
Master
postgres@db-master:~$ psql -c "\x" -c "SELECT slot_name, slot_type, active_pid FROM pg_replication_slots;"
Expanded display is on.
-[ RECORD 1 ]----------
slot_name  | pgstandby1
slot_type  | physical
active_pid | 6208

Slave
postgres@db-slave:~$ psql -c "\x" -c "SELECT pid, status, slot_name, sender_host, conninfo FROM pg_stat_wal_receiver;"
Expanded display is on.
-[ RECORD 1 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid         | 5246
status      | streaming
slot_name   | pgstandby1
sender_host | 10.128.0.18
conninfo    | user=replicator passfile=/var/lib/postgresql/.pgpass dbname=replication host=10.128.0.18 port=5432 fallback_application_name=12/main sslmode=prefer sslcompression=0 gssencmode=prefer krbsrvname=postgres target_session_attrs=any

