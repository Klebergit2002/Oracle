##### MULTIPLEXANDO ARCHIVELOGS 

```sql
# VER NUMERO DE PROCESSO ARCn
ps -ef | grep arc
No oracle voce pode checar a view 
select * fv$archive_processes
O parametro é LOG_ARCHIVE_MAX_PROCESSES


LOG_ARCHIVE_DEST_1 = /local1 MANDATORY REOPEN = 300
LOG_ARCHIVE_DEST_2 = /local2 MANDATORY REOPEN = 300
log_archive_min_succeed_dest = 2
LOG_ARCHIVE_MAX_PROCESSES = 4
```

##### SETANDO ALGUNS PARAMETROS PARA OS ARCHIVELOGS 

```sql
# LOGMINER - Verificação de archivelogs
# DBMS_LOGMNR
select count (*),inst_id,PARSING_SCHEMA_NAME
from gv$sql where COMMAND_TYPE in (2,6,7)
group by inst_id,PARSING_SCHEMA_NAME
order by 1 ,3,2

SELECT * FROM DBA_TAB_MODIFICATIONS WHERE table_owner = 'IPESAUDE' and  to_date(timestamp) = to_date('18/08/2021','DD/MM/YYYY') and inserts <> 0 and deletes <> 0;

# ARCHIVES POR HORA
alter session set nls_date_format='dd/mm/yyyy hh24:mi:ss';

select trunc(COMPLETION_TIME,'HH') Hour,thread# , round(sum(BLOCKS*BLOCK_SIZE)/1048576) MB,count(*) Archives 
from v$archived_log   
group by trunc(COMPLETION_TIME,'HH'),thread#  
order by 1;

# VERIFICANDO O CAMINHO
show parameters log_archive_dest;

# SETANDO O CAMINHO
alter system set log_archive_dest_1='LOCATION=c:\app\Administrador\virtual\oradataX\archivelogs' scope=spfile;

# COLOCANDO OS ARCHIVES NA FRA
alter system set log_archive_dest_1='LOCATION=USE_DB_RECOVERY_FILE_DEST' scope=both;

# CONFIGURANDO O FORMATO DO NOME DO ARCHIVE
show parameter log_archive_format;
alter system set log_archive_format='';
alter system set log_archive_format = '%t_%s_%r.arc' scope=spfile;

# SETANDO O TEMPO DE ARCHIVE
alter system set archive_lag_target = 600 SCOPE=BOTH; --600seg

# LIMPANDO OS LOGS
crosscheck archivelog all;
list expired archivelog all;

delete expired archivelog all;
delete noprompt expired archivelog all;
delete archivelog all;
```

##### ATIVANDO/DESATIVANDO O MODO ARCHIVE

```sql
# VERIFICANDO O MODO ARCHIVE
sqlplus / as sysdba
archive log list;
select * from V$ARCHIVED_LOG;

# ATIVANDO MODO ARCHIVE
shutdown immediate;
startup mount;
# startup mount exclusive;
alter database archivelog; 
alter database open;
archive log list;

# DESATIVANDO MODO ARCHIVE
shutdown immediate;
startup mount;
# startup mount exclusive;
alter database noarchivelog;
alter database open;
archive log list;

# CONFERINDO DEPOIS DO PROCEDIMENTO 
alter system archive log current;
select * from v$log l join v$logfile lf on l.group# = lf.group#;
select log_mode from v$database;
select open_mode from v$database;
select archiver from v$instance;
```

