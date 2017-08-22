---
layout: post
title: Table(MView) online redefine
categories: Tec
tags: Oracle Database
date: 2017-08-22
---

### Create table for online redefine.
##### Redefine partitioned table.

```
  6   1  create table tab_redefine (
  7   2      tid     number(10)
  8   3    , cdate   date
  9   4    , udate   date
 10   5    , status  number(2)
 11   6    , flg     char(1)
 12   7* )
 13 SQL> /
 14 
 15 Table created.
 16 
 17 Elapsed: 00:00:00.03
 18 SQL>
```
- - -

### Create primary key.

```
 23   1* create unique index idxp_redefie on tab_redefine( tid )
 24 SQL> /
 25 
 26 Index created.
 27 
 28 Elapsed: 00:00:00.02
 29 SQL>
```
```
 34   1* alter table tab_redefine add constraints idxp_redefine primary key( tid )
 35 SQL> /
 36 
 37 Table altered.
 38 
 39 Elapsed: 00:00:00.10
 40 SQL>
```
- - -

### Insert test data.

```
 45   1  declare
 46   2    cnt number(10);
 47   3  begin
 48   4    cnt := 0;
 49   5    for i in 1..10000 loop
 50   6      insert into tab_redefine values( i
 51   7                                     , sysdate
 52   8                                     , sysdate
 53   9                                     , mod( i, 10 )
 54  10                                     , mod( trunc( dbms_random.value*10 ), 2 )
 55  11                                     );
 56  12      cnt := cnt + 1;
 57  13      if mod( cnt, 100 ) = 0 then
 58  14        commit;
 59  15      end if;
 60  16    end loop;
 61  17    commit;
 62  18* end;
 63 SQL> /
 64 
 65 PL/SQL procedure successfully completed.
 66 
 67 Elapsed: 00:00:00.70
 68 SQL>
 69 SQL> select count(*) from tab_redefine;
 70 
 71   COUNT(*)
 72 ----------
 73      10000
 74 
 75 1 row selected.
 76 
 77 Elapsed: 00:00:00.01
 78 SQL>
```
- - -

### Create index.

```
 83   1* create index idx1_redefine on tab_redefine( status )
 84 SQL> /
 85 
 86 Index created.
 87 
 88 Elapsed: 00:00:00.02
 89 SQL>
```
- - -

### Gather stats.

```
 94   1  begin
 95   2      dbms_stats.gather_table_stats (
 96   3          ownname => 'TEST01'
 97   4        , tabname => 'TAB_REDEFINE'
 98   5        , cascade => true
 99   6      );
100   7* end;
101 SQL> /
102 
103 PL/SQL procedure successfully completed.
104 
105 Elapsed: 00:00:01.52
106 SQL>
```
- - -

### Create MLOG.

```
111   1  create materialized view log on tab_redefine
112   2  with primary key
113   3* including new values
114 SQL> /
115 
116 Materialized view log created.
117 
118 Elapsed: 00:00:00.08
119 SQL>
```
- - -

### Create MView.

```
124   1  create materialized view mv_redefine (
125   2      tid
126   3    , cdate
127   4    , udate
128   5    , status
129   6    , flg
130   7    , cno
131   8  )
132   9    using no index
133  10    refresh fast
134  11    with primary key
135  12    as select tid
136  13            , cdate
137  14            , udate
138  15            , status
139  16            , flg
140  17            , mod( tid, 100 )
141  18*       from tab_redefine
142 SQL> /
143 
144 Materialized view created.
145 
146 Elapsed: 00:00:00.39
147 SQL>
```
- - -

### Confirm MLOG.

```
152   1  select log_owner
153   2       , master
154   3       , log_table
155   4       , object_id
156   5       , last_purge_date
157   6       , last_purge_status
158   7       , num_rows_purged
159   8*   from user_mview_logs
160 SQL> /
161 
162 LOG_OWNER        MASTER       LOG_TABLE                      OBJECT_ID    LAST_PURGE_DATE     LAST_PURGE_STATUS NUM_ROWS_PURGED
163 ---------------- ------------ ------------------------------ ------------ ------------------- ----------------- ---------------
164 TEST01           TAB_REDEFINE MLOG$_TAB_REDEFINE             NO           2017/08/22 10:47:27                 0               0
165 
166 1 row selected.
167 
168 Elapsed: 00:00:00.01
169 SQL>
```
- - -

### Confirm MView.

```
174   1  select owner
175   2       , mview_name
176   3       , last_refresh_type
177   4       , last_refresh_date
178   5       , compile_state
179   6       , staleness
180   7       , after_fast_refresh
181   8*   from user_mviews
182 SQL> /
183 
184 OWNER                MVIEW_NAME           LAST_REFRESH_TYPE    LAST_REFRESH_DATE   COMPILE_STATE       STALENESS           AFTER_FAST_REFRESH
185 -------------------- -------------------- -------------------- ------------------- ------------------- ------------------- -------------------
186 TEST01               MV_REDEFINE          COMPLETE             2017/08/22 10:47:27 VALID               FRESH               FRESH
187 
188 1 row selected.
189 
190 Elapsed: 00:00:00.04
191 SQL>
```
- - -

### Confirm object.

```
196   1  select object_type
197   2       , object_id
198   3       , object_name
199   4       , created
200   5       , last_ddl_time
201   6       , status
202   7    from user_objects
203   8*  order by 1, 2
204 SQL> /
205 
206 OBJECT_TYPE               OBJECT_ID OBJECT_NAME                    CREATED             LAST_DDL_TIME       STATUS
207 ----------------------- ----------- ------------------------------ ------------------- ------------------- -------
208 INDEX                         92822 IDXP_REDEFIE                   2017/08/22 10:38:52 2017/08/22 10:38:52 VALID
209 INDEX                         92823 IDX1_REDEFINE                  2017/08/22 10:44:37 2017/08/22 10:44:37 VALID
210 INDEX                         92829 I_MLOG$_TAB_REDEFINE           2017/08/22 10:46:20 2017/08/22 10:46:20 VALID
211 MATERIALIZED VIEW             92831 MV_REDEFINE                    2017/08/22 10:47:27 2017/08/22 10:47:27 VALID
212 TABLE                         92821 TAB_REDEFINE                   2017/08/22 10:36:50 2017/08/22 10:46:20 VALID
213 TABLE                         92827 MLOG$_TAB_REDEFINE             2017/08/22 10:46:20 2017/08/22 10:46:20 VALID
214 TABLE                         92828 RUPD$_TAB_REDEFINE             2017/08/22 10:46:20 2017/08/22 10:46:20 VALID
215 TABLE                         92830 MV_REDEFINE                    2017/08/22 10:47:27 2017/08/22 10:47:27 VALID
216 
217 8 rows selected.
218 
219 Elapsed: 00:00:00.03
220 SQL>
```
- - -

### Can redefine table.

```
225   1  begin
226   2  dbms_redefinition.can_redef_table( 'TEST01', 'TAB_REDEFINE', DBMS_REDEFINITION.CONS_USE_PK, null );
227   3* end;
228 SQL> /
229 
230 PL/SQL procedure successfully completed.
231 
232 Elapsed: 00:00:00.04
233 SQL>
```
procedure ok.
- - -

### Create partitioned table for redefine.

```
238   1  create table tab_redefine_new
239   2  partition by list ( status )
240   3  (
241   4      partition p000 values( 0 )
242   5    , partition p001 values( 1 )
243   6    , partition pdef values( default )
244   7  )
245   8  enable row movement
246   9* as select * from tab_redefine where 1 = 2
247 SQL> /
248 
249 Table created.
250 
251 Elapsed: 00:00:00.04
252 SQL>
```
- - -

### Create primary key.

```
257   1* create unique index idxp_redefine_new on tab_redefine_new( tid )
258 SQL> /
259 
260 Index created.
261 
262 Elapsed: 00:00:00.02
263 SQL>
```
```
268   1* alter table tab_redefine_new add constraints idxp_redefine_new primary key( tid )
269 SQL> /
270 
271 Table altered.
272 
273 Elapsed: 00:00:00.03
274 SQL>
```
- - -

### Create local index.

```
279   1* create index idx1_redefine_new on tab_redefine_new( status ) local
280 SQL> /
281 
282 Index created.
283 
284 Elapsed: 00:00:00.09
285 SQL>
```
- - -

### Create MLOG.

```
290   1  create materialized view log on tab_redefine_new
291   2  with primary key
292   3* including new values
293 SQL> /
294 
295 Materialized view log created.
296 
297 Elapsed: 00:00:00.05
298 SQL>
```
- - -

### Start redefine table.

```
303   1  begin
304   2      dbms_redefinition.start_redef_table (
305   3          uname        => 'TEST01'
306   4        , orig_table   => 'TAB_REDEFINE'
307   5        , int_table    => 'TAB_REDEFINE_NEW'
308   6        , col_mapping  => NULL
309   7        , options_flag => DBMS_REDEFINITION.CONS_USE_PK
310   8        , part_name    => NULL
311   9      );
312  10* end;
313 SQL> /
314 
315 PL/SQL procedure successfully completed.
316 
317 Elapsed: 00:00:03.22
318 SQL>
```
- - -

### Register dependent object.

```
323   1  begin
324   2  dbms_redefinition.register_dependent_object(
325   3      uname         => 'TEST01',
326   4      orig_table    => 'TAB_REDEFINE',
327   5      int_table     => 'TAB_REDEFINE_NEW',
328   6      dep_type      => DBMS_REDEFINITION.CONS_MVLOG,
329   7      dep_owner     => 'TEST01',
330   8      dep_orig_name => 'MLOG$_TAB_REDEFINE',
331   9      dep_int_name  => 'MLOG$_TAB_REDEFINE_NEW'
332  10  );
333  11* END;
334 SQL> /
335 
336 PL/SQL procedure successfully completed.
337 
338 Elapsed: 00:00:00.02
339 SQL>
```
- - -

### Copy table dependents.

```
344   1  declare
345   2      num_errors PLS_INTEGER;
346   3  begin
347   4      dbms_redefinition.copy_table_dependents (
348   5          uname            => 'TEST01'
349   6        , orig_table       => 'TAB_REDEFINE'
350   7        , int_table        => 'TAB_REDEFINE_NEW'
351   8        , copy_indexes     => DBMS_REDEFINITION.CONS_ORIG_PARAMS
352   9        , copy_triggers    => TRUE
353  10        , copy_constraints => TRUE
354  11        , copy_privileges  => TRUE
355  12        , ignore_errors    => TRUE
356  13        , num_errors       => num_errors
357  14        , copy_statistics  => TRUE
358  15        , copy_mvlog       => TRUE
359  16      );
360  17* end;
361 SQL> /
362 
363 PL/SQL procedure successfully completed.
364 
365 Elapsed: 00:00:07.23
366 SQL>
```
- - -

### Confirm redefine errors.

```
372   1  select object_type
373   2       , object_owner
374   3       , object_name
375   4       , base_table_owner
376   5       , base_table_name
377   6       , ddl_txt
378   7*   from dba_redefinition_errors
379 SQL> /
380 
381 OBJECT_TYPE  OBJECT_OWNER         OBJECT_NAME                    BASE_TABLE_OWNER     BASE_TABLE_NAME      DDL_TXT
382 ------------ -------------------- ------------------------------ -------------------- -------------------- --------------------------------------------------------------------------------
383 INDEX        TEST01               IDXP_REDEFIE                   TEST01               TAB_REDEFINE         CREATE UNIQUE INDEX "TEST01"."TMP$$_IDXP_REDEFIE0" ON "TEST01"."TAB_REDEFINE_NEW
384                                                                                                            " ("TID")
385                                                                                                              PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS
386                                                                                                              STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
387                                                                                                              PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
388                                                                                                              BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
389                                                                                                              TABLESPACE "USERS"
390 
391 INDEX        TEST01               IDX1_REDEFINE                  TEST01               TAB_REDEFINE         CREATE INDEX "TEST01"."TMP$$_IDX1_REDEFINE0" ON "TEST01"."TAB_REDEFINE_NEW" ("ST
392                                                                                                            ATUS")
393                                                                                                              PCTFREE 10 INITRANS 2 MAXTRANS 255 COMPUTE STATISTICS
394                                                                                                              STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
395                                                                                                              PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
396                                                                                                              BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
397                                                                                                              TABLESPACE "USERS"
398 
399 CONSTRAINT   TEST01               IDXP_REDEFINE                  TEST01               TAB_REDEFINE         ALTER TABLE "TEST01"."TAB_REDEFINE_NEW" ADD CONSTRAINT "TMP$$_IDXP_REDEFINE0" PR
400                                                                                                            IMARY KEY ("TID")
401                                                                                                              USING INDEX PCTFREE 10 INITRANS 2 MAXTRANS 255
402                                                                                                              STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
403                                                                                                              PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
404                                                                                                              BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
405                                                                                                              TABLESPACE "USERS"  ENABLE NOVALIDATE
406 
407 
408 3 rows selected.
409 
410 Elapsed: 00:00:00.01
411 SQL>
```
The objects already defined.
- - -

### Sync interim table.

```
416   1  begin
417   2      dbms_redefinition.sync_interim_table (
418   3          uname      => 'TEST01'
419   4        , orig_table => 'TAB_REDEFINE'
420   5        , int_table  => 'TAB_REDEFINE_NEW'
421   6      );
422   7* end;
423 SQL> /
424 
425 PL/SQL procedure successfully completed.
426 
427 Elapsed: 00:00:00.10
428 SQL>
```
- - -

### Count interim table.

```
433   1  select 'TAB_REDEFINE' tname, count(*) from TAB_REDEFINE union all
434   2* select 'TAB_REDEFINE_NEW' tname, count(*) from TAB_REDEFINE_NEW
435 SQL> /
436 
437 TNAME              COUNT(*)
438 ---------------- ----------
439 TAB_REDEFINE          10000
440 TAB_REDEFINE_NEW      10000
441 
442 2 rows selected.
443 
444 Elapsed: 00:00:00.01
445 SQL>
```
- - -

### Finish redefine process.

```
450   1  begin
451   2      dbms_redefinition.finish_redef_table (
452   3          uname      => 'TEST01'
453   4        , orig_table => 'TAB_REDEFINE'
454   5        , int_table  => 'TAB_REDEFINE_NEW'
455   6      );
456   7* end;
457 SQL> /
458 
459 PL/SQL procedure successfully completed.
460 
461 Elapsed: 00:00:00.70
462 SQL>
```
- - -

### Confirm MLOG.

```
467   1  select log_owner
468   2       , master
469   3       , log_table
470   4       , object_id
471   5       , last_purge_date
472   6       , last_purge_status
473   7       , num_rows_purged
474   8*   from user_mview_logs
475 SQL> /
476 
477 LOG_OWNER        MASTER               LOG_TABLE                      OBJECT_ID    LAST_PURGE_DATE     LAST_PURGE_STATUS NUM_ROWS_PURGED
478 ---------------- -------------------- ------------------------------ ------------ ------------------- ----------------- ---------------
479 TEST01           TAB_REDEFINE         MLOG$_TAB_REDEFINE             NO           2017/08/22 11:02:57                 0               0
480 TEST01           TAB_REDEFINE_NEW     MLOG$_TAB_REDEFINE_NEW         NO           2017/08/22 11:11:00                 0               0
481 
482 2 rows selected.
483 
484 Elapsed: 00:00:00.00
485 SQL>
```
- - -

### Confirm MView.

```
490   1  select owner
491   2       , mview_name
492   3       , last_refresh_type
493   4       , last_refresh_date
494   5       , compile_state
495   6       , staleness
496   7       , after_fast_refresh
497   8*   from user_mviews
498 SQL> /
499 
500 OWNER                MVIEW_NAME           LAST_REFRESH_TYPE    LAST_REFRESH_DATE   COMPILE_STATE       STALENESS           AFTER_FAST_REFRESH
501 -------------------- -------------------- -------------------- ------------------- ------------------- ------------------- -------------------
502 TEST01               MV_REDEFINE          COMPLETE             2017/08/22 10:47:27 NEEDS_COMPILE       NEEDS_COMPILE       NEEDS_COMPILE
503 
504 1 row selected.
505 
506 Elapsed: 00:00:00.06
507 SQL>
```
Table status NEEDS_COMPILE.
- - -

### Confirm objects.

```
512   1  select object_type
513   2       , object_id
514   3       , object_name
515   4       , last_ddl_time
516   5       , status
517   6    from user_objects
518   7*  order by 1, 3
519 SQL> /
520 
521 OBJECT_TYPE               OBJECT_ID OBJECT_NAME                    LAST_DDL_TIME       STATUS
522 ----------------------- ----------- ------------------------------ ------------------- -------
523 INDEX                         92823 IDX1_REDEFINE                  2017/08/22 10:44:37 VALID
524 INDEX                         92845 IDX1_REDEFINE_NEW              2017/08/22 11:02:58 VALID
525 INDEX                         92822 IDXP_REDEFIE                   2017/08/22 10:38:52 VALID
526 INDEX                         92836 IDXP_REDEFINE_NEW              2017/08/22 11:02:58 VALID
527 INDEX                         92829 I_MLOG$_TAB_REDEFINE           2017/08/22 10:46:20 VALID
528 INDEX                         92843 I_MLOG$_TAB_REDEFINE_NEW       2017/08/22 11:02:10 VALID
529 INDEX PARTITION               92846 IDX1_REDEFINE_NEW              2017/08/22 11:02:58 VALID
530 INDEX PARTITION               92848 IDX1_REDEFINE_NEW              2017/08/22 11:02:58 VALID
531 INDEX PARTITION               92847 IDX1_REDEFINE_NEW              2017/08/22 11:02:58 VALID
532 MATERIALIZED VIEW             92831 MV_REDEFINE                    2017/08/22 10:47:27 INVALID
533 TABLE                         92841 MLOG$_TAB_REDEFINE             2017/08/22 10:46:20 VALID
534 TABLE                         92827 MLOG$_TAB_REDEFINE_NEW         2017/08/22 11:11:00 VALID
535 TABLE                         92830 MV_REDEFINE                    2017/08/22 10:47:27 VALID
536 TABLE                         92828 RUPD$_TAB_REDEFINE             2017/08/22 10:46:20 VALID
537 TABLE                         92842 RUPD$_TAB_REDEFINE_NEW         2017/08/22 11:02:10 VALID
538 TABLE                         92832 TAB_REDEFINE                   2017/08/22 11:11:00 VALID
539 TABLE                         92821 TAB_REDEFINE_NEW               2017/08/22 11:11:00 VALID
540 TABLE PARTITION               92833 TAB_REDEFINE                   2017/08/22 10:58:46 VALID
541 TABLE PARTITION               92834 TAB_REDEFINE                   2017/08/22 10:58:46 VALID
542 TABLE PARTITION               92835 TAB_REDEFINE                   2017/08/22 10:58:46 VALID
543 
544 20 rows selected.
545 
546 Elapsed: 00:00:00.03
547 SQL>
```
MView status invalid.
- - -

### MView recompile.

```
552   1* alter materialized view mv_redefine compile
553 SQL> /
554 
555 Materialized view altered.
556 
557 Elapsed: 00:00:00.22
558 SQL>
```
- - -

### Confirm MView again.

```
586   1  select owner
587   2       , mview_name
588   3       , last_refresh_type
589   4       , last_refresh_date
590   5       , compile_state
591   6       , staleness
592   7       , after_fast_refresh
593   8*   from user_mviews
594 SQL> /
595 
596 OWNER                MVIEW_NAME           LAST_REFRESH_TYPE    LAST_REFRESH_DATE   COMPILE_STATE       STALENESS           AFTER_FAST_REFRESH
597 -------------------- -------------------- -------------------- ------------------- ------------------- ------------------- -------------------
598 TEST01               MV_REDEFINE          COMPLETE             2017/08/22 10:47:27 VALID               UNUSABLE            NA
599 
600 1 row selected.
601 
602 Elapsed: 00:00:00.04
603 SQL>
```
STALENESS=UNUSABLE
- - -

### Try fast refresh.

```
605 SQL> execute dbms_mview.refresh( 'mv_redefine', 'f' );
606 BEGIN dbms_mview.refresh( 'mv_redefine', 'f' ); END;
607 
608 *
609 ERROR at line 1:
610 ORA-12034: materialized view log on "TEST01"."TAB_REDEFINE" younger than last refresh
611 ORA-06512: at "SYS.DBMS_SNAPSHOT", line 2821
612 ORA-06512: at "SYS.DBMS_SNAPSHOT", line 3058
613 ORA-06512: at "SYS.DBMS_SNAPSHOT", line 3017
614 ORA-06512: at line 1
615 
616 
617 Elapsed: 00:00:00.27
618 SQL>
```
See failed.
- - -

### Exec complete refresh.

```
620 SQL> execute dbms_mview.refresh( 'mv_redefine', 'c' );
621 
622 PL/SQL procedure successfully completed.
623 
624 Elapsed: 00:00:00.29
625 SQL>
```
- - -

### Confirm MView again.

```
630   1  select owner
631   2       , mview_name
632   3       , last_refresh_type
633   4       , last_refresh_date
634   5       , compile_state
635   6       , staleness
636   7       , after_fast_refresh
637   8*   from user_mviews
638 SQL> /
639 
640 OWNER                MVIEW_NAME           LAST_REFRESH_TYPE    LAST_REFRESH_DATE   COMPILE_STATE       STALENESS           AFTER_FAST_REFRESH
641 -------------------- -------------------- -------------------- ------------------- ------------------- ------------------- -------------------
642 TEST01               MV_REDEFINE          COMPLETE             2017/08/22 11:17:42 VALID               FRESH               FRESH
643 
644 1 row selected.
645 
646 Elapsed: 00:00:00.04
647 SQL>
```
Status is OK.
- - -

### Try again fast refresh.

```
649 SQL> execute dbms_mview.refresh( 'mv_redefine', 'f' );
650 
651 PL/SQL procedure successfully completed.
652 
653 Elapsed: 00:00:00.07
654 SQL>
```
- - -
