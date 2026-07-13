Title: AWR AND ASH commands! 

ASH Top sessions: 

SELECT session_id, session_serial#, COUNT(*) samples, MAX(sql_id) last_sql
FROM v$active_session_history
WHERE sample_time > SYSTIMESTAMP - INTERVAL '15' MINUTE
GROUP BY session_id, session_serial#
ORDER BY samples DESC
FETCH FIRST 20 ROWS ONLY;

ASG Tio sql 5mins: 

SELECT ash.sql_id,
       COUNT(*) samples,
       SUM(CASE WHEN ash.event IS NULL THEN 1 ELSE 0 END) cpu_samples
FROM v$active_session_history ash
WHERE sample_time > SYSTIMESTAMP - INTERVAL '5' MINUTE
GROUP BY ash.sql_id
ORDER BY samples DESC
FETCH FIRST 20 ROWS ONLY;

ASH Top waits last 10 mins: 

-- top wait events in last 10 minutes (samples)
SELECT event,
       COUNT(*) samples,
       ROUND(100 * COUNT(*) / (SELECT COUNT(*) FROM v$active_session_history WHERE sample_time > SYSTIMESTAMP - INTERVAL '10' MINUTE),2) pct
FROM v$active_session_history
WHERE sample_time > SYSTIMESTAMP - INTERVAL '10' MINUTE
GROUP BY event
ORDER BY samples DESC;

AWR Create snapshot:

-- create an immediate AWR snapshot (run as SYS)
EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT;


AWR generate report via api:

-- generate AWR HTML for specific snapshot ids (returns CLOB)
SET LONG 100000000
SET PAGESIZE 0
SELECT DBMS_WORKLOAD_REPOSITORY.AWR_REPORT_HTML(
  (SELECT DBID FROM V$DATABASE),
  (SELECT INSTANCE_NUMBER FROM V$INSTANCE),
  :begin_snap_id,
  :end_snap_id
) FROM DUAL;
-- Bind :begin_snap_id and :end_snap_id or replace with literal numbers

List AWR snapshots:

SELECT snap_id, begin_interval_time, end_interval_time
FROM dba_hist_snapshot
ORDER BY snap_id DESC
FETCH FIRST 50 ROWS ONLY;

Top SQL from AWR range:

-- top SQL by elapsed time between AWR snapshots (example)
SELECT s.sql_id,
       t.sql_text,
       SUM(s.elapsed_time_delta) total_elapsed,
       SUM(s.cpu_time_delta) total_cpu,
       SUM(s.buffer_gets_delta) total_buffer_gets,
       SUM(s.executions_delta) total_executions
FROM dba_hist_sqlstat s
JOIN dba_hist_sqltext t ON s.sql_id = t.sql_id
WHERE s.snap_id BETWEEN :begin_snap AND :end_snap
  AND s.dbid = (SELECT DBID FROM V$DATABASE)
GROUP BY s.sql_id, t.sql_text
ORDER BY total_elapsed DESC
FETCH FIRST 20 ROWS ONLY;
