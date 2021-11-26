#### REQUISITOS E BOAS PRÁTICAS DE BACKUP COM RMAN

##### MULTIPLEXAR OS CONTROLFILES

```sql
# Coloca-los em discos diferentes
sqlplus / as sysdba
select * from v$controlfile;

startup nomount;
alter system set control_files='c:\app\administrador\virtual\oradata\orcl\control01.ctl',
 'c:\app\Administrador\virtual\oradataX\controlfiles\control02.ctl',
 'c:\app\Administrador\virtual\oradataX\controlfiles\control03.ctl' scope=spfile;
shutdown immediate;

# Copiar os controlfiles para as pastas definidas
# Fazer a copia do control03.ctl a partir do control02.ctl
startup;
---
show parameter dump

# Verificar o alert.log
C:\app\Administrador\virtual\diag\rdbms\orcl\orcl\trace 
alert_orcl
```

##### HABILITANDO A FAST RECOVERY AREA

```sql
# PRINCIPAIS VARIÁVEIS DE AMBIENTE
SET ORACLE_SID=PRODUCAO
SET ORACLE_BASE=C:\APP\ADMINISTRATOR\VIRTUAL
SET ORACLE_HOME=C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\12.2.0\dbhome_1

show parameter db_recover;
select * from v$flash_recovery_area_usage;
select * from v$recovery_file_dest;

archive log list;
select archiver from v$instance;
alter system switch logfile;

# Criar a pasta FRA
alter system set db_recovery_file_dest_size = 200G scope=both;
alter system set db_recovery_file_dest = '%ORACLE_BASE%\FRA' scope=both;
# alter system set db_recovery_file_dest_size = 200G scope=both SID='*';

# SCRIPT PARA CALCULAR O TAMANHO DA FRA
DECLARE
v_backup_hold NUMBER;
v_size_fra NUMBER;
v_growth_rate NUMBER;
v_size_db NUMBER;
v_size_archive NUMBER;
BEGIN
v_backup_hold := 2;
v_growth_rate := 1.2;
v_size_fra := 0;
-- Tamanho atual do DB em GB
SELECT ROUND(SUM(bytes)/1024/1024/1024,2) INTO v_size_db FROM v$datafile;
-- Tamanho de 24 horas de archivelog em GB
SELECT ROUND(SUM(blocks*block_size)/1024/1024/1024,2) INTO v_size_archive 
FROM V$ARCHIVED_LOG WHERE completion_time > sysdate-1;
-- Tamanho da FRA
SELECT ROUND(((v_size_db*v_backup_hold)+v_size_archive)*v_growth_rate,2) 
INTO v_size_fra FROM dual;
SYS.DBMS_OUTPUT.PUT_LINE('');
SYS.DBMS_OUTPUT.PUT_LINE('Size Database (GB): '||v_size_db);
SYS.DBMS_OUTPUT.PUT_LINE('Size Archives 24h (GB): '||v_size_archive);
SYS.DBMS_OUTPUT.PUT_LINE('Number backups level 0 hold: '||v_backup_hold||' Backups.');
SYS.DBMS_OUTPUT.PUT_LINE('Growth rate (%): '||v_growth_rate);
SYS.DBMS_OUTPUT.PUT_LINE('');
SYS.DBMS_OUTPUT.PUT_LINE('Size FRA = ((DB Size * Number backups level 0 hold) + Archivelog Size 24h) * Growth rate');
SYS.DBMS_OUTPUT.PUT_LINE('');
SYS.DBMS_OUTPUT.PUT_LINE('Size FRA (GB): '||v_size_fra);
END;
/
```

##### HBILITANDO OS ARCHIVELOGS

```sql
set ORACLE_BASE='c:\app\adminstrator\virtual'
sqlplus / as sysdba
archive log list;
select * from V$ARCHIVED_LOG;

#DESTINO DO ARCHIVELOG
alter system set log_archive_dest_1='LOCATION=%ORACLE_BASE%\oradataX\archivelogs' scope=spfile;

# ARCHIVELOG NA FRA
alter system set log_archive_dest_1='LOCATION=USE_DB_RECOVERY_FILE_DEST' scope=both;

show parameter log_archive_format;
alter system set log_archive_format='';
alter system set log_archive_format = '%t_%s_%r.arc' scope=spfile;
alter system set archive_lag_target = 600 SCOPE=BOTH; --600seg
---
# HABILITANDO/DESABILITANDO MODO ARCHIVE
shutdown immediate;
startup mount;
# startup mount exclusive;
alter database archivelog; 
# alter database noarchivelog;
alter database open;
archive log list;

alter system archive log current;
select * from v$log l join v$logfile lf on l.group# = lf.group#;
select log_mode from v$database;
select open_mode from v$database;
select archiver from v$instance;
---
crosscheck archivelog all;
list expired archivelog all;
delete expired archivelog all;
delete noprompt expired archivelog all;
delete archivelog all;
```

