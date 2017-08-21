---
layout: post
title: Sequence maxValue test
categories: Tec
tags: Oracle Database
date: 2017-08-21
---

## nocycle sequence
### Create nocycle sequence

```
  1  create sequence seq_pid
  2  increment by 1
  3  start with 1
  4  maxvalue 99999
  5  minvalue 1
  6  nocycle
  7  cache 100
  8* order
SQL> /

Sequence created.

Elapsed: 00:00:00.01
SQL>
```
- - -

### Create table

```
  1  create table tab_p_test
  2  (
  3      pid        number(5)
  4    , pdate      date
  5    , status     char(1)
  6* )
SQL> /

Table created.

Elapsed: 00:00:00.02
SQL>
```

### Insert data seq_pid.nextval

```
SQL> set serveroutput on format wrapped
  1  declare
  2      sdate          varchar2(30);
  3      edate          varchar2(30);
  4      insert_rows    number(10);
  5  begin
  6      select to_char( sysdate, 'yyyy/mm/dd hh24:mi:ss' ) into sdate from dual;
  7      insert_rows := 0;
  8      for i in 1..99999 loop
  9          insert into tab_p_test values ( seq_pid.nextval, sysdate, DBMS_RANDOM.STRING( 'a', 1 ) );
 10          insert_rows := insert_rows + 1;
 11          if mod( insert_rows, 100 ) = 0 then
 12              commit;
 13          end if;
 14      end loop;
 15      commit;
 16      select to_char( sysdate, 'yyyy/mm/dd hh24:mi:ss' ) into edate from dual;
 17      DBMS_OUTPUT.PUT_LINE( 'START[' || sdate || '] end[' || edate || ']');
 18  exception
 19      when others then
 20          DBMS_OUTPUT.PUT_LINE( sqlerrm );
 21          rollback;
 22* end;
 23  /
START[2017/08/21 16:01:20] end[2017/08/21 16:01:25]

PL/SQL procedure successfully completed.

Elapsed: 00:00:05.65
SQL>
```
- - -

### Confirm sequence

```
  1  select sequence_name
  2       , max_value
  3       , increment_by
  4       , cycle_flag
  5       , order_flag
  6       , cache_size
  7       , last_number
  8    from user_sequences
  9*  where sequence_name = 'SEQ_PID'
SQL> /

SEQUENCE_NAME         MAX_VALUE INCREMENT_BY CYCLE_FLAG ORDER_FLAG CACHE_SIZE LAST_NUMBER
-------------------- ---------- ------------ ---------- ---------- ---------- -----------
SEQ_PID                   99999            1 N          Y                 100      100000

1 row selected.

Elapsed: 00:00:00.01
SQL>
```
- - -

### One more seq_pid.nextval

```
SQL> select seq_pid.nextval from dual;
select seq_pid.nextval from dual
       *
ERROR at line 1:
ORA-08004: sequence SEQ_PID.NEXTVAL exceeds MAXVALUE and cannot be instantiated


Elapsed: 00:00:00.02
SQL>
```
- - -

