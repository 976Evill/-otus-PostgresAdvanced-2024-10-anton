max_connections = 140
shared_buffers = 256MB
effective_cache_size = 768MB
maintenance_work_mem = 128MB
checkpoint_completion_target = 0.9
wal_buffers = 7864kB
default_statistics_target = 500
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 1638kB
huge_pages = off
min_wal_size = 4GB
max_wal_size = 16GB

Создал виртуальную машину CENTOS 8 с 1 процессором и 1 Гб оперативной памяти
Установил ванильный 15-й postgresql
Создал тестовую базу  benchmark
Заполнил данными и произвёл первой измерение с настройками по умолчанию:
bash-4.4$ /usr/pgsql-15/bin/pgbench -i -s 50 benchmark
dropping old tables...
creating tables...
generating data (client-side)...
5000000 of 5000000 tuples (100%) done (elapsed 4.59 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 8.99 s (drop tables 0.01 s, create tables 0.01 s, client-side generate 4    

bash-4.4$ /usr/pgsql-15/bin/pgbench -c 10 -j 2 -t 10000 benchmark
pgbench (15.10)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 3.346 ms
initial connection time = 12.690 ms
tps = 2988.972425 (without initial connection time)
bash-4.4$
Вставил  рекомендуемые настройки в postgresql.conf 

shared_buffers = 256MB
effective_cache_size = 768MB
maintenance_work_mem = 128MB
checkpoint_completion_target = 0.9
wal_buffers = 7864kB
default_statistics_target = 500
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 1638kB
huge_pages = off
min_wal_size = 4GB
max_wal_size = 16GB

Произвёл повторный запуск:
                                                                                                                                                         .61 s, vacuum 1.86 s, primary keys 2.50 s).
bash-4.4$ /usr/pgsql-15/bin/pgbench -c 10 -j 2 -t 10000 benchmark
pgbench (15.10)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 3.095 ms
initial connection time = 12.238 ms
tps = 3231.356776 (without initial connection time)
bash-4.4$

Наблюдаю рост tps.
