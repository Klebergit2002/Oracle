##### valCRIANDO PONTOS DE RESTAURAÇÃO

```sql
SQL> create restore point teste;
RMAN> backup incremental level 0 database;
SQL> shutdown immediate;
SQL> startup mount;
RMAN> restore database until restore point teste;
RMAN> recover database until restore point teste;
RMAN> alter database open resetlogs;
```

##### SCRIPTS DE BACKUP - ATUAIS

##### BACKUP DIARIO

```sql
@ECHO OFF
CLS
SET NLS_DATE_FORMAT=DD-MON-YYYY HH24:MI:SS
SET ORACLE_SCRIPT=C:\app\administrator\virtual\scripts
SET ORACLE_LOG=%ORACLE_SCRIPT%\log_backup
SET ORACLE_HOME=C:\app\Administrator\virtual\product\12.2.0\dbhome_1
SET ORACLE_SID=PRODUCAO

REM --------------------------------------
REM PEGA DATA E HORA NO FORMATO ADEQUADO
REM --------------------------------------
for /f "delims=" %%a in ('wmic OS Get localdatetime  ^| find "."') do set dt=%%a
set YYYY=%dt:~0,4%
set MM=%dt:~4,2%
set DD=%dt:~6,2%
set HH=%dt:~8,2%
set Min=%dt:~10,2%
set Sec=%dt:~12,2%
set DATA_HORA=%YYYY%-%MM%-%DD%_%HH%-%Min%-%Sec%

cd %ORACLE_SCRIPT%
%ORACLE_HOME%\bin\rman target / 
cmdfile=%ORACLE_SCRIPT%\backup_diario_homo_ag.sql msglog=%ORACLE_LOG%\backup_diario_homo_ag_%DATA_HORA%.log

@ECHO ON
---
set echo on

CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

crosscheck backup;
crosscheck archivelog all;
delete noprompt expired backup;
delete noprompt expired archivelog all; 


run {
  sql 'alter system checkpoint';
  sql 'alter system switch logfile';
  sql 'alter system archive log current';

  backup incremental level 1 cumulative as compressed backupset database tag 'database-1';
  backup as compressed backupset archivelog all tag 'archives-1';
  backup as compressed backupset current controlfile tag 'controls-1';
}
exit;

# EXEMPLO DE SCRIPT
# Remove os backups realizados ha mais de 15 dias
# crosscheck  backup completed between 'sysdate-90' AND 'sysdate-6';
# crosscheck archivelog until time 'sysdate-6';
# delete noprompt archivelog until time 'sysdate-6';
# delete noprompt backup completed between 'sysdate-90' AND 'sysdate-6';
# delete noprompt expired backup;
# delete noprompt expired archivelog all;
 
# CONFIGURAÇÕES USADAS NO BACKUP
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

# ESTUDAR AS 2 CONFIGURAÇÕES ABAIXO:
CONFIGURE RMAN OUTPUT TO KEEP FOR 7 DAYS; # default
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default

# COMPLEMENTOS
run {
   crosscheck backup;
   crosscheck archivelog all;
   delete noprompt expired backup;
   delete noprompt expired archivelog all; 
   delete noprompt obsolete;
}

report obsolete;   
delete obsolete;

```

##### BACKUP SEMANAL

```sql
@ECHO OFF
CLS
SET NLS_DATE_FORMAT=DD-MON-YYYY HH24:MI:SS
SET ORACLE_SCRIPT=C:\app\administrator\virtual\scripts
SET ORACLE_LOG=%ORACLE_SCRIPT%\log_backup
SET ORACLE_HOME=C:\app\Administrador\virtual\product\12.2.0\dbhome_1
SET ORACLE_SID=PRODUCAO

REM --------------------------------------
REM PEGA DATA E HORA NO FORMATO ADEQUADO
REM --------------------------------------
for /f "delims=" %%a in ('wmic OS Get localdatetime  ^| find "."') do set dt=%%a
set YYYY=%dt:~0,4%
set MM=%dt:~4,2%
set DD=%dt:~6,2%
set HH=%dt:~8,2%
set Min=%dt:~10,2%
set Sec=%dt:~12,2%
set DATA_HORA=%YYYY%-%MM%-%DD%_%HH%-%Min%-%Sec%

cd %ORACLE_SCRIPT%
%ORACLE_HOME%\bin\rman target / 
cmdfile=%ORACLE_SCRIPT%\backup_full_homo_ag.sql msglog=%ORACLE_LOG%\backup_full_homo_ag_%DATA_HORA%.log

@ECHO ON

set echo on
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE RETENTION POLICY TO REDUNDANCY 1;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

crosscheck backup;
crosscheck archivelog all;
delete noprompt expired backup;
delete noprompt expired archivelog all; 
delete noprompt obsolete;

run {
  sql 'alter system checkpoint';
  sql 'alter system switch logfile';
  sql 'alter system archive log current';
  backup incremental level 0 cumulative as compressed backupset database tag 'database-0';
  backup as compressed backupset archivelog all tag 'archives-0';
  backup as compressed backupset current controlfile tag 'controls-0';
}
exit;

backup archivelog all delete all input;
backup database plus delete input;
```

##### COMANDOS

```sql
# KILL RMAN
select b.sid, b.serial#, a.spid, b.client_info
from v$process a, v$session b
where a.addr=b.paddr and client_info like 'rman%'

alter system kill session '592,12' immediate;

# REDIRECIONANDO A SAIDA DO RMAN
rman target / msglog=\app\Administrator\virtual\scripts\log_backup\saida_rman_teste.log

---

select * from V$RMAN_CONFIGURATION;

set ORACLE_SID=PRODUCAO
rman target /
rman target sys/"****"@producao
rman target sys/"***"

catalog start with '/ora/backup/semanal/sead'; 

show all;	          	    	(mostra a configuração do RMAN)
show controlfile autobackup;

list incarnation; 	          	(mostra o BDID)
list incarnation of database;
list incarnation of database seplan;

list backup; 
list backup summary;	        (mostra o catalogo dos backups)  
list backup of database summary;
list backupset;
list backup of database;
list backup of database by backup;
list backup of archivelog from sequence 3090;
list backup of archivelog all;
list backup of archivelog all summary;
list backup of controlfile;
list backup of spfile;
list backup by file;

list backupset;
list backupset of database;
list backupset of archivelog all;
list backupset of datafile 1;

list copy by file;

list archivelog all;
list archivelog until time 'sysdate-1';
list archivelog from time  "to_date('18/08/2021 00:00:00','DD/MM/YYYY HH24:MI:SS')" until time "to_date('18/08/2021 10:00:003','DD/MM/YYYY HH24:MI:SS')";

list expired backup;
list expired backup by file;
list expired archivelog all;

report schema;	(mostra caminho dos datafiles e temp)
report need backup;
REPORT OBSOLETE;
REPORT OBSOLETE RECOVERY WINDOW OF 1 DAYS;
REPORT OBSOLETE REDUNDANCY 7;
REPORT OBSOLETE REDUNDANCY = 7 DEVICE TYPE DISK;
---
REPORT UNRECOVERABLE;
REPORT UNRECOVERABLE DATABASE;
REPORT UNRECOVERABLE TABLESPACE 'USERS';
--Ver objetos afetados pelo UNRECOVERABLE
select distinct dbo.owner,dbo.object_name, dbo.object_type, dfs.tablespace_name,
dbt.logging table_level_logging, ts.logging tablespace_level_logging
from v$segstat ss, dba_tablespaces ts, dba_objects dbo, dba_tables dbt,
v$datafile df, dba_data_files dfs, v$tablespace vts
where ss.statistic_name ='physical writes direct'
and dbo.object_id = ss.obj#
and vts.ts# = ss.ts#
and ts.tablespace_name = vts.name
and ss.value != 0
and df.unrecoverable_change# != 0
and dfs.file_name = df.name
and ts.tablespace_name = dfs.tablespace_name
and dbt.owner = dbo.owner
and dbt.table_name = dbo.object_name;
-

REPORT NEED BACKUP RECOVERY WINDOW OF 2 DAYS ;
REPORT NEED BACKUP REDUNDANCY 2 DATAFILE 1;
REPORT NEED BACKUP TABLESPACE TBS_3; # uses configured retention policy
REPORT NEED BACKUP INCREMENTAL 2; # checks entire database

RECOVERY WINDOW OF 2 DAYS DATABASE SKIP TABLESPACE TBS_2;

crosscheck archivelog all;
crosscheck archivelog until time 'sysdate-6';
crosscheck backup completed between 'sysdate-90' AND 'sysdate-6';
crosscheck backup;                	(sincroniza arquivos fisicos (TODOS) com o catalogo)
crosscheck backup of database;    	(sincroniza arquivos fisicos (DATAFILES) com o catalogo)
crosscheck backup of controlfile;
crosscheck backup of archiveslog all;
crosscheck copy;

delete backup; # ATENÇÃO!! Deleta todos os backups
delete backupset 10,11,12;
delete backup of archivelog all;
delete backup completed between 'sysdate-90' AND 'sysdate-6';

delete archivelog all;
delete archivelog all backed up 5 times to disk; # Novo
delete archivelog until sequence = 1130;
delete archivelog until time 'sysdate-6';
delete archivelog from time 'SYSDATE-10';

delete expired archivelog all;
delete expired backup;
delete force archivelog all;
delete force expired archivelog all;

DELETE OBSOLETE;
DELETE OBSOLETE REDUNDANCY = 2 DEVICE TYPE DISK;
DELETE OBSOLETE RECOVERY WINDOW OF 7 DAYS;
delete obsolete redundancy = 3;
delete obsolete device type disk;

list backupset 582;
change backupset 5 unavailable;
change datafilecopy 'd:\backup\demo_37.bak' unavailable;

# VALIDATES
backup validate datafile 1;
backup validate tablespace users;
backup validate database;
backup validate check logical datafile 1;
backup validate check logical tablespace users;
backup validate check logical database;
-- ou sem o backup na frente: 
validate database;
validate backupset 10;
---
restore validate datafile 1,2;
restore validate tablespace users,sysaux;
restore validate database;
restore database validate;
resotre validate check logical datafile 1,2;
restore validate check logical tablespace users,sysaux;
restore validate check logical database;
restore tablespace nome;
restore datafile 1;

RESTORE DATABASE PREVIEW;
RESTORE DATABASE PREVIEW SUMMARY;
RESTORE TABLESPACE dev1 PREVIEW;
RESTORE DATAFILE '/u01/oradata/devdb/dev1_01.dbf' PREVIEW;
RESTORE ARCHIVELOG ALL PREVIEW;
RESTORE ARCHIVELOG FROM SCN 234546 PREVIEW;
RESTORE DATABASE VALIDATE;

RECOVER DATABASE UNTIL CANCEL USING BACKUP CONTROLFILE;
RECOVER DATABASE PREVIEW;
recover datafile 10,13;
---
run {
  set until time "TO_DATE('18/10/2021 15:00:00','DD/MM/YYYY HH24:MI:SS')";     
  #recover database preview;
  restore database preview;
}

recover database preview until time "to_date('18/10/2021 15:00:00','DD/MM/YYYY HH24:MI:SS')";

```

##### PARAMETROS

```sql
EXEC DBMS_BACKUP_RESTORE.RESETCONFIG;
---
CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 3 TIMES TO DISK;
CONFIGURE ARCHIVELOG DELETION POLICY CLEAR;
---
CONFIGURE BACKUP OPTIMIZATION CLEAR;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK CLEAR;
CONFIGURE DEVICE TYPE DISK CLEAR;
--
CONFIGURE RETENTION POLICY TO REDUNDANCY 7; 
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET;
---
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F';
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

# EXEMPLO
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE DEFAULT DEVICE TYPE TO DISK;
CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 1 TIMES TO DISK;

# EXEMPLO DE POLITICA DE BACKUP
CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 1 TIMES TO DISK;
CONFIGURE RMAN OUTPUT TO KEEP FOR 1 DAYS;
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 1 DAYS;
DELETE OBSOLETE RECOVERY WINDOW OF 1 DAYS
DELETE ARCHIVELOG ALL BACKED UP 1 TIMES TO DISK;
```

##### LIMPEZA DO CATÁLOGO DO CONTROLFILE

```sql
# LIMPEZA DO CATALOGO DO CONTROLFILE
run {
  crosscheck backup;  
  crosscheck backup of database;
  crosscheck backup of controlfile;
  crosscheck backup of archivelog all;
  crosscheck archivelog all;
  delete expired backup;
  delete expired archivelog all;
  delete expired backup of archivelog all;
  delete obsolete;
}

  delete noprompt expired backup;
  delete noprompt expired archivelog all;
  delete noprompt expired backup of archivelog all;
  delete noprompt obsolete;

run {
   crosscheck backup;
   delete noprompt expired backup;
   crosscheck archivelog all;
   delete noprompt expired archivelog all;
}
```

##### APLICANDO ARCHIVELOGS EM UM COLD BACKUP

```sql
sqlplus / as sysdba
startup
SQL> set timing on;
SQL> show parameter log_archive_format
SQL> archive log list
SQL> select group#,thread#,sequence#,bytes,members,archived,status from v$log;
SQL> shutdown immediate
SQL> exit

cp -a /u01/app/oracle/oradata /backup
sqlplus / as sysdba
startup

SQL> create user scott identified by tiger default tablespace users;
SQL> grant connect,resource to scott;
SQL> connect scott/tiger

SCOTT> create table t1 (id number);
SCOTT> insert into t1 select level from dual connect by level <= 100000;
SCOTT> commit;
SCOTT> update t1 set id=1000;
SCOTT> rollback;
SCOTT> drop table t1 purge;
SCOTT> create table t2 (data timestamp);
SCOTT> insert into t2 select localtimestamp from dual;
SCOTT> commit;

SQL> connect / as sysdba;
SQL> alter system archive log current;
SQL> select recid,name from v$archived_log;
SQL> shutdown immediate
SQL> exit

ls -lhatr
rm -rf /u01/app/oracle/oradata
sqlplus / as sysdba

SQL> startup mount
SQL> recover database;
ORA-00283: sessão de recuperação cancelada devido a erros
ORA-00264: nenhuma recuperação necessária
SQL> recover database using backup controlfile;

SQL> alter database open resetlogs;
SQL> recover database using backup controlfile until cancel;
SQL> alter database open resetlogs;
SQL> connect scott/tiger
SQL> select * from t2; 
```

##### EXEMPLOS DE SCRIPTS DE BACKUP 

```sql
#!/bin/bash
#
# Script de backup incremental do banco de dados Oracle, conforme indicado em:
#    "Oracle Database 2 Day DBA"
#    "Oracle Database Backup and Recovery User's Guide"
# Acrescentei o backup do init.ora e o backup trace do controlfile.
#
# Por Abrantes Araújo Silva Filho
#
# ANTES DE EXECUTAR A ROTINA DE BACKUP, configure o RMAN com os seguintes
# parâmetros (dependentes da versão do database e se enterprise ou standard): 
# (atenção com os PATHs também!):
#
# Oracle 12c (12.1.0.1.0) Standard Edition One: 
#   - CONFIGURE RETENTION POLICY TO REDUNDANCY 1; 
#   - CONFIGURE BACKUP OPTIMIZATION ON; 
#   - CONFIGURE DEFAULT DEVICE TYPE TO DISK; 
#   - CONFIGURE CONTROLFILE AUTOBACKUP ON; 
#   - CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/u09/fast_recovery_area/ORACLE1/controlfile_autobackup/%F'; 
#   - CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; 
#   - CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; 
#   - CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; 
#   - CONFIGURE MAXSETSIZE TO UNLIMITED; 
#   - CONFIGURE ENCRYPTION FOR DATABASE OFF; 
#   - CONFIGURE ENCRYPTION ALGORITHM 'AES128'; 
#   - CONFIGURE COMPRESSION ALGORITHM 'BASIC' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE; 
#   - CONFIGURE RMAN OUTPUT TO KEEP FOR 7 DAYS; 
#   - CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 2 TIMES TO DISK; 
#   - CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u09/fast_recovery_area/ORACLE1/controlfile_snapshot/snapcf_oracle.f';
#
# Depois de tudo pronto, crie um crontab para o usuário oracle:
#
# Endereço de e-mail do log do crontab:
#    MAILTO=abrantesasf@gmail.com
#
#
# Roda o backup incremental diário, 19:00h.
#    00 19 * * * /home/oracle/bin/backup_oracle.sh
#
#
# Não altere este script se não souber o que está fazendo!

# Variáveis de ambiente
#######################
export ORACLE_HOME=/u01/app/oracle/product/12.1.0.1.0/db export ORACLE_SID=oracle1 export PATH=$ORACLE_HOME/bin:$PATH 

# Roda RMAN 
###########
rman <<EOF connect target / RUN {  ALLOCATE CHANNEL disco_de_backup DEVICE TYPE DISK;
RECOVER COPY OF DATABASE WITH TAG backup_incremental_diario UNTIL TIME "SYSDATE-3";
BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG backup_incremental_diario DATABASE PLUS ARCHIVELOG;
CROSSCHECK BACKUP;  DELETE NOPROMPT OBSOLETE; } exit EOF

# Cria backup do init.ora e backup trace do controlfile: ######################################################## 
# Backup do init.ora antigo: 
if [ -a /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora.bak ] ; then
  rm /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora.bak
  mv /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora.bak
else if [ -a /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora ] ; then
  mv /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora /u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora.bak
  fi
 fi

# Backup do controlfile antigo:
if [ -a /u09/fast_recovery_area/ORACLE1/outros/control_file.txt.bak ] ; then
  rm /u09/fast_recovery_area/ORACLE1/outros/control_file.txt.bak
  mv /u09/fast_recovery_area/ORACLE1/outros/control_file.txt /u09/fast_recovery_area/ORACLE1/outros/control_file.txt.bak
 else  if [ -a /u09/fast_recovery_area/ORACLE1/outros/control_file.txt ] ; then
  mv /u09/fast_recovery_area/ORACLE1/outros/control_file.txt /u09/fast_recovery_area/ORACLE1/outros/control_file.txt.bak
  fi
 fi

# Backup do init.ora novo e do controlfile novo:
sqlplus /nolog << EOF
connect / as sysdba
create pfile='/u09/fast_recovery_area/ORACLE1/outros/initoracle1.ora' from spfile='/u01/app/oracle/product/12.1.0.1.0/db/dbs/spfileoracle1.ora';
alter database backup controlfile to trace as '/u09/fast_recovery_area/ORACLE1/outros/control_file.txt';
exit
EOF
```

