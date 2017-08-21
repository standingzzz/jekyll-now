---
layout: post
title: Drop partition test
categories: Tec
tags: Oracle Database
date: 2017-08-21
---

[prepared test data](https://standingzzz.github.io/CreateTestData/)
- - -

### Drop partition p001

```
  1* alter table tab_p_test drop partition p001
SQL> /

Table altered.

Elapsed: 00:00:00.15
SQL> 
```
- - -

### Confirm object part 1

```
  1  select object_name		
  2	  , object_id		
  3	  , data_object_id 	
  4	  , object_type		
  5	  , created		
  6	  , last_ddl_time		
  7	  , status 		
  8    from user_objects
  9   order by
 10	    object_type
 11	  , object_name
 12	  , object_id
 13*	  , data_object_id
SQL> /

OBJECT_NAME			OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE		 CREATED	     LAST_DDL_TIME	 STATUS
------------------------------ ---------- -------------- ----------------------- ------------------- ------------------- -------
IDX1_P_TEST			    92798		 INDEX			 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST			    92793		 INDEX			 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDX1_P_TEST			    92800	   92800 INDEX PARTITION	 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDX1_P_TEST			    92801	   92801 INDEX PARTITION	 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDX1_P_TEST			    92802	   92802 INDEX PARTITION	 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST			    92795	   92795 INDEX PARTITION	 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDXP_P_TEST			    92796	   92796 INDEX PARTITION	 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDXP_P_TEST			    92797	   92797 INDEX PARTITION	 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
TAB_P_TEST			    92787		 TABLE			 2017/08/21 12:27:13 2017/08/21 15:09:24 VALID
TAB_P_TEST			    92789	   92789 TABLE PARTITION	 2017/08/21 12:27:13 2017/08/21 12:27:13 VALID
TAB_P_TEST			    92790	   92790 TABLE PARTITION	 2017/08/21 12:27:13 2017/08/21 12:27:13 VALID
TAB_P_TEST			    92791	   92791 TABLE PARTITION	 2017/08/21 12:27:13 2017/08/21 12:27:13 VALID

12 rows selected.

Elapsed: 00:00:00.04
SQL> 
```
- - -

### Confirm table part 1

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
P002			       2000	    PID 		 2017/08/21 12:56:59	   1000 DISABLED
P010			       10000	    PID 		 2017/08/21 12:56:59	   3001 DISABLED
PMAX			       MAXVALUE     PID 		 2017/08/21 12:56:59	      0 DISABLED

3 rows selected.

Elapsed: 00:00:00.06
SQL> 
```
- - -

### Confirm index part 1

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
SQL> /

INDEX_NAME		       COLUMN_NAME	    PARTITION_NAME		   HIGH_VALUE	STATUS	   NUM_ROWS LAST_ANALYZED
------------------------------ -------------------- ------------------------------ ------------ -------- ---------- -------------------
IDX1_P_TEST		       STATUS		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:39
IDX1_P_TEST		       STATUS		    P010			   10000	USABLE	       3001 2017/08/21 12:57:39
IDX1_P_TEST		       STATUS		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    P010			   10000	USABLE	       3001 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDXP_P_TEST		       PID		    P002			   2000 	USABLE	       1000 2017/08/21 12:57:51
IDXP_P_TEST		       PID		    P010			   10000	USABLE	       3001 2017/08/21 12:57:51
IDXP_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:51

9 rows selected.

Elapsed: 00:00:00.19
SQL> 
```
- - -

### Drop partition p002

```
  1* alter table tab_p_test drop partition p002
SQL> /

Table altered.

Elapsed: 00:00:00.03
SQL> 
```
- - -

### Drop partition P010

```
  1* alter table tab_p_test drop partition p010
SQL> /

Table altered.

Elapsed: 00:00:00.03
SQL> 
```
- - -

### Confirm object part 2

```
  1  select object_name		
  2	  , object_id		
  3	  , data_object_id 	
  4	  , object_type		
  5	  , created		
  6	  , last_ddl_time		
  7	  , status 		
  8    from user_objects
  9   order by
 10	    object_type
 11	  , object_name
 12	  , object_id
 13*	  , data_object_id
SQL> /

OBJECT_NAME			OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE		 CREATED	     LAST_DDL_TIME	 STATUS
------------------------------ ---------- -------------- ----------------------- ------------------- ------------------- -------
IDX1_P_TEST			    92798		 INDEX			 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST			    92793		 INDEX			 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
IDX1_P_TEST			    92802	   92802 INDEX PARTITION	 2017/08/21 12:30:15 2017/08/21 12:30:15 VALID
IDXP_P_TEST			    92797	   92797 INDEX PARTITION	 2017/08/21 12:28:26 2017/08/21 12:28:26 VALID
TAB_P_TEST			    92787		 TABLE			 2017/08/21 12:27:13 2017/08/21 15:19:58 VALID
TAB_P_TEST			    92791	   92791 TABLE PARTITION	 2017/08/21 12:27:13 2017/08/21 12:27:13 VALID

6 rows selected.

Elapsed: 00:00:00.04
SQL> 
```
- - -

### Confirm table part 2

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
PMAX			       MAXVALUE     PID 		 2017/08/21 12:56:59	      0 DISABLED

1 row selected.

Elapsed: 00:00:00.06
SQL> 
```
- - -

### Confirm index part 2

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
IDX1_P_TEST		       STATUS		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDX1_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:39
IDXP_P_TEST		       PID		    PMAX			   MAXVALUE	USABLE		  0 2017/08/21 12:57:51

3 rows selected.

Elapsed: 00:00:00.17
SQL> 
```
- - -

### Drop partition pmax

```
  1* alter table tab_p_test drop partition pmax
SQL> /
alter table tab_p_test drop partition pmax
                                      *
ERROR at line 1:
ORA-14083: cannot drop the only partition of a partitioned table


Elapsed: 00:00:00.00
SQL> 
```
**specification.**
