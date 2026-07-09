
---
slug: first-production-hotfix
title: RMAN backup failure and logs check
authors: [VikasPadmashali123]
tags: [Rman, backup, logs]
---
###This post has details of RMAN backup status and logs


Check RMAN backup status and rman backup logs: 

STEP 1:  To find the status of the jobs:

set lines 300
 col STATUS format a22
 col hrs format 999.99
 select
 SESSION_KEY, SESSION_RECID, SESSION_STAMP,INPUT_TYPE, STATUS,
 to_char(START_TIME,'mm/dd/yy hh24:mi') start_time,
 to_char(END_TIME,'mm/dd/yy hh24:mi')   end_time,
 elapsed_seconds/3600                   hrs
 from V$RMAN_BACKUP_JOB_DETAILS
 where status='RUNNING'
 order by session_key;


STEP 2: Check the logs or output of the running RMAN jobs 
set lines 200
 set pages 1000
 select output from GV$RMAN_OUTPUT
 where session_recid = &SESSION_RECID
 and session_stamp = &SESSION_STAMP
 order by recid;

