#### INSTALAÇÃO ORACLE 12c - WINDOWS

##### ACESSOS

```ini
ANY: 309173425
SENHA: *
PROXY DO IPES: proxy.ipesaude.gov-se:8080
```

##### ENTERPRISE MANAGER EXPRESS

```ini
# Configuração do Enterprise Manager Express pelo IP
https://localhost:5500/em
exec dbms_xdb_config.sethttpsport (5500);
exec dbms_xdb_config.SetGlobalPortEnabled(TRUE);
```

##### CRIANDO O BANCO DE DADOS

```sql
# dbca para criar o banco
# Não criar os pdbs

# APAGAR CONTAINERS CASO NECESSÁRIO
DROP PLUGGABLE DATABASE orclpdb  INCLUDING DATAFILES;
DROP PLUGGABLE DATABASE PDBSEED  INCLUDING DATAFILES;

select * from dba_pdbs;
ALTER PLUGGABLE DATABASE pdbseed UNPLUG INTO 'C:\app\Administrador\virtual\oradata\orcl\pdbseed.pdb';
```

##### POS INSTALAÇÃO

```ini
# windows+R  ou direto do prompt
# C: services.msc 
# C:\app\Administrador\virtual\product\12.2.0\dbhome_1\network\admin
# tnsnames.ora, sqlnet.ora, listener
---
# Ajustar o desempenho do Win10
# C: sysdm.cpl
```

##### CONFERENCIAS POS INSTALAÇÃO

```sql
# LINUX
.  oraenv
cat /etc/oratab
ps -ef | grep smon
show pdbs (Containers)
---
# WINDOWS
set ORACLE_SID=ORCL
set ORACLE_HOME=c:\app\Administrador\virtual\product\12.2.0\dbhome_1
set ORACLE_BASE=c:\app\Administrador\virtual
---
- sc query OracleServiceORCL
- sc qc OracleServiceORCL
# Para ver o ORACLE_SID e o ORACLE_HOME -
- regedit

set oracle_sid=orcl
echo %oracle_sid%
lsnrctl status

sqlplus / as sysdba
show parameter control_files;
---
select * from v$session;
select * from dict where table_name like '%DBA%';
select * from v$database;

select * from dba_tablespaces;
show parameters control_files;

archive log list;
select * from V$LOG;
select * from V$LOGFILE;
select * from V$ARCHIVED_LOG;
select * from V$ARCHIVE_DEST;
select * from V$ARCHIVE_DEST_STATUS;
select * from V$ARCHIVE_PROCESSES;

SELECT NAME FROM V$CONTROLFILE UNION 
select FILE_NAME AS NAME from dba_data_files UNION 
select FILE_NAME AS NAME from dba_temp_files;

select sum(bytes)/Power(1024,3) Gbytes from dba_data_files;
select tablespace_name, bytes/Power(1024,3) Gbytes, file_name,autoextensible from dba_data_files order by 2 desc;

startup pfile=initSID.ora
startup pfile=C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\spfileorcl.ora
```

