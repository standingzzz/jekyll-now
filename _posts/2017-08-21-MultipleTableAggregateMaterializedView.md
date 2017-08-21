---
layout: post
title: Create a aggregate materialized view of a multiple table
categories: Tec
tags: Oracle Database
date: 2017-08-21
---

### Create table 1

```
  1  create table tab_mv_test1
  2  (
  3       mid        number(10)
  4     , cdate      date
  5* )
SQL> /

Table created.

Elapsed: 00:00:00.05
SQL>
```
- - -

### Create table 2

```
  1  create table tab_mv_test2
  2  (
  3       mid        number(10)
  4     , cdate      date
  5* )
SQL> /

Table created.

Elapsed: 00:00:00.03
SQL>
```
- - -

### Create MLOG on table 1

```
  1  create materialized view log on tab_mv_test1
  2  with rowid, sequence( mid )
  3* including new values
SQL> /

Materialized view log created.

Elapsed: 00:00:00.15
SQL>
```
- - -

### Create MLOG on table 2

```
  1  create materialized view log on tab_mv_test2
  2  with rowid, sequence( mid )
  3* including new values
SQL> /

Materialized view log created.

Elapsed: 00:00:00.03
SQL>
```
- - -

### Create MVIEW inner join

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select t1.mid
  5          , count(*)
  6       from tab_mv_test1 t1
  7      inner join tab_mv_test2 t2
  8         on t2.mid = t1.mid
  9      group by
 10*           t1.mid
SQL> /

Materialized view created.

Elapsed: 00:00:01.02
SQL>
```
```
SQL> execute DBMS_MVIEW.REFRESH( 'mv_mv_test', 'f' );

PL/SQL procedure successfully completed.

Elapsed: 00:00:00.58
SQL>
```
- - -

### Create MVIEW inner join erace count(*)

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select t1.mid
  5       from tab_mv_test1 t1
  6      inner join tab_mv_test2 t2
  7         on t2.mid = t1.mid
  8      group by
  9*           t1.mid
SQL> /
       on t2.mid = t1.mid
                   *
ERROR at line 7:
ORA-12015: cannot create a fast refresh materialized view from a complex query


Elapsed: 00:00:00.01
SQL>
```

### Create MVIEW left join

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select t1.mid
  5          , count(*)
  6       from tab_mv_test1 t1
  7       left join tab_mv_test2 t2
  8         on t2.mid = t1.mid
  9      group by
 10*           t1.mid
SQL> /
       on t2.mid = t1.mid
                   *
ERROR at line 8:
ORA-12015: cannot create a fast refresh materialized view from a complex query


Elapsed: 00:00:00.02
SQL>
```
- - -

### Create MVIEW full outer join

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select t1.mid
  5          , count(*)
  6       from tab_mv_test1 t1
  7       full outer join tab_mv_test2 t2
  8         on t2.mid = t1.mid
  9      group by
 10*           t1.mid
SQL> /
       on t2.mid = t1.mid
                   *
ERROR at line 8:
ORA-12015: cannot create a fast refresh materialized view from a complex query


Elapsed: 00:00:00.01
SQL>
```
- - - 

### Create MVIEW table 1 union all table 2

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select t1.mid
  5       from tab_mv_test1 t1
  6     union all
  7     select t2.mid
  8*      from tab_mv_test2 t2
SQL> /
     from tab_mv_test2 t2
          *
ERROR at line 8:
ORA-12015: cannot create a fast refresh materialized view from a complex query


Elapsed: 00:00:00.02
SQL>
```
- - -

### Create MVIEW sum function

```
  1  create materialized view mv_mv_test
  2  using no index
  3  refresh fast with rowid
  4  as select to_char('cdate', 'yyyymm') cdate
  5          , sum(mid)
  6       from tab_mv_test1 t1
  7      group by
  8*           to_char('cdate', 'yyyymm')
SQL> /

Materialized view created.

Elapsed: 00:00:00.06
SQL>
```
```
SQL> execute DBMS_MVIEW.REFRESH( 'mv_mv_test', 'f' );

PL/SQL procedure successfully completed.

Elapsed: 00:00:00.08
SQL>
```
- - -


