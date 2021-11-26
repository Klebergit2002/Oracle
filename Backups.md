##### AGENDAMENTOS DOS BACKUPS ORACLE-IPES

```ini
# AG-PRODUÇÃO
-----------
BACKUP RMAN
-----------
Agenda: Rman_Backup_diario: 22 hs (Seg,Ter,Qui,Sex,Sab)
Script: C:\app\administrator\virtual\rman_scripts\backup_diario_prod_ag.bat
Local : F:\FRA
Tempo : 18 minutos

Agenda: Rman_Backup_full: 22 hs (Dom,Qua)
Script: C:\app\administrator\virtual\rman_scripts\backup_full_prod_ag.bat
Local : F:\FRA
Tempo : 01:50h

---------
DATA PUMP
---------
Agenda: Backup_export: 16hs (Todos os dias)
Script: C:\app\administrator\virtual\scripts\Export_full_producao.bat
Local : E:\bkp_export
Tempo : 04:40h

# CLINICAS-PRODUÇÃO
-----------
BACKUP RMAN
-----------
Agenda: Rman_Backup_diario: 22 hs (Seg,Ter,Qui,Sex,Sab)
Script: C:\app\administrator\virtual\rman_scripts\backup_diario_prod_clinicas.bat
Local : E:\FRA
Tempo : 5 minutos

Agenda: Rman_Backup_full: 22 hs (Dom,Qua)
Script: C:\app\administrator\virtual\rman_scripts\backup_full_prod_clinicas.bat
Local : E:\FRA
Tempo : 15 minutos

---------
DATA PUMP
---------
Agenda: BKP_Export_Clinicas: 16hs (Todos os dias)
Script: C:\app\administrator\virtual\scripts\BKP_Export_Clinicas.bat
Local : C:\app\Administrator\virtual\admin\clinicas\dpdump
Tempo : 02:15h

# ITABS-PRODUÇÃO
-----------
BACKUP RMAN
-----------
Agenda: Rman_Backup_diario:
Script: 
Local : 
Tempo :

Agenda: Rman_Backup_full: 
Script: 
Local : 
Tempo : 

---------
DATA PUMP
---------
Agenda: Backup_itabs: 19hs (Todos os dias)
Script: C:\Backup\itabs_exp.bat
Param : C:\Backup\itabs.par
Local : E:\DPDUMP
Tempo : 01:15h

```

Boa tarde Igor
Agendei todos os Backups do RMAN para as 22hs e os Backups do DATA_PUMP estão agendados para as 16hs para não chocar com os do RMAN.

Vi que você alterou o script BKP_Export_full.bat em CLINICAS para fazer a transferencia dos backups para a rede, mas esse mesmo script fazia o data_pump do banco Clinicas, entao criei outro script para fazer esse backup (BKP_Export_Clinicas.bat).

Com relação ao ITABS, você está pegando o DUMP do lugar errado, por enquanto estou fazendo ele em (E:\DPDUMP), até que agente habilite os archivelogs e defina uma nova estratégia, esse DUMP está sendo feito as 19hs todos os dias da semana

Agora vamos acompanhar para ver se precisa fazer alguns ajustes.

##### MODELOS DE BACKUP

```sql
# Estratégia de backup
Dom Seg Ter Qua Qui Sex Sab
FUL INC INC INC INC INC INC (POLICY TO RECOVERY WINDOW OF 2 DAYS;)
FUL INC INC INC FUL INC INC (POLICY TO RECOVERY WINDOW OF 2 DAYS;)
FUL INC INC FUL INC INC FUL (POLICY TO RECOVERY WINDOW OF 2 DAYS;)

rman target /

# BACKUP SEMANAL
run {
  sql 'alter system checkpoint';
  sql 'alter system switch logfile';
  sql 'alter system archive log current';
  backup incremental level 0 cumulative as compressed backupset database tag 'database-0';
  backup as compressed backupset archivelog all tag 'archives-0';
  backup as compressed backupset current controlfile tag 'controls-0';
}

-- delete noprompt obsolete;
-- SQL "alter database backup controlfile to trace as /adirectory/controlfile.txt"

# BACKUP DIARIO
run {
  sql 'alter system checkpoint';
  sql 'alter system switch logfile';
  sql 'alter system archive log current';

  backup incremental level 1 cumulative as compressed backupset database tag 'database-1';
  backup as compressed backupset archivelog all tag 'archives-1';
  backup as compressed backupset current controlfile tag 'controls-1';
}
```

##### MONITORANDO BACKUPS

```sql
# POWER SHELL
Get-Content C:\app\Administrator\virtual\scripts\log_backup\backup_diario_homo_ag_2021-05-02_16-11-03.log -Wait -Tail 10

# SQLDEVELOPER/SQ
select sid,serial#,context,sofar,totalwork,round(sofar/totalwork*100,2) "%_complete"
  from v$session_longops
 where opname like 'RMAN%'
   and opname not like '%aggregate%'
   and totalwork != 0
   and sofar <> totalwork;
---
select operation as "OPERACAO",object_type as "tipo",status,output_device_type as "media",
           to_char(end_time,´DD-MM-RRRR HH24:MI:SS´) as "DATA",
           round(MBYTES_PROCESSED/1024,2) as "TAMANHO(MB)"
    from v$rman_status
    where operation <> 'CATALOG'
          and trunc(end_time)>=trunc(sysdate-1)
   order by  end_time;
---
select * from v$rman_status;
```

##### DATA PUMP - CONSISTENT - EXPDP/IMPDP

```sql
# MODOS DE IMPORTAÇÃO/EXPORTACAO COM DATAPUMP
Modo FULL
Modo SCHEMA
Modo TABLE
Modo TABLESPACE
Modo TRANSPORTAVEL TABLESPACE

# OBJETO DE DIRETÓRIO
CREATE OR REPLACE DIRECTORY dump AS '/u01/app/oracle/oradata/';
GRANT READ, WRITE ON DIRECTORY dump TO public;
OBS: public-> qualquer usuário do banco
---
select * from dba_directories;
select * from dba_users order by account_status desc,user_id desc;
---
GRANT DATAPUMP_EXP_FULL_DATABASE, DATAPUMP_IMP_FULL_DATABASE TO user_name;

expdp system/IPESAUDE@clinicas_ip DIRECTORY=DATA_PUMP_DIR DUMPFILE=portal.dmp LOGFILE=portal_exp.log FLASHBACK_TIME=systimestamp SCHEMAS=portal PARALLEL=4 COMPRESSION=all COMPRESSION_ALGORITHM=basic

drop user portal cascade

# TABLESPACES DOS SCHEMAS
select distinct owner, tablespace_name from dba_segments
 where owner in ( select username from dba_users where user_id > 47 );

# DATAFILES
select * from dba_data_files where tablespace_name in 
   (select distinct tablespace_name from dba_segments
    where owner in (select username from dba_users where user_id > 47))
order by file_name;

# CRIACAO DOS USUÁRIOS/SCHEMAS
select 'create user '||username||' identified by values '''|| password||''' '||
       'default tablespace '|| default_tablespace || ' temporary tablespace '||temporary_tablespace || ' profile '||profile||';'
from dba_users
where user_id > 47 order by user_id;

# TEM QUE CRIAR SCHEMA PORTAL COM TODOS OS GRANTS E TABLESPACES ASSOCIADOS
impdp system/nomanager@clinicas DIRECTORY=DATA_PUMP_DIR DUMPFILE=portal.dmp LOGFILE=portal_imp.log SCHEMA=portal 

----

call expdp ipesaude/bennerhosp01@clinicas full=y dumpfile=BKP_EXPDP_clinicas_full.dmp logfile=BKP_EXPDP_clinicas_full.log directory=DATA_PUMP_DIR PARALLEL=4 COMPRESSION=all COMPRESSION_ALGORITHM=basic

call expdp parfile=expdp_itabs.par
USERID=system/manager$itabs@itabs
FULL=yes
FLASHBACK_TIME=systimestamp
REUSE_DUMPFILES=y
DIRECTORY=DPDUMP_DIR
DUMPFILE=itabs.dmp
LOGFILE=itabs_exp.log
PARALLEL=4
COMPRESSION=all 
COMPRESSION_ALGORITHM=basic

#-----------------------
# IMPORTAÇÃO COM IMPDP
#-----------------------
\\172.24.8.41\e$ (administrator/1p35@ud3!@dm1n)

CREATE TABLESPACE "ITABS" DATAFILE 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\ITABS\DATAFILE\ITABS01.DBF' SIZE 512M AUTOEXTEND ON;

select * from dba_segments where owner = 'ITABS';
select distinct owner, tablespace_name from dba_segments where owner = 'ITABS';
---
impdp ipesaude/bennerhosp01@clinicas DIRECTORY=data_pump_dir DUMPFILE=BKP_EXPDP_clinicas_full.dmp REMAP_SCHEMA=ipesaude:ipesaude LOGFILE=ipesaude_imp.log

impdp system/nomanager@clinicas DIRECTORY=DATA_PUMP_DIR DUMPFILE=clinicas.dmp REMAP_SCHEMA=ipesaude:ipesaude

impdp system/manager$itabs@ITABS DIRECTORY=DATA_PUMP_DIR DUMPFILE=itabs.dmp REMAP_SCHEMA=ipesaude:ipesaude LOGFILE=itabs_impdp.log

# CRIAR ESTRUTURA FISICA

select * from dba_recyclebin order by type;
select * from recyclebin$ order by original_name;
select * from dba_recyclebin where object_name='BIN$xKmddIHnQAm5SfoCyx4xXg==$0';
PURGE RECYCLEBIN;	-- Apaga a lixeira do usuario atual
PURGE DBA_RECYCLEBIN;	-- Apaga a lixeira de todos os usuarios

# MODELO itabs_impdp.par
impdp parfile=itabs_impdp.par
impdp parfile=itabs_impdp.par sqlfile=itabs_ddl.sql

USERID=system/manager$itabs
SCHEMAS=CAMPANHAMENSAGENS,PORTALBENEF,GESTAO,AGENDAIPES,GDPORTAL
DUMPFILE=itabs.dmp
LOGFILE=itabs_impdp.log
DIRECTORY=DATA_PUMP_DIR
PARALLEL=4
REMAP_TABLESPACE=ITABS:ITABS
REMAP_TABLESPACE=USERS:ITABS
REMAP_TABLESPACE=SYSTEM:ITABS

CLUSTER=N
REMAP_SCHEMA=SCOTT:MYDATA_TABLE
TABLE_EXISTS_ACTION=APPEND
TABLE_EXISTS_ACTION=REPLACE

# CONFERENCIA 
CREATE PUBLIC DATABASE LINK PROD_ITABS
CONNECT TO system IDENTIFIED BY manager$itabs 
USING '172.24.8.41:1521/itabs';
---
select object_type,count(*) from dba_objects where owner='CAMPANHAMENSAGENS' group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner='CAMPANHAMENSAGENS' group by object_type;

select object_type,count(*) from dba_objects where owner='GDPORTAL' group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner='GDPORTAL' group by object_type;

select object_type,count(*) from dba_objects where owner='PORTALBENEF' group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner='PORTALBENEF' group by object_type; --

select object_type,count(*) from dba_objects where owner='GESTAO' group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner='GESTAO' group by object_type; --

select object_type,count(*) from dba_objects where owner='AGENDAIPES' group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner='AGENDAIPES' group by object_type;

# APAGAR SCHEMAS/USERS
drop user GESTAO CASCADE;
drop user CAMPANHAMENSAGENS CASCADE;
drop user PORTALBENEF CASCADE;
drop user AGENDAIPES CASCADE;
drop user GDPORTAL CASCADE;
```

##### ALTERANDO O CHARSET DO BANCO

```sql
SELECT value$ FROM sys.props$@PROD_ITABS WHERE name = 'NLS_CHARACTERSET'; 
SELECT value FROM nls_database_parameters@PROD_ITABS WHERE parameter = 'NLS_CHARACTERSET';

SELECT value FROM v$nls_parameters@PROD_ITABS;
--
SELECT value$ FROM sys.props$ WHERE name = 'NLS_CHARACTERSET'; 
--

SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER SYSTEM ENABLE RESTRICTED SESSION;
ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
ALTER DATABASE OPEN;
ALTER DATABASE CHARACTER SET WE8MSWIN1252; --Erro ORA-12712
SHUTDOWN IMMEDIATE;
STARTUP;
```



##### DATA PUMP TOTALMENTE TRANSPORTÁVEL - Version 11 ou superior

```sql
# BANCO ORIGEM
SELECT tablespace_name, file_name FROM dba_data_files; 
---
ALTER TABLESPACE example READ ONLY;
ALTER TABLESPACE fsindex READ ONLY;
ALTER TABLESPACE fsdata READ ONLY;
ALTER TABLESPACE users READ ONLY;

expdp system FULL=y TRANSPORTABLE=always VERSION=12 DUMPFILE=expdat.dmp DIRECTORY=dp

# Copiar os arquivos abaixo para o Banco Destino
expdat.dmp 
example01.dbf 
fsdata01.dbf
fsindex01.dbf
users01.dbf
---
# BANCO DESTINO
ALTER TABLESPACE example READ WRITE;
ALTER TABLESPACE fsdata READ WRITE;
ALTER TABLESPACE fsindex READ WRITE;
ALTER TABLESPACE users READ WRITE;

impdp system@PDB2 FULL=y DIRECTORY=dp \
TRANSPORT_DATAFILES='/u02/app/oracle/oradata/ORCL/PDB2/example01.dbf',\'/u02/app/oracle/oradata/ORCL/PDB2/fsdata01.dbf',\'/u02/app/oracle/oradata/ORCL/PDB2/fsindex01.dbf,'\'/u02/app/oracle/oradata/ORCL/PDB2/users01.dbf'

```

