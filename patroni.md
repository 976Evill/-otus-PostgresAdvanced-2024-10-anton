3 ноды etcd
[student@etcd1 ~]$ etcdctl  member list
158c70ccda4f126, started, etcd3, http://192.168.21.16:2380, http://192.168.21.16:2379, false
d37d5e72a203cbc1, started, etcd2, http://192.168.21.79:2380, http://192.168.21.79:2379, false
f00ea04c88a413f4, started, etcd1, http://192.168.20.59:2380, http://192.168.20.59:2379, false
[student@etcd1 ~]$ vi /etc/etcd/etcd.conf
ETCD_NAME="etcd1"
ETCD_LISTEN_CLIENT_URLS="http://192.168.20.59:2379,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.20.59:2379"
ETCD_LISTEN_PEER_URLS="http://192.168.20.59:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.20.59:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-postgres-cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.20.59:2380,etcd2=http://192.168.21.79:2380,etcd3=http://192.168.21.16:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
ETCD_LOG_OUTPUT="/var/lib/etcd/etcd.log"

3 хоста patroni
192.168.26.223  node1
192.168.26.153  node2
192.168.26.200  node3
[student@node1 ~]$ patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni-cluster (7461942838885935538) -------+----+-----------+
| Member    | Host           | Role         | State     | TL | Lag in MB |
+-----------+----------------+--------------+-----------+----+-----------+
| patroni-1 | 192.168.26.223 | Leader       | running   | 34 |           |
| patroni-2 | 192.168.26.153 | Replica      | streaming | 34 |         0 |
| patroni-3 | 192.168.26.200 | Sync Standby | streaming | 34 |         0 |
+-----------+----------------+--------------+-----------+----+-----------+
student@node1 ~]$ vi /etc/patroni/config.yml

scope: patroni-cluster

name: patroni-1

namespace: /service

log:
  traceback_level: INFO
  level: INFO
  dir: /etc/patroni/logs/
  file_num: 5

restapi:

  listen: 192.168.26.223:8008

  connect_address: 192.168.26.223:8008

etcd:

  hosts: etcd1:2379,etcd2:2379,etcd3:2379

bootstrap:

  method: initdb

  dcs:

    ttl: 30

    loop_wait: 10

    retry_timeout: 10

    maximum_lag_on_failover: 1048576

  initdb:

    - data-checksums

  postgresql:
    use_pg_rewind: true

    use_slots: true

    parameters:

      superuser_reserved_connections: 5

      max_connections: 1500

      shared_buffers: 512MB

      effective_cache_size: 4GB

      work_mem: 128MB

      maintenance_work_mem: 256MB

      random_page_cost: 4

      effective_io_concurrency: 2

      huge_pages: try

      checkpoint_timeout: 15min

      wal_level: replica

      wal_compression: on

      max_wal_senders: 10

      synchronous_commit: on

      max_replication_slots: 10

haproxy:
ip = 192.168.26.114 
Last login: Thu Feb 13 16:20:20 2025 from 10.4.14.47
student@haproxy:~$ vi /etc/haproxy/haproxy.cfg

global
    maxconn 100

defaults
    log global
    mode tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen boorvar_cluster
    bind *:5432
    option httpchk
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 192.168.26.223:5432 maxconn 100 check port 8008
    server node2 192.168.26.153:5432 maxconn 100 check port 8008
    server node3 192.168.26.200:5432 maxconn 100 check port 8008

Проверяем:
1) student@test:~$ psql -h 192.168.26.114 -U postgres                              psql (15.10, сервер 16.6)
ПРЕДУПРЕЖДЕНИЕ: psql имеет базовую версию 15, а сервер - 16.
                Часть функций psql может не работать.
Введите "help", чтобы получить справку.


postgres=# create table test1 ( s text);
CREATE TABLE
postgres=#
2) выключим мастер, проверим файловер и переезд мастера на другую ноду:
Last login: Tue Feb 11 09:32:48 2025 from 10.4.14.47
[student@node2 ~]$ patronictl -c /etc/patroni/config.yml list
+ Cluster: patroni-cluster (7461942838885935538) -------+----+-----------+
| Member    | Host           | Role         | State     | TL | Lag in MB |
+-----------+----------------+--------------+-----------+----+-----------+
| patroni-2 | 192.168.26.153 | Sync Standby | streaming | 35 |         0 |
| patroni-3 | 192.168.26.200 | Leader       | running   | 35 |           |
+-----------+----------------+--------------+-----------+----+-----------+
Patroni отработал успешно
3) Проверяем подключение со стороны haproxy
postgres=# \q
student@test:~$ psql -h 192.168.26.114 -U postgres
psql (15.10, сервер 16.6)
ПРЕДУПРЕЖДЕНИЕ: psql имеет базовую версию 15, а сервер - 16.
                Часть функций psql может не работать.
Введите "help", чтобы получить справку.

postgres=# \dt
          Список отношений
 Схема  |  Имя  |   Тип   | Владелец
--------+-------+---------+----------
 public | test  | таблица | postgres
 public | test1 | таблица | postgres
(2 строки)

postgres=#
haproxy также отработал корректно



 










