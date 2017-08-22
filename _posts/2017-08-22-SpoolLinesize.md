---
layout: post
title: Spool Linesize
categories: Tec
tags: Oracle Database
date: 2017-08-22
---

### Count table.

```
SQL> select count(*) from tab_p_test;

  COUNT(*)
----------
    104900

1 row selected.

Elapsed: 00:00:00.01
SQL> 
```
- - -

### Spool set linesize 32767.

```
bash-3.2$ cat spool_test_lines32767.sql 
set echo off
set linesize 32767
set pagesize 0
set feedback off
set trimspool on
set timing on
set term off
set verify off
spool spool_test_lines32767.log
select /*+ full(t) */ * from tab_p_test t;
spool off
exit;
bash-3.2$ 
```
```
bash-3.2$ sqlplus -s -L TEST01@ORA12PDB @spool_test_lines32767

bash-3.2$ wc -l spool_test_lines32767.log 
  104901 spool_test_lines32767.log
bash-3.2$ 
bash-3.2$ tail -1 spool_test_lines32767.log 
Elapsed: 00:00:12.73
bash-3.2$ 
```
12 sec
- - -

### Spool set linesize 1000.

```
bash-3.2$ cat spool_test_lines1000.sql 
set echo off
set linesize 1000 
set pagesize 0
set feedback off
set trimspool on
set timing on
set term off
set verify off
spool spool_test_lines1000.log
select /*+ full(t) */ * from tab_p_test t;
spool off
exit;
bash-3.2$ 
```
```
bash-3.2$ sqlplus -s -L TEST01@ORA12PDB @spool_test_lines1000

bash-3.2$ wc -l spool_test_lines1000.log 
  104901 spool_test_lines1000.log
bash-3.2$ 
bash-3.2$ tail -1 spool_test_lines1000.log 
Elapsed: 00:00:03.67
bash-3.2$ 
```
3 sec
- - -
