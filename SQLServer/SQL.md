SELECT @@VERSION AS [SQL Server and OS Version Info]

----下面这个查询，能够获取数据库Server上有关OS、语言等更多信息：
SELECT windows_release, windows_service_pack_level,windows_sku, os_language_version 
FROM sys.dm_os_windows_info WITH (NOLOCK) OPTION (RECOMPILE);

---下面的查询会告诉你有多少逻辑处理器、处理器的超线程比率、有多少物理CPU及多大物理内存。

-- Hardware information from SQL Server 2012 (new virtual_machine_type_desc)(Cannot distinguish between HT and multi-core)  
SELECT cpu_count AS [Logical CPU Count], hyperthread_ratio AS [HyperthreadRatio],cpu_count/hyperthread_ratio AS [Physical CPU Count],  
physical_memory_kb/1024 AS [Physical Memory (MB)],affinity_type_desc, virtual_machine_type_desc, sqlserver_start_time  
FROM sys.dm_os_sys_info WITH (NOLOCK) OPTION (RECOMPILE);  
-- Gives you some good basic hardware information about your database server  


----下面的查询会读取SQL Server错误日志，以获取厂商及数据库Server的型号
EXEC xp_readerrorlog 0,1,"Manufacturer"

---下面的查询会返回安装哪些SQL Server服务及其如何配置的信息：
-- SQL Server Services information from SQL Server 2012  
SELECT servicename, startup_type_desc, status_desc,last_startup_time, service_account, is_clustered, cluster_nodename  
FROM sys.dm_server_services WITH (NOLOCK) OPTION (RECOMPILE);  
-- Gives you information about your installed SQL Server Services, whether they are clustered, and which node owns the cluster resources 


----下面的查询返回SQL Server实例有多少在运行的数据库，它们位于哪里：
SELECT DB_NAME([database_id])AS [Database Name],  
[file_id], name, physical_name, type_desc, state_desc,  
CONVERT( bigint, size/128.0/1024) AS [Total Size in GB]  
FROM sys.master_files WITH (NOLOCK)  
WHERE [database_id] > 4  
AND [database_id] <> 32767  
OR [database_id] = 2  
ORDER BY DB_NAME([database_id]) OPTION (RECOMPILE);

-- 查询数据库所有者，mdf文件所在位置，数据库兼容级别，数据库创建时间
select a.name as 'DBName',b.name as 'Owner',b.type_desc,a.filename,a.crdate,a.cmptlevel
from sys.sysdatabases a 
left join sys.server_principals b on a.sid=b.sid


-----下面的查询返回哪个数据库文件有最大的I/O延迟：
SELECT DB_NAME(fs.database_id) AS [Database Name], mf.physical_name,  
io_stall_read_ms, num_of_reads,  
CAST(io_stall_read_ms/(1.0 + num_of_reads) AS NUMERIC(10,1)) AS  
[avg_read_stall_ms],io_stall_write_ms,  
num_of_writes,CAST(io_stall_write_ms/(1.0+num_of_writes) AS NUMERIC(10,1)) AS  
[avg_write_stall_ms],  
io_stall_read_ms + io_stall_write_ms AS [io_stalls], num_of_reads + num_of_writes  
AS [total_io],  
CAST((io_stall_read_ms + io_stall_write_ms)/(1.0 + num_of_reads + num_of_writes) AS  
NUMERIC(10,1))  
AS [avg_io_stall_ms]  
FROM sys.dm_io_virtual_file_stats(null,null) AS fs  
INNER JOIN sys.master_files AS mf WITH (NOLOCK)  
ON fs.database_id = mf.database_id  
AND fs.[file_id] = mf.[file_id]  
ORDER BY avg_io_stall_ms DESC OPTION (RECOMPILE); 

----下面的查询返回占用最多内存的用户数据库：
SELECT DB_NAME(database_id) AS [Database Name],  
COUNT(*) * 8/1024.0 AS [Cached Size (MB)]  
FROM sys.dm_os_buffer_descriptors WITH (NOLOCK)  
WHERE database_id > 4 -- system databases  
AND database_id <> 32767 -- ResourceDB  
GROUP BY DB_NAME(database_id)  
ORDER BY [Cached Size (MB)] DESC OPTION (RECOMPILE); 


-----下面的查询返回使用最多处理器时间的用户数据库：
WITH DB_CPU_Stats  
AS  
(SELECT DatabaseID, DB_Name(DatabaseID) AS [DatabaseName],  
SUM(total_worker_time) AS [CPU_Time_Ms]  
FROM sys.dm_exec_query_stats AS qs  
CROSS APPLY (SELECT CONVERT(int, value) AS [DatabaseID]  
FROM sys.dm_exec_plan_attributes(qs.plan_handle)  
WHERE attribute = N'dbid') AS F_DB  
GROUP BY DatabaseID)  
SELECT ROW_NUMBER() OVER(ORDER BY [CPU_Time_Ms] DESC) AS [row_num],  
DatabaseName, [CPU_Time_Ms],  
CAST([CPU_Time_Ms] * 1.0 / SUM([CPU_Time_Ms])  
OVER() * 100.0 AS DECIMAL(5, 2)) AS [CPUPercent]  
FROM DB_CPU_Stats  
WHERE DatabaseID > 4 -- system databases  
AND DatabaseID <> 32767 -- ResourceDB  
ORDER BY row_num OPTION (RECOMPILE);  


--查询占用CPU的进程
SELECT TOP 100 s.session_id,
           r.status,
           r.cpu_time,
           r.logical_reads,
           r.reads,
           r.writes,
           r.total_elapsed_time / (1000 * 60) 'Elaps M',
           SUBSTRING(st.TEXT, (r.statement_start_offset / 2) + 1,
           ((CASE r.statement_end_offset
                WHEN -1 THEN DATALENGTH(st.TEXT)
                ELSE r.statement_end_offset
            END - r.statement_start_offset) / 2) + 1) AS statement_text,
           COALESCE(QUOTENAME(DB_NAME(st.dbid)) + N'.' + QUOTENAME(OBJECT_SCHEMA_NAME(st.objectid, st.dbid)) 
           + N'.' + QUOTENAME(OBJECT_NAME(st.objectid, st.dbid)), '') AS command_text,
           r.command,
           s.login_name,
           s.host_name,
           s.program_name,
           s.last_request_end_time,
           s.login_time,
           r.open_transaction_count,
		   st.TEXT
FROM sys.dm_exec_sessions AS s
JOIN sys.dm_exec_requests AS r ON r.session_id = s.session_id CROSS APPLY sys.Dm_exec_sql_text(r.sql_handle) AS st
WHERE r.session_id != @@SPID
ORDER BY r.cpu_time DESC

--查询进程来自哪个服务器，以及服务器IP
SELECT  [服务器名称]= CONVERT(NVARCHAR(128),SERVERPROPERTY('a.SERVERNAME')) 
,a.LOCAL_NET_ADDRESS AS '服务器IP'
,a.CLIENT_NET_ADDRESS AS '客户端IP'
,b.host_name
 FROM SYS.DM_EXEC_CONNECTIONS a,sys.dm_exec_Sessions b 
 WHERE a.SESSION_ID = b.session_id  and a.session_id = 97


----下面的查询返回实例上累积的信号（CPU）等待：信号等待是CPU相关的等待。通常信号等待在15~20%就表示CPU压力。
SELECT CAST(100.0 * SUM(signal_wait_time_ms) / SUM (wait_time_ms) AS NUMERIC(20,2))  
AS [%signal (cpu) waits],  
CAST(100.0 * SUM(wait_time_ms - signal_wait_time_ms) / SUM (wait_time_ms) AS  
NUMERIC(20,2)) AS [%resource waits]  
FROM sys.dm_os_wait_stats WITH (NOLOCK) OPTION (RECOMPILE);  


-----下面的查询返回最常连接数据库的登录信息：
SELECT login_name, COUNT(session_id) AS [session_count]  
FROM sys.dm_exec_sessions WITH (NOLOCK)  
GROUP BY login_name  
ORDER BY COUNT(session_id) DESC OPTION (RECOMPILE);  

----下面的查询返回当前的任务急pending的I/O计数信息，返回的3个值越低越好：
SELECT AVG(current_tasks_count) AS [Avg Task Count],  
AVG(runnable_tasks_count) AS [Avg Runnable Task Count],  
AVG(pending_disk_io_count) AS [Avg Pending DiskIO Count]  
FROM sys.dm_os_schedulers WITH (NOLOCK)  
WHERE scheduler_id < 255 OPTION (RECOMPILE);  



----下面的查询返回过去256分钟CPU使用的历史状况，1分钟一个间隔： 
---如果Other Process CPU Utilization持续超过5%，你就应该查看什么在使用CPU。
DECLARE @ts_now bigint = (SELECT cpu_ticks/(cpu_ticks/ms_ticks) FROM sys.dm_os_sys_info WITH (NOLOCK));  

SELECT TOP(256) SQLProcessUtilization AS [SQL Server Process CPU Utilization],  --SQL进程CPU使用率
SystemIdle AS [System Idle Process],  --系统空闲进程
100 - SystemIdle - SQLProcessUtilization  AS [Other Process CPU Utilization],  --其它程序CPU使用率
DATEADD(ms, -1 * (@ts_now - [timestamp]),GETDATE()) AS [Event Time]  
FROM (
	SELECT record.value('(./Record/@id)[1]', 'int') AS record_id,  
	record.value('(./Record/SchedulerMonitorEvent/SystemHealth/SystemIdle)[1]', 'int') AS[SystemIdle],
	record.value('(./Record/SchedulerMonitorEvent/SystemHealth/ProcessUtilization)[1]','int') AS [SQLProcessUtilization] ,[timestamp]  
	FROM (
		SELECT [timestamp], CONVERT(xml, record) AS [record]  
		FROM sys.dm_os_ring_buffers WITH (NOLOCK)  
		WHERE ring_buffer_type = N'RING_BUFFER_SCHEDULER_MONITOR' AND record LIKE N'%<SystemHealth>%'
	) AS x  
) AS y  
ORDER BY record_id DESC OPTION (RECOMPILE);  

------下面的查询返回OS级别物理内存的状况：
SELECT total_physical_memory_kb, available_physical_memory_kb,  
total_page_file_kb, available_page_file_kb,  
system_memory_state_desc  
FROM sys.dm_os_sys_memory WITH (NOLOCK) OPTION (RECOMPILE);

-----下面的查询返回SQL Server的内存使用情况：
SELECT physical_memory_in_use_kb,locked_page_allocations_kb,  
page_fault_count, memory_utilization_percentage,  
available_commit_limit_kb, process_physical_memory_low,  
process_virtual_memory_low  
FROM sys.dm_os_process_memory WITH (NOLOCK) OPTION (RECOMPILE); 

/*---查看SQL Server是否处于内存压力的一个经典做法是查看Page Life Expectancy (PLE)，PLE越高越好：
比如你的服务内存是64G，分配给Sql Server最大内存（上述Max Server Memory）是60G
那么PLE的参考值就是60/4*300=4500S，大概是75分钟，也就是说，最低限度是每75分钟
*/
SELECT cntr_value AS [Page Life Expectancy]  
FROM sys.dm_os_performance_counters WITH (NOLOCK)  
WHERE [object_name] LIKE N'%Buffer Manager%' -- Handles named instances  
AND counter_name = N'Page life expectancy' OPTION (RECOMPILE);  

---下面的查询返回Memory Grants Outstanding：
-- 内存授予默认实例的突出值 
SELECT cntr_value AS [Memory Grants Outstanding]  
FROM sys.dm_os_performance_counters WITH (NOLOCK)  
WHERE [object_name] LIKE N'%Memory Manager%' -- Handles named instances  
AND counter_name = N'Memory Grants Outstanding' OPTION (RECOMPILE);  
-- 超过零的,是一个非常强烈的内存压力的指标

----下面的查询返回Memory Grants Pending：
-- 超过零的,是一个非常强烈的内存压力的指标
SELECT cntr_value AS [Memory Grants Pending]  
FROM sys.dm_os_performance_counters WITH (NOLOCK)  
WHERE [object_name] LIKE N'%Memory Manager%' -- Handles named instances  
AND counter_name = N'Memory Grants Pending' OPTION (RECOMPILE);  

/*
Total Server Memory 是Sql Server内存管理器“已提交”内存，说白了就是已经占用了的内存
Target Server Memory是Sql Server内存管理器可用的最大内存
当Total Server Memory小于Target Server Memory的时候，Sql Server还知道系统还有可用内存，在需要内存中的时候，直接跟系统申请，
此时Total Server Memory会逐渐变大。
但是，当Total Server Memory接近于或者等于Target Server Memory的时候，Sql Server会意识到已经用完了系统的可用内存，
如果在需要内存的时候，系统已经无法继续分配新的内存，它就需要清理已用的内存空间，将新清理出来的空间给新的数据使用
*/
select counter_name, ltrim(cntr_value*1.0/1024/1024)+'G' as memoryGB 
FROM master.sys.dm_os_performance_counters  
where counter_name like '%target%server%memory%'or  counter_name like '%total%memory%'



----当你从上面3个查询中看到任何内存压力，要进一步查看整体的内存使用状况：

-- Memory Clerk Usage for instance  
-- Look for high value for CACHESTORE_SQLCP (Ad-hoc query plans)  
SELECT TOP(10) [type] AS [Memory Clerk Type],  
SUM(pages_kb) AS [SPA Mem, Kb]  
FROM sys.dm_os_memory_clerks WITH (NOLOCK)  
GROUP BY [type]  
ORDER BY SUM(pages_kb) DESC OPTION (RECOMPILE);  
-- CACHESTORE_SQLCP SQL Plans  
-- These are cached SQL statements or batches that  
-- aren't in stored procedures, functions and triggers  
--  
-- CACHESTORE_OBJCP Object Plans  
-- These are compiled plans for  
-- stored procedures, functions and triggers  
--  
-- CACHESTORE_PHDR Algebrizer Trees  
-- An algebrizer tree is the parsed SQL text  
-- that resolves the table and column names  


---如果你看到 CACHESTORE_SQLCP memory clerk使用很多内存，那么你可以确定在Procedure缓存里是否有很多只用一次且占用了大量内存的ad hoc查询计划：
-- cp.objtype：Adhoc--即席查询；Proc--存储过程；Trigger--触发器；View--视图；SysTab--系统表；UsrTab--用户表；Check--约束；Rule--规则；Prepared--预定义语句；ReplProc--复制筛选过程Default--默认值 
-- cp.cacheobjtype：Compiled Plan--执行计划；Executable Plan--可执行计划；Parse Tree--分析数；Cursor--游标；Extended stored procedure--扩展存储过程
SELECT TOP(20) cp.objtype,cp.cacheobjtype, [text] AS [QueryText], cp.size_in_bytes 占用内存量,cp.usecounts 使用次数
FROM sys.dm_exec_cached_plans AS cp WITH (NOLOCK)  
CROSS APPLY sys.dm_exec_sql_text(plan_handle)  
WHERE cp.cacheobjtype = N'Compiled Plan'  
AND cp.objtype = N'Adhoc'  
---AND cp.usecounts = 1  
ORDER BY cp.size_in_bytes DESC OPTION (RECOMPILE);  
-- Gives you the text and size of single-use ad-hoc queries that  
-- waste space in the plan cache  
-- Enabling 'optimize for ad hoc workloads' for the instance  
-- can help (SQL Server 2008 and above only)  
-- Enabling forced parameterization for the database can help, but test first! 

/*
当遇到数据库不能收缩时，先执行DBCC FREESYSTEMCACHE，再进行收缩
不过此DBCC会将计划缓存清空。
*/
DBCC FREESYSTEMCACHE ('ALL') WITH MARK_IN_USE_FOR_REMOVAL 


/*********************** 数据库级别的查询 *************************************/
USE Salesmanagement;  
GO 

--查询数据库大小：
-- Individual File Sizes and space available for current database  
SELECT name AS [File Name], physical_name AS [Physical Name], size/128.0 AS [Total Size in MB],  
size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS int)/128.0 AS [Available Space In MB], [file_id]  
FROM sys.database_files WITH (NOLOCK) OPTION (RECOMPILE);  

--查看事务日志大小及空间使用：
-- Get transaction log size and space information for the current database  
SELECT DB_NAME(database_id) AS [Database Name], database_id,  
CAST((total_log_size_in_bytes/1048576.0) AS DECIMAL(10,1)) AS [Total_log_size(MB)],  
CAST((used_log_space_in_bytes/1048576.0) AS DECIMAL(10,1)) AS [Used_log_space(MB)],  
CAST(used_log_space_in_percent AS DECIMAL(10,1)) AS [Used_log_space(%)]  
FROM sys.dm_db_log_space_usage WITH (NOLOCK) OPTION (RECOMPILE); 

---按文件搜集I/O统计：
-- I/O Statistics by file for the current database  
SELECT DB_NAME(DB_ID()) AS [Database Name],[file_id], num_of_reads, num_of_writes,  
io_stall_read_ms, io_stall_write_ms,  
CAST(100. * io_stall_read_ms/(io_stall_read_ms + io_stall_write_ms)  
AS DECIMAL(10,1)) AS [IO Stall Reads Pct],  
CAST(100. * io_stall_write_ms/(io_stall_write_ms + io_stall_read_ms)  
AS DECIMAL(10,1)) AS [IO Stall Writes Pct],  
(num_of_reads + num_of_writes) AS [Writes + Reads], num_of_bytes_read,  
num_of_bytes_written,  
CAST(100. * num_of_reads/(num_of_reads + num_of_writes) AS DECIMAL(10,1))  
AS [# Reads Pct],  
CAST(100. * num_of_writes/(num_of_reads + num_of_writes) AS DECIMAL(10,1))  
AS [# Write Pct],  
CAST(100. * num_of_bytes_read/(num_of_bytes_read + num_of_bytes_written)  
AS DECIMAL(10,1)) AS [Read Bytes Pct],  
CAST(100. * num_of_bytes_written/(num_of_bytes_read + num_of_bytes_written)  
AS DECIMAL(10,1)) AS [Written Bytes Pct]  
FROM sys.dm_io_virtual_file_stats(DB_ID(), NULL) OPTION (RECOMPILE);

---查看事务日志Virtual Log File (VLF)计数：
-- Get VLF count for transaction log for the current database,  
-- number of rows equals the VLF count. Lower is better!  
DBCC LOGINFO;  
-- High VLF counts can affect write performance  
-- and they can make database restore and recovery take much longer  

----事务日志中如果有大量VLF，就会影响写入事务日志的性能，更重要的是影响恢复数据库的时间。
--查看特定数据库上的查询活动：
-- Top cached queries by Execution Count (SQL Server 2012)  
SELECT qs.execution_count, qs.total_rows, qs.last_rows, qs.min_rows, qs.max_rows,  
qs.last_elapsed_time, qs.min_elapsed_time, qs.max_elapsed_time,  
SUBSTRING(qt.TEXT,qs.statement_start_offset/2 +1,  
(CASE WHEN qs.statement_end_offset = -1  
THEN LEN(CONVERT(NVARCHAR(MAX), qt.TEXT)) * 2  
ELSE qs.statement_end_offset END - qs.statement_start_offset)/2)  
AS query_text  
FROM sys.dm_exec_query_stats AS qs WITH (NOLOCK)  
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS qt  
ORDER BY qs.execution_count DESC OPTION (RECOMPILE);  


-----查看缓存的存储过程：
-- Top Cached SPs By Execution Count (SQL Server 2012)  
SELECT TOP(250) p.name AS [SP Name], qs.execution_count,  
ISNULL(qs.execution_count/DATEDIFF(Second, qs.cached_time, GETDATE()), 0)  
AS [Calls/Second],  
qs.total_worker_time/qs.execution_count AS [AvgWorkerTime],  
qs.total_worker_time AS [TotalWorkerTime],qs.total_elapsed_time,  
qs.total_elapsed_time/qs.execution_count AS [avg_elapsed_time],  
qs.cached_time  
FROM sys.[procedures] AS p WITH (NOLOCK)  
INNER JOIN sys.[dm_exec_procedure_stats] AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
ORDER BY qs.execution_count DESC OPTION (RECOMPILE);  


-----查看存储过程在缓存里待多长时间（Cached_time）：
-- Top Cached SPs By Avg Elapsed Time (SQL Server 2012)  
SELECT TOP(25) p.name AS [SP Name], qs.total_elapsed_time/qs.execution_count  
AS [avg_elapsed_time], qs.total_elapsed_time, qs.execution_count,  
ISNULL(qs.execution_count/DATEDIFF(Second, qs.cached_time,  
GETDATE()), 0) AS [Calls/Second],  
qs.total_worker_time/qs.execution_count AS [AvgWorkerTime],  
qs.total_worker_time AS [TotalWorkerTime], qs.cached_time  
FROM sys.[procedures] AS p WITH (NOLOCK)  
INNER JOIN sys.[dm_exec_procedure_stats] AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
ORDER BY avg_elapsed_time DESC OPTION (RECOMPILE);


----从整体CPU角度查看最耗时的存储过程：
/*字段解释：total_worker_time -- 此存储过程自编译以来执行所用的 CPU 时间总量（微秒）
            execution_count -- 存储过程自上次编译以来所执行的次数。
			cached_time -- 存储过程添加到缓存的时间
			total_elapsed_time -- 完成此存储过程的执行所用的总时间（微秒）   */
SELECT TOP(25) p.name AS [SP Name], qs.total_worker_time AS '占用CPU总时长',  
qs.execution_count as '执行次数', qs.total_worker_time/qs.execution_count AS '平均CPU时长'
,last_worker_time as '上次CPU时长', 
ISNULL(qs.execution_count/DATEDIFF(Second, qs.cached_time, GETDATE()), 0)  AS 'Calls/Second'
,last_execution_time as '上次运行时间'
,qs.total_elapsed_time as '执行总时长', qs.total_elapsed_time/qs.execution_count  AS '平均执行时长'
,last_elapsed_time as '上次执行时长'
, qs.cached_time as '添加到缓存的时间' 
FROM sys.[procedures] AS p WITH (NOLOCK)  
INNER JOIN sys.[dm_exec_procedure_stats] AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
ORDER BY qs.total_worker_time DESC OPTION (RECOMPILE); 
-- This helps you find the most expensive cached  
-- stored procedures from a CPU perspective  
-- You should look at this if you see signs of CPU pressure 

---从逻辑度的角度查看缓存的存储过程相关的信息： 
-- Logical reads relate to memory pressure  
SELECT TOP(25) p.name AS [SP Name], qs.total_logical_reads  
AS [TotalLogicalReads], qs.total_logical_reads/qs.execution_count  
AS [AvgLogicalReads],qs.execution_count,  
ISNULL(qs.execution_count/DATEDIFF(Second, qs.cached_time, GETDATE()), 0)  
AS [Calls/Second], qs.total_elapsed_time,qs.total_elapsed_time/qs.execution_count  
AS [avg_elapsed_time], qs.cached_time  
FROM sys.procedures AS p WITH (NOLOCK)  
INNER JOIN sys.dm_exec_procedure_stats AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
ORDER BY qs.total_logical_reads DESC OPTION (RECOMPILE);  
-- This helps you find the most expensive cached  
-- stored procedures from a memory perspective  
-- You should look at this if you see signs of memory pressure  



---从物理度的角度查看最耗时的存储过程：
-- Physical reads relate to disk I/O pressure  
SELECT TOP(25) p.name AS [SP Name],qs.total_physical_reads  
AS [TotalPhysicalReads],qs.total_physical_reads/qs.execution_count  
AS [AvgPhysicalReads], qs.execution_count, qs.total_logical_reads,  
qs.total_elapsed_time, qs.total_elapsed_time/qs.execution_count  
AS [avg_elapsed_time], qs.cached_time  
FROM sys.procedures AS p WITH (NOLOCK)  
INNER JOIN sys.dm_exec_procedure_stats AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
AND qs.total_physical_reads > 0  
ORDER BY qs.total_physical_reads DESC,  
qs.total_logical_reads DESC OPTION (RECOMPILE);  
-- This helps you find the most expensive cached  
-- stored procedures from a read I/O perspective  
-- You should look at this if you see signs of I/O pressure or of memory pressure  


----从逻辑写来看最耗时的缓存存储过程： 
-- Logical writes relate to both memory and disk I/O pressure  
SELECT TOP(25) p.name AS [SP Name], qs.total_logical_writes  
AS [TotalLogicalWrites], qs.total_logical_writes/qs.execution_count  
AS [AvgLogicalWrites], qs.execution_count,  
ISNULL(qs.execution_count/DATEDIFF(Second, qs.cached_time, GETDATE()), 0)  
AS [Calls/Second],qs.total_elapsed_time, qs.total_elapsed_time/qs.execution_count  
AS [avg_elapsed_time], qs.cached_time  
FROM sys.procedures AS p WITH (NOLOCK)  
INNER JOIN sys.dm_exec_procedure_stats AS qs WITH (NOLOCK)  
ON p.[object_id] = qs.[object_id]  
WHERE qs.database_id = DB_ID()  
ORDER BY qs.total_logical_writes DESC OPTION (RECOMPILE);  
-- This helps you find the most expensive cached  
-- stored procedures from a write I/O perspective  
-- You should look at this if you see signs of I/O pressure or of memory pressure


----从平均I/O来看缓存的存储过程中最耗资源的语句：
-- usage for the current database  
SELECT TOP(50) OBJECT_NAME(qt.objectid) AS [SP Name],  
(qs.total_logical_reads + qs.total_logical_writes) /qs.execution_count  
AS [Avg IO],SUBSTRING(qt.[text],qs.statement_start_offset/2,  
(CASE  
WHEN qs.statement_end_offset = -1  
THEN LEN(CONVERT(nvarchar(max), qt.[text])) * 2  
ELSE qs.statement_end_offset  
END - qs.statement_start_offset)/2) AS [Query Text]  
FROM sys.dm_exec_query_stats AS qs WITH (NOLOCK)  
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS qt  
WHERE qt.[dbid] = DB_ID()  
ORDER BY [Avg IO] DESC OPTION (RECOMPILE);  
-- Helps you find the most expensive statements for I/O by SP 

-----查看写比读更多的非聚集索引：
-- Possible Bad NC Indexes (writes > reads)  
SELECT OBJECT_NAME(s.[object_id]) AS [Table Name], i.name AS [Index Name],  
i.index_id,user_updates AS [Total Writes],  
user_seeks + user_scans + user_lookups AS [Total Reads],  
user_updates - (user_seeks + user_scans + user_lookups) AS [Difference]  
FROM sys.dm_db_index_usage_stats AS s WITH (NOLOCK)  
INNER JOIN sys.indexes AS i WITH (NOLOCK)  
ON s.[object_id] = i.[object_id]  
AND i.index_id = s.index_id  
WHERE OBJECTPROPERTY(s.[object_id],'IsUserTable') = 1  
AND s.database_id = DB_ID()  
AND user_updates > (user_seeks + user_scans + user_lookups)  
AND i.index_id > 1  
ORDER BY [Difference] DESC, [Total Writes] DESC, [Total Reads] ASC OPTION  (RECOMPILE);  
-- Look for indexes with high numbers of writes  
-- and zero or very low numbers of reads  
-- Consider your complete workload  
-- Investigate further before dropping an index!  


----查看缺失的索引信息：
-- Missing Indexes current database by Index Advantage  
SELECT user_seeks * avg_total_user_cost * (avg_user_impact * 0.01) AS [index_advantage],  
migs.last_user_seek, mid.[statement] AS [Database.Schema.Table],  
mid.equality_columns, mid.inequality_columns, mid.included_columns,  
migs.unique_compiles, migs.user_seeks, migs.avg_total_user_cost,  
migs.avg_user_impact  
FROM sys.dm_db_missing_index_group_stats AS migs WITH (NOLOCK)  
INNER JOIN sys.dm_db_missing_index_groups AS mig WITH (NOLOCK)  
ON migs.group_handle = mig.index_group_handle  
INNER JOIN sys.dm_db_missing_index_details AS mid WITH (NOLOCK)  
ON mig.index_handle = mid.index_handle  
WHERE mid.database_id = DB_ID() -- Remove this to see for entire instance  
ORDER BY index_advantage DESC OPTION (RECOMPILE);  
-- Look at last user seek time, number of user seeks  
-- to help determine source and importance  
-- SQL Server is overly eager to add included columns, so beware  
-- Do not just blindly add indexes that show up from this query!!!

-- 查看索引缺失，（有拼接创建索引的语句）
SELECT MID.[statement] AS ObjectName
      ,MID.equality_columns AS EqualityColumns
      ,MID.inequality_columns AS InequalityColms
      ,MID.included_columns AS IncludedColumns
      ,MIGS.last_user_seek AS LastUserSeek
      ,MIGS.avg_total_user_cost 
       * MIGS.avg_user_impact 
       * (MIGS.user_seeks + MIGS.user_scans) AS Impact
      ,N'CREATE NONCLUSTERED INDEX <Add Index Name here> ' + 
       N'ON ' + MID.[statement] + 
       N' (' + MID.equality_columns 
             + ISNULL(', ' + MID.inequality_columns, N'') +
       N') ' + ISNULL(N'INCLUDE (' + MID.included_columns + N');', ';')
       AS CreateStatement 
FROM sys.dm_db_missing_index_group_stats AS MIGS
INNER JOIN sys.dm_db_missing_index_groups AS MIG ON MIGS.group_handle = MIG.index_group_handle
INNER JOIN sys.dm_db_missing_index_details AS MID ON MIG.index_handle = MID.index_handle
WHERE database_id = DB_ID()
AND MIGS.last_user_seek >= DATEDIFF(month, GetDate(), -1)
ORDER BY Impact DESC;



----查看缓存的执行计划中缺失索引警告：
-- Find missing index warnings for cached plans in the current database  
-- Note: This query could take some time on a busy instance  
SELECT TOP(25) OBJECT_NAME(objectid) AS [ObjectName],query_plan,  
cp.objtype, cp.usecounts  
FROM sys.dm_exec_cached_plans AS cp WITH (NOLOCK)  
CROSS APPLY sys.dm_exec_query_plan(cp.plan_handle) AS qp  
WHERE CAST(query_plan AS NVARCHAR(MAX)) LIKE N'%MissingIndex%'  
AND dbid = DB_ID()  
ORDER BY cp.usecounts DESC OPTION (RECOMPILE);  
-- Helps you connect missing indexes to specific stored procedures or queries  
-- This can help you decide whether to add them or not  

--- 查看SQL Server缓冲池中占用最多空间的表和索引：
-- Breaks down buffers used by current database  
-- by object (table, index) in the buffer cache  
SELECT OBJECT_NAME(p.[object_id]) AS [ObjectName],  
p.index_id, COUNT(*)/128 AS [Buffer size(MB)], COUNT(*) AS [BufferCount],  
p.data_compression_desc AS [CompressionType]  
FROM sys.[allocation_units] AS a WITH (NOLOCK)  
INNER JOIN sys.[dm_os_buffer_descriptors] AS b WITH (NOLOCK)  
ON a.allocation_unit_id = b.allocation_unit_id  
INNER JOIN sys.[partitions] AS p WITH (NOLOCK)  
ON a.container_id = p.hobt_id  
WHERE b.database_id = CONVERT(int,DB_ID())  
AND p.[object_id] > 100  
GROUP BY p.[object_id], p.index_id, p.data_compression_desc  
ORDER BY [BufferCount] DESC OPTION (RECOMPILE);  
-- Tells you what tables and indexes are  
-- using the most memory in the buffer cache  



----查看数据库中所有表的大小及数据压缩状态：
-- Get Table names, row counts, and compression status  
-- for the clustered index or heap  
SELECT OBJECT_NAME(object_id) AS [ObjectName],  
SUM(Rows) AS [RowCount], data_compression_desc AS [CompressionType]  
FROM sys.partitions WITH (NOLOCK)  
WHERE index_id < 2 --ignore the partitions from the non-clustered index if any  
AND OBJECT_NAME(object_id) NOT LIKE N'sys%'  
AND OBJECT_NAME(object_id) NOT LIKE N'queue_%'  
AND OBJECT_NAME(object_id) NOT LIKE N'filestream_tombstone%'  
AND OBJECT_NAME(object_id) NOT LIKE N'fulltext%'  
AND OBJECT_NAME(object_id) NOT LIKE N'ifts_comp_fragment%'  
AND OBJECT_NAME(object_id) NOT LIKE N'filetable_updates%'  
GROUP BY object_id, data_compression_desc  
ORDER BY SUM(Rows) DESC OPTION (RECOMPILE);  
-- Gives you an idea of table sizes, and possible data compression opportunities  


---查看数据库中所有索引最后一次统计更新的时间：
-- When were Statistics last updated on all indexes?  
SELECT o.name, i.name AS [Index Name],STATS_DATE(i.[object_id],  
i.index_id) AS [Statistics Date], s.auto_created,  
s.no_recompute, s.user_created, st.row_count  
FROM sys.objects AS o WITH (NOLOCK)  
INNER JOIN sys.indexes AS i WITH (NOLOCK)  
ON o.[object_id] = i.[object_id]  
INNER JOIN sys.stats AS s WITH (NOLOCK)  
ON i.[object_id] = s.[object_id]  
AND i.index_id = s.stats_id  
INNER JOIN sys.dm_db_partition_stats AS st WITH (NOLOCK)  
ON o.[object_id] = st.[object_id]  
AND i.[index_id] = st.[index_id]  
WHERE o.[type] = 'U'  
ORDER BY STATS_DATE(i.[object_id], i.index_id) ASC OPTION (RECOMPILE);  
-- Helps discover possible problems with out-of-date statistics  
-- Also gives you an idea which indexes are most active  


----查看当前数据库中碎片最多的索引：
-- Get fragmentation info for all indexes  
-- above a certain size in the current database  
-- Note: This could take some time on a very large database  
SELECT DB_NAME(database_id) AS [Database Name],  
OBJECT_NAME(ps.OBJECT_ID) AS [Object Name],  
i.name AS [Index Name], ps.index_id, index_type_desc,  
avg_fragmentation_in_percent, fragment_count, page_count  
FROM sys.dm_db_index_physical_stats(DB_ID(),NULL, NULL, NULL ,'LIMITED') AS ps  
INNER JOIN sys.indexes AS i WITH (NOLOCK)  
ON ps.[object_id] = i.[object_id]  
AND ps.index_id = i.index_id  
WHERE database_id = DB_ID()  
AND page_count > 500  
ORDER BY avg_fragmentation_in_percent DESC OPTION (RECOMPILE);  
-- Helps determine whether you have fragmentation in your relational indexes  
-- and how effective your index maintenance strategy is  


/*
如果你发现超过10%碎片的索引，你就需要决定是重组还是重建它们。重组通常是Online操作，能在任何时间停止。
重建可以是Online或Offline操作。在SQL Server 2012中，你可以Online重建聚集索引，而不必考虑表包含什么类型的数据。
收缩数据文件非常消耗资源。不要犯常见的错误定期去重建所有索引，这非常浪费资源。
可以在Internet上找一些好的索引维护脚本。Ola Hallengren开发了一个非常好的脚本，可以从 http://ola.hallengren.com  获取。
*/


-- 数据库提示正在还原中的处理办法
RESTORE DATABASE [ReportServer$SKYPEDB] WITH RECOVERY