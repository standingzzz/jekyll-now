---
layout: post
title: Create test data
categories: Tec
tags: Oracle Database
date: 2017-08-21
---

### Create range-partition table.

```
  1  create table tab_p_test
  2  (
  3      pid        number(5)
  4    , pdate      date
  5    , status     char(1)
  6  )
  7  partition by range( pid )
  8  (
  9      partition p001 values less than (  1000 )
 10    , partition p002 values less than (  2000 )
 11    , partition p010 values less than ( 10000 )
 12    , partition pmax values less than ( maxvalue )
 13  )
 14* enable row movement
SQL> /

Table created.

Elapsed: 00:00:00.02
SQL>
```
- - -

### Create primary key.

```
  1* create unique index idxp_p_test on tab_p_test( pid ) local
SQL> /

Index created.

Elapsed: 00:00:00.02
SQL>
```

```
  1* alter table tab_p_test add constraints idxp_p_test primary key( pid )
SQL> /

Table altered.

Elapsed: 00:00:00.01
SQL>
```
- - -

### Using to insert test data.

```
SQL> set serveroutput on format wrapped

  1  declare
  2      sdate          varchar2(30);
  3      edate          varchar2(30);
  4      insert_rows    number(10);
  5  begin
  6      select to_char( sysdate, 'yyyy/mm/dd hh24:mi:ss' ) into sdate from dual;
  7      insert_rows := 0;
  8      for i in 1..5000 loop
  9          insert into tab_p_test values ( i, sysdate, DBMS_RANDOM.STRING( 'a', 1 ) );
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
START[2017/08/21 12:29:44] end[2017/08/21 12:29:44]

PL/SQL procedure successfully completed.

Elapsed: 00:00:00.44
SQL>
```
- - -

### Create local index.

```
  1* create index idx1_p_test on tab_p_test( status, pid ) local
SQL> /

Index created.

Elapsed: 00:00:00.12
SQL>
```
- - -

### Confirm object.

```
  1  select object_name
  2       , object_id
  3       , data_object_id
  4       , object_type
  5       , created
  6       , last_ddl_time
  7       , status
  8    from user_objects
  9   order by
 10         object_type
 11       , object_name
 12       , object_id
 13*      , data_object_id
SQL> /

OBJECT_NAME                     OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE             CREATED             LAST_DDL_TIME       STATUS
------------------------------ ---------- -------------- ----------------------- ------------------- ------------------- -------
IDX1_P_TEST                         92798                INDEX                   2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST                         92793                INDEX                   2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDX1_P_TEST                         92799          92799 INDEX PARTITION         2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDX1_P_TEST                         92800          92800 INDEX PARTITION         2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDX1_P_TEST                         92801          92801 INDEX PARTITION         2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDX1_P_TEST                         92802          92802 INDEX PARTITION         2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST                         92794          92794 INDEX PARTITION         2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDXP_P_TEST                         92795          92795 INDEX PARTITION         2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDXP_P_TEST                         92796          92796 INDEX PARTITION         2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDXP_P_TEST                         92797          92797 INDEX PARTITION         2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
TAB_P_TEST                          92787                TABLE                   2017/08/21 12:27:13 2017/08/21 12:30:15 VALID
TAB_P_TEST                          92788          92788 TABLE PARTITION         2017/08/21 12:27:13 2017/08/21 12:27:13 VALID
TAB_P_TEST                          92789          92789 TABLE PARTITION         2017/08/21 12:27:13 2017/08/21 12:27:13 VALID
TAB_P_TEST                          92790          92790 TABLE PARTITION         2017/08/21 12:27:13 2017/08/21 12:27:13 VALID
TAB_P_TEST                          92791          92791 TABLE PARTITION         2017/08/21 12:27:13 2017/08/21 12:27:13 VALID

15 rows selected.

Elapsed: 00:00:00.04
SQL>
```
- - -

### Confirm table.

```
  1  select pt.partition_name
  2	  , pt.high_value
  3	  , (select key.column_name
  4	       from user_part_key_columns key
  5	      where key.name = pt.table_name
  6		and rownum = 1) key_column_name
  7	  , pt.last_analyzed
  8	  , pt.num_rows
  9	  , pt.compression
 10    from user_tab_partitions pt
 11   where pt.table_name = 'TAB_P_TEST'
 12   order by
 13*	    pt.partition_position
SQL> /

PARTITION_NAME		       HIGH_VALUE   KEY_COLUMN_NAME	 LAST_ANALYZED	       NUM_ROWS COMPRESS
------------------------------ ------------ -------------------- ------------------- ---------- --------
P001			       1000	    PID 		 2017/08/21 12:56:59	    999 DISABLED
P002			       2000	    PID 		 2017/08/21 12:56:59	   1000 DISABLED
P010			       10000	    PID 		 2017/08/21 12:56:59	   3001 DISABLED
PMAX			       MAXVALUE     PID 		 2017/08/21 12:56:59	      0 DISABLED

4 rows selected.

Elapsed: 00:00:00.06
SQL> 
```
- - -

### Confirm index.

```
  1  select col.index_name
  2	  , col.column_name
  3	  , prt.partition_name
  4	  , prt.high_value
  5	  , prt.status
  6	  , prt.num_rows
  7	  , prt.last_analyzed
  8    from user_ind_columns col
  9   inner join user_ind_partitions prt
 10	 on prt.index_name = col.index_name
 11   where col.index_name in ('IDXP_P_TEST', 'IDX1_P_TEST')
 12   order by
 13	    col.index_name
 14	  , col.column_position
 15*	  , prt.partition_position
 16  /

INDEX_NAME		       COLUMN_NAME	    PARTITION_NAME		   HIGH_VALUE	STATUS	   NUM_ROWS LAST_ANALYZED
------------------------------ -------------------- ------------------------------ ------------ -------- ---------- -------------------
IDX1_P_TEST		       STATUS		    P001			   1000 	USABLE		999 2017/08/21 12:57:39
IDX1_P_TEST		       STATUS		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:39
IDX1_P_TEST		       STATUS		    P010			   10000	USABLE	       3001 2017/08/21 12:57:39
IDX1_P_TEST		       STATUS		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    P001			   1000 	USABLE		999 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    P010			   10000	USABLE	       3001 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDXP_P_TEST		       PID		    P001			   1000 	USABLE		999 2017/08/21 12:57:51
IDXP_P_TEST		       PID		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:51
IDXP_P_TEST		       PID		    P010			   10000	USABLE	       3001 2017/08/21 12:57:51
IDXP_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:51

12 rows selected.

Elapsed: 00:00:00.15
SQL> 
```
- - -
