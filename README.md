# mysql_sys.host_summary

SELECT 
    IF((`performance_schema`.`accounts`.`HOST` IS NULL),
        'background',
        `performance_schema`.`accounts`.`HOST`) AS `host`,
    SUM(`sys`.`stmt`.`total`) AS `statements`,
    FORMAT_PICO_TIME(SUM(`sys`.`stmt`.`total_latency`)) AS `statement_latency`,
    FORMAT_PICO_TIME(IFNULL((SUM(`sys`.`stmt`.`total_latency`) / NULLIF(SUM(`sys`.`stmt`.`total`), 0)),
                    0)) AS `statement_avg_latency`,
    SUM(`sys`.`stmt`.`full_scans`) AS `table_scans`,
    SUM(`sys`.`io`.`ios`) AS `file_ios`,
    FORMAT_PICO_TIME(SUM(`sys`.`io`.`io_latency`)) AS `file_io_latency`,
    SUM(`performance_schema`.`accounts`.`CURRENT_CONNECTIONS`) AS `current_connections`,
    SUM(`performance_schema`.`accounts`.`TOTAL_CONNECTIONS`) AS `total_connections`,
    COUNT(DISTINCT `performance_schema`.`accounts`.`USER`) AS `unique_users`,
    FORMAT_BYTES(SUM(`sys`.`mem`.`current_allocated`)) AS `current_memory`,
    FORMAT_BYTES(SUM(`sys`.`mem`.`total_allocated`)) AS `total_memory_allocated`
FROM
    (((`performance_schema`.`accounts`
    JOIN `sys`.`x$host_summary_by_statement_latency` `stmt` ON ((`performance_schema`.`accounts`.`HOST` = `sys`.`stmt`.`host`)))
    JOIN `sys`.`x$host_summary_by_file_io` `io` ON ((`performance_schema`.`accounts`.`HOST` = `sys`.`io`.`host`)))
    JOIN `sys`.`x$memory_by_host_by_current_bytes` `mem` ON ((`performance_schema`.`accounts`.`HOST` = `sys`.`mem`.`host`)))
GROUP BY IF((`performance_schema`.`accounts`.`HOST` IS NULL),
    'background',
    `performance_schema`.`accounts`.`HOST`)
