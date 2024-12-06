
ДЗ - Делам бэкап Постгреса используя WAL-G или pg_probackup и восстанавливаемся на другом кластере

1) Установливаем pg_probackup в обеих системах: backuphost (сервер резервного копирования) и node3 (сервер баз данных).
yum install pg-probackup-ent-16
2)Настраиваем  беспарольное подключение через SSH с backuphost к node3 для нашего пользователя ОС:
3)сами действия по созданию резервоной копии и восстановлении
на хосте backuphost:
создаём папку для екапов
mkdir /backups
chown -R student /backups
инициализируем каталог резервных копий:
[student@backuphost bin]$ ./pg_probackup init -B /backups
INFO: Backup catalog '/backups' successfully initialized
[student@backuphost bin]$ 
Добавляем экземпляр с именем bihadb в наш каталог резервных копий:
[student@backuphost backups]$ pg_probackup add-instance -B /backups -D /var/lib/pgpro/ent-16/data/ --instance=bihadb --remote-host=node3 --remote-user=postgres
INFO: Instance 'bihadb' successfully initialized
Делаем полную резервную копию:
[student@backuphost backups]$ pg_probackup backup \
    -B /backups \
    -b FULL \
    --instance=bihadb \
    --stream \
    --remote-host=node3 \
    --remote-user=postgres \
   -U postgres \
    -d biha_db
INFO: Backup start, pg_probackup version: 2.8.2, instance: bihadb, backup ID: SO25GO, backup mode: FULL, wal mode: STREAM, remote: true, compress-algorithm: none, compress-level: 1
INFO: This PostgreSQL instance was initialized with data block checksums. Data block corruption will be detected
WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_probackup under superuser.
INFO: Backup SO25GO is going to be taken from standby
INFO: Database backup start
INFO: wait for pg_backup_start()
Password for user postgres: 
INFO: PGDATA size: 39MB
INFO: Current Start LSN: 0/E3375988, TLI: 12
INFO: Start transferring data files
INFO: Data files are transferred, time elapsed: 1s
INFO: wait for pg_stop_backup()
INFO: pg_stop_backup() successfully executed
INFO: stop_stream_lsn 0/E339CB90 currentpos 0/E339CB90
INFO: backup->stop_lsn 0/E339CB90
INFO: Getting the Recovery Time from WAL
INFO: Failed to find Recovery Time in WAL, forced to trust current_timestamp
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 0
INFO: Validating backup SO25GO
INFO: Backup SO25GO data files are valid
INFO: Backup SO25GO resident size: 55MB
INFO: Backup SO25GO completed
 Смотрим список резервных копий экземпляра:
[student@backuphost backups]$ pg_probackup show -B /backups --instance=bihadb
================================================================================================================================================
 Instance  Version  ID      Recovery Time                  Mode  WAL Mode  TLI   Time  Data   WAL  Zalg  Zratio  Start LSN   Stop LSN    Status 
================================================================================================================================================
 bihadb    16       SO25GO  2024-12-06 08:48:33.685609+03  FULL  STREAM    18/0   18s  39MB  16MB  none    1.00  0/E3375988  0/E339CB90  OK     
[student@backuphost backups]$ 

на хосте с СУБД удаляем наш кластер:
 bash-5.2 $ rm -R /var/lib/pgpro/ent-16/data/
bash-5.2$ 

восстанавливаемся с хоста где бекапы:
pg_probackup restore -B /backups -D /var/lib/pgpro/ent-16/data --instance=bihadb --remote-host=node3 --remote-user=postgres 

[student@backuphost backups]$ pg_probackup restore -B /backups -D /var/lib/pgpro/ent-16/data --instance=bihadb --remote-host=node3 --remote-user=postgres 
INFO: Tablespace 16384 will be restored using old path "/space"
INFO: Validating backup SO25GO
INFO: Backup SO25GO data files are valid
WARNING: Thread [1]: Could not read WAL record at 0/E339CB90: invalid record length at 0/E339CB90: wanted 28, got 0
INFO: Backup SO25GO WAL segments are valid
INFO: Backup SO25GO is valid.
INFO: Restoring the database from backup SO25GO on node3
INFO: Start restoring backup files. PGDATA size: 55MB
INFO: Backup files are restored. Transferred bytes: 55MB, time elapsed: 1s
INFO: Restore incremental ratio (less is better): 100% (55MB/55MB)
INFO: Syncing restored files to disk
INFO: Restored backup files are synced, time elapsed: 1s
INFO: Restore of backup SO25GO completed.
[student@backuphost backups]$ 

стартуем восстановленный сервер СУБД:
bash-5.2$ pg_ctl -D /var/lib/pgpro/ent-16/data/ start
ожидание запуска сервера....2024-12-06 09:32:57.237 MSK [119665] СООБЩЕНИЕ:  [BiHA] Initializing BiHA extension
2024-12-06 09:32:57.237 MSK [119665] СООБЩЕНИЕ:  [BiHA] Registering BiHA worker
2024-12-06 09:32:57.286 MSK [119665] СООБЩЕНИЕ:  Start CFS version 0.54 supported compression algorithms pglz,zlib,lz4,zstd encryption disabled GC enabled
2024-12-06 09:32:57.292 MSK [119665] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2024-12-06 09:32:57.292 MSK [119665] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен




