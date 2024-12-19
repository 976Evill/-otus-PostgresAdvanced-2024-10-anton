clickhouse
create table test1 (id UInt64, value String ) ENGINE=MergeTree ORDER BY id;
insert into test1 select number, number from numbers(15000000);
SELECT sum(id) FROM test1;
e5fb18feb7e9 :) SELECT sum(id) FROM test1;

SELECT sum(id)
FROM test1

Query id: 54991b09-4513-4a0f-9f36-38db86e4e180

   ┌─────────sum(id)─┐
1. │ 112499992500000 │ -- 112.50 trillion
   └─────────────────┘
↙ Progress: 15.00 million rows, 120.00 MB (240.07 million rows/s., 1.92 GB/s.)  ← Progress: 15.00 million rows, 120.00 MB (240.07 million rows/s., 1.92 GB/s.)  
1 row in set. Elapsed: 0.063 sec. Processed 15.00 million rows, 120.00 MB (238.41 million rows/s., 1.91 GB/s.)
Peak memory usage: 343.66 KiB.

e5fb18feb7e9 :) 




ванильный POSTGRESQL:
create table test2 (id int, desct text);  
insert into test2 ( id, desct)  SELECT generate_series(1,15000000) AS id, md5(random()::text) AS descr;
antonlebedev=# explain analyze SELECT sum(id) FROM test2;
                                                                  QUERY PLAN                                                                  
----------------------------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=204125.22..204125.23 rows=1 width=8) (actual time=2522.984..2526.005 rows=1 loops=1)
   ->  Gather  (cost=204125.00..204125.21 rows=2 width=8) (actual time=2522.618..2525.978 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Partial Aggregate  (cost=203125.00..203125.01 rows=1 width=8) (actual time=2493.998..2494.031 rows=1 loops=3)
               ->  Parallel Seq Scan on test2  (cost=0.00..187500.00 rows=6250000 width=4) (actual time=0.807..1864.549 rows=5000000 loops=3)
 Planning Time: 0.318 ms
 Execution Time: 2526.588 ms
(8 rows)
 
 Выигрыш CLICKHOUSE по сравнению с POSTGRESQL  в производительности подобных запросов приблизительно 40000 раз.

 Поэксперементровал в порядке общего ознакомления с airbyte и вашим dataset из git. 





