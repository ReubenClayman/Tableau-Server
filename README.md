# PARSEING data from Tableau Server Log Files from data stores in JSON formats 
The query is designed to:

Parse complex JSON log data into a structured tabular format for analysis.

Focus specifically on logs related to an "end-bootstrap-session" action.

Extract fields that are likely important for performance analysis, user behavior tracking, or debugging, such as:
- Process ID
- User and session identifiers
- Workbook and sheet identifiers
- Timing and resource utilization metrics (elapsed, ucpu_e, ucpu_i).

 ```sql
  
  SELECT 
  PARSE_JSON(DATA):pid::int AS process_id, 
  PARSE_JSON(DATA):ts::string AS event_timestamp, 
  PARSE_JSON(DATA):user::string AS user, 
  PARSE_JSON(DATA):sess::string AS server_vizql_session,
  PARSE_JSON(PARSE_JSON(DATA):ctx):wb::string AS workbook,
  PARSE_JSON(PARSE_JSON(DATA):ctx):vw::string AS sheet,
  PARSE_JSON(PARSE_JSON(DATA):a):elapsed::int AS elapsed,
  PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:e::string AS ucpu_e,
  PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:i::string AS ucpu_i,
  AVG(PARSE_JSON(PARSE_JSON(DATA):a):elapsed::int) OVER (PARTITION BY PARSE_JSON(PARSE_JSON(DATA):ctx):wb::string) AS avg_elapsed_time,
  COUNT(*) OVER (PARTITION BY PARSE_JSON(PARSE_JSON(DATA):ctx):wb::string) AS event_count_per_workbook,

 AVG(CASE 
    WHEN PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:e::string IS NOT NULL 
    THEN CAST(PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:e::string AS FLOAT) 
    ELSE 0 
  END) OVER (PARTITION BY PARSE_JSON(PARSE_JSON(DATA):ctx):wb::string) AS avg_cpu_e_usage,

 AVG(CASE 
    WHEN PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:i::string IS NOT NULL 
    THEN CAST(PARSE_JSON(PARSE_JSON(PARSE_JSON(DATA):a):res):ucpu:i::string AS FLOAT) 
    ELSE 0 
  END) OVER (PARTITION BY PARSE_JSON(PARSE_JSON(DATA):ctx):wb::string) AS avg_cpu_i_usage

  FROM 
  LOGS_RAW

WHERE 
  PARSE_JSON(DATA):user::string <> '-' 
  AND PARSE_JSON(DATA):k::string LIKE 'end-bootstrap-session-action.bootstrap-session'

LIMIT 100;
```
  
This approach enables analysts to focus on a subset of the logs for further investigation or reporting.
