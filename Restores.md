##### RESTORE PRODUÇÃO-CLINICAS EM LAB  (19/08/2021)

```sql
RESTORE SPFILE TO PFILE '/u01/app/oracle/product/9.2.0/dbs/initorcl.ora' FROM AUTOBACKUP;
---
startup nomount;
restore controlfile from 'C:\app\Administrador\virtual\FRA\CLINICAS\AUTOBACKUP\2021_08_18\O1_MF_S_1080947605_JKVHL62F_.BKP';
alter database mount;
list backup;
crosscheck backup;
delete expired backup;
catalog recovery area;
restore database;
recover database 
alter database open resetslog;

```

##### RESTORE PRODUÇÃO-AG EM LAB  (15/08/2021)

```sql
# RESTORE DE PRODUCAO-AG PARA LAB 
# C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\
# CONTROLFILE, LOGFILE, TEMPFILE e DBFILE
#restore controlfile from autobackup;
#restore controlfile validate;

#PRODUCAO-LAB (DBID=3455422565)
#PRODUCAO-AG  (DBID=3331875998)

#show parameter db_create;
#alter system set DB_CREATE_FILE_DEST='C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA';

startup pfile= 'C:\app\Administrator\virtual\product\12.2.0\dbhome_1\database\INITCLINICAS.ORA';

set ORACLE_SID=PRODUCAO
rman target /
startup nomount;
--set DBID=3331875998;
restore spfile from 'C:\BackupAG\O1_MF_S_1080425990_JKCL5PCN_.BKP';
create pfile from spfile;
shutdown immediate;

# OBS: ACERTA OS CONTROLS FILES NO INITPRODUCAO.ORA - NESSE CASO FICOU SO UM CONTROL PORQUE NA NOVA MAQUINA NÃO TINHA DRIVE E:
startup nomount pfile='C:\APP\ADMINISTRADOR\VIRTUAL\PRODUCT\12.2.0\DBHOME_1\DATABASE\INITPRODUCAO.ORA';
restore controlfile from 'C:\BackupAG\O1_MF_S_1080425990_JKCL5PCN_.BKP';
alter database mount;
catalog start with 'C:\BackupAG';
catalog start with 'C:\app\Administrador\virtual\FRA';
catalog recovery area;

# SINCRONIZAR O BACKUP
# set archivelog destination to 'C:\BackupAG';
list backup;
run {
CROSSCHECK BACKUP;
CROSSCHECK COPY;
CROSSCHECK backup of controlfile;
CROSSCHECK archivelog all;
delete expired archivelog all;
delete expired backup;
delete obsolete;
}
delete obsolete device type disk;

list backup;

---------------------------------------------
# MODELOS PARA RESTAURAR EM OUTRO CAMINHO
---------------------------------------------
select 'Set Newname For Datafile ' || file# || ' to ''' || name || ''';' from v$datafile;
-
select 'Set Newname For Datafile ''' || name || ''' to ''' || name || ''';' from v$datafile;
select 'Set Newname For Datafile ''' || name || ''' to ''' || name || ''';' from v$tempfile;
select 'Set Newname For Datafile ''' || member || ''' to ''' || member || ''';' from v$logfile;

-----

select 'Alter Database Rename File ''' || name || ''' To ''' || name || ''';' from v$datafile;
select 'Alter Database Rename File ''' || name || ''' To ''' || name || ''';' from v$tempfile;
select 'Alter Database Rename File ''' || member || ''' To ''' || member || ''';' from v$logfile;

# DATAFILES 
select 'Set Newname For Datafile ''' || name || ''' 
    to ''' || 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\' || substr(name,instr(name, '\',-1)+1) || ''';' 
  from v$datafile;

# LOGFILES
select 'Alter Database Rename File ''' || member || ''' 
    to ''' || 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG' || substr(member,instr(member, '\',-1)+1) || ''';'
  from v$logfile
  where member like '%E:%';

select 'Alter Database Rename File ''' || member || ''' 
    to ''' || 'C:\APP\ADMINISTRADOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\' || substr(member,instr(member, '\',-1)+1) || ''';'
  from v$logfile
  where member like '%C:%';

# TEMPFILES
select 'Alter Database Rename File ''' || name || ''' 
    to ''' || 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\' || substr(name,instr(name, '\',-1)+1) || ''';'
  from v$tempfile
  
----------------------------------    
# RESTORE DATABASE EM OUTRO PATH 
-----------------------------------
# Fazer primeiro os "alter database rename file" e depois os "set new names"

Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO01A.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO01A.LOG';
Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO02A.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO02A.LOG';
Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO03A.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO03A.LOG';

Alter Database Rename File 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO01B.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO01B.LOG';
Alter Database Rename File 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO02B.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO02B.LOG';
Alter Database Rename File 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO03B.LOG' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO03B.LOG';

Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\TEMP01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP01.DBF';
    
Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\TEMP02.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF';

# OBS: Copiar primeiro para o bloco de notas e depois copiar do bloco de notas para o RMAN

run {
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\SYSTEM01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SYSTEM01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\SYSAUX01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SYSAUX01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\UNDOTBS01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\UNDOTBS01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\USERS01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\USERS01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT09.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT09.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT02.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT02.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT03.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT03.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT04.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT04.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT05.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT05.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT06.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT06.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT07.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT07.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX01.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX02.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX02.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX03.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX03.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX04.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX04.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX05.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX05.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX06.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX06.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX08.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX08.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX07.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX07.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF';
Set Newname For Datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\TS_WORKFLOW01.DBF' 
    to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TS_WORKFLOW01.DBF';

restore database;
switch datafile all;
}
# restore validate database;
recover database;
alter database open resetlogs;

create spfile from pfile;

# Ajustar os redos, controls files e agregados


alter database mount;
restore database preview; # Checar se backup pieces e archivelogs estão disponiveis
restore database;
-recover database until cancel;
recover database;

alter database open; (Não resolveu)
alter database open resetlogs; (Precisou resetlogs)

# Nova encarnação de backups
# Fazer novo backup
RMAN> list backup;
RMAN> validate database;
RMAN> crosscheck archivelog all; 
RMAN> 
RMAN> RMAN> RMAN> RMAN> 

RESTORE DATABASE VALIDATE;

# Criar a mesma estrutura de pastas em LAB
# Copiar os arquivos do backup
set ORACLE_SID=PRODUCAO

rman target /
report need backup; # Verificar se temos backups de todos os arquivos
list backup summary;
list backup;
restore database validade; # Verificar backups corrompidos (Não escreve em disco)

```

##### RESTORE EM LAB DIA 07/08/2021

```sql
rman target /
shutdown immediate;
startup nomount;
---
RMAN> restore controlfile from autobackup;
RMAN> alter database mount;
RMAN> restore database;
RMAN> recover database;
RMAN> alter database open; (Não resolveu)
RMAN> alter database open resetlogs; (Precisou resetlogs)
# Nova encarnação de backups
# Fazer novo backup
RMAN> list backup;
RMAN> validate database;
RMAN> crosscheck archivelog all; 
RMAN>
```

##### RESTAURANDO O BACKUP EM OUTRO COMPUTADOR

```sql
# Criar a mesma estrutura de pastas da ORIGEM no DESTINO
\app\Adminstrator\virtual\oradata\producao
      controlfile
      datafile
      onlinelog
\app\Adminstrator\virtual\fast_recovery_area\producao\producao
      archivelog
      autobackup
      backupset
      controlfile
      onlinelog

# CRIAR A INSTANCIA/SENHA
# Criar com dbca
C:\> dbca
---
oradim -new -sid PRODUCAO
services.msc
orapwd file=PWDproducao password=* format=12 force=y entries=10

oradim -new -sid AG
oradim -delete -sid AG
# Essa senha é somente para acesso via prompt de comando
orapwd file=PWDitabs password= format=12 force=y entries=10

# Se seus servidores antigos e novos não usam os mesmos caminhos 
# de arquivo para dados Oracle, execute o seguinte:

select '  Set Newname For Datafile ' || file# || ' to ''' || name || ''';' from v$datafile;
select '  Alter Database Rename File ''' || Member || ''' To ''' || Member || ''';' from v$logfile;

create pfile from spfile;

startup pfile = 'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITAG.ORA';

set ORACLE_SID=PRODUCAO
set ORACLE_SID=AG
echo %ORACLE_SID%
---
restore controlfile from 'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\SNCFPRODUCAO.ORA';

set DBID=3426120398; # DBID do Banco ORIGEM AG-HOMOLOGAÇÃO
catalog start with 'C:\BackupAG'; 

C:\> rman target /
startup nomount;
set DBID=3331875998; # DBID do Banco ORIGEM AG-PRODUÇÃO
restore spfile from 'C:\BackupAG\O1_MF_S_1080425990_JKCL5PCN_.BKP';
shutdown immediate;

set DBID=3331875998; 
startup nomount;
# restore controlfile validate from 'C:\BackupAG\O1_MF_S_1080425990_JKCL5PCN_.BKP';
restore controlfile from 'C:\BackupAG\O1_MF_S_1080425990_JKCL5PCN_.BKP';

# AJUSTAR OS CAMINHOS DOS DATAFILES 
'C:\app\Administrador\virtual\FRA\AG\AG\CONTROLFILE\O1_MF_JKFVKP98_.CTL'
'C:\app\Administrador\virtual\oradata\AG\CONTROLFILE\O1_MF_JKFVKP67_.CTL'

alter database mount;

# SINCRONIZAR O catalogo caso seja necessário
# Se o backup estiver com a  mesma estrutura de pastas da ORIGEM, usar so os crosschecks
# Não precisa do catalog start with 'c:'

# backup validate database;
# validate database;
restore database;

# recover database preview;
recover database;

alter database open;
alter database open resetlogs;
---
select log_mode from v$database;
select name,open_mode from v$database;
select instance_name,status from v$instance;

# Depois disso fazer logo um novo backup;
```

##### RESTORE/RECOVER

```sql
startup nomount;
exit;
rman target /
restore controlfile from '/path/controlfile_name'; --pegar o control file do nivel 0
sql 'alter database mount';
exit;

sqlplus / as sysdba
select name, open_mode from v$database; --(DBINST,MOUNTED)
exit;

rman target /
list backup of database summary;
restore database from tag='TAG????'; --pegar TAG nivel 0
list backup of database summary;

recover database from tag='TAG?????';--pegar TAG nivel 1
sql  'alter database open resetlogs';
exit;

sqlplus '/ as sysdba'
select name, open_mode from v$database;
archive log list;

select name from v$controlfile;
select name from v$datafile;

set ORACLE_SID=PRODUCAO
rman target /
rman target sys/"*"@producao
show all;
list backup;
list backup of database summary;
```

##### RMAN NO WINDOWS

```sql
---
set oracle_home=d:\app\\\
set oracle_sid=producao
sqlplus /nolog
conn /as sysdba
archive log list;
select tablespace_name from dba_tablespaces;
show con_name;
select file_name from dba_data_files;
desc dba_data_files;
select file_name, file_id from dba_data_files;
set linesize 120
column file_name for a70
/
host
rman
connect target /
report schema
list backup;
list backup summary;
report need backup;
show all;
list copy;
backup tablespace tbs01;

SQL> host rman  vai para -> RMAN>

sqlplus / as sysdba 
archive log list;
select name form v$controlfile;
select group#,members,sequence#,status from v$log;
select name,sequence# from v$archived_log;
alter system switch logfile;

slqplus / as sysdba
show parameter pfile

validate database;
list backup
backup database;
```

##### RECUPERAÇÃO DO DATABASE PARA SEU DESTINO ORIGINAL

```sql
SET DBID ;
STARTUP FORCE NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RESTORE DATABASE PREVIEW;
RESTORE DATABASE;
RECOVER DATABASE UNTIL CANCEL;
ALTER DATABASE OPEN RESETLOGS;
```

##### RESTORE & RECOVER

```sql
restore database validate;
validate database;

rman
connect target /
startup mount
restore controlfile from  'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\SNCFORCL';
restore controlfile from backup;
restore controlfile from tag '';
restore controlfile from autobackup;

alter database mount;
sql 'alter database mount';
restore database;
recover database;
slq 'alter database open resetlogs';

run{
set archivelog destination to 'C:\app\Administrador\virtual\FRA';
restore archivelog all;
restore archivelog from sequence 29 until sequence 31 preview;
restore archivelog from SCN 56789;
};

restore archivelog preview;
restore achivelog from SCN 56789 preview;
restore archivelog all preview;
restore database preview;
restore database preview summary;

restore datafile 1;
restore database;

recover datafile 1 preview;

recover database until cancel using backup controlfile;
recover database until time ;
recover database until chance ;
---
list failure;
advise failure;
---
run {
set archivelog destination to '';
crosscheck archivelog all;
recover datafile 1 preview;
}
---
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000029_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000030_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000031_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000032_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000033_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000034_1070383006.0001';
catalog archivelog 'C:\app\Administrador\virtual\oradataX\archivelogs\ARC0000000035_1070383006.0001';
```

##### QUANDO SE PERDE OS ARCHIVELOGS

```sql
restore database until time="to_date('22/04/2021 20:06:00','dd/mm/yyyy hh24:mi:ss')";
recover database until time="to_date('22/04/2021 20:06:00','dd/mm/yyyy hh24:mi:ss')";
alter database open resetlogs;

run
{
set until time="to_date('22/04/2021 20:06:00','dd/mm/yyyy hh24:mi:ss')";
restore database;
recover database;
alter database open resetlogs;
}
---
SELECT INPUT_TYPE, STATUS,TO_CHAR(START_TIME,'DD/MM/YYYY HH24:MI') START_TIME,
       TO_CHAR(END_TIME,'DD/MM/YYYY HH24:MI') END_TIME, ELAPSED_SECONDS/3600 HRS
  FROM V$RMAN_BACKUP_JOB_DETAILS
 WHERE INPUT_TYPE='ARCHIVELOG'
 ORDER BY SESSION_KEY;
```

##### RENOMEAR INSTÂNCIA

```sql
# PROBLEMAS NO SPFILE: ORA-01078, LRM-00109
--Resolvido: 
--ORACLE_SID: registro do windows
--PFILE: caminho dos controlfiles
--Iniciar com o pfile
--Trocar a senha de acesso do serviço da instancia

#startup pfile = #'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITPRODUCAO.ORA';
#restore spfile to 'C:\app\Administrator\virtual\FRA\PRODUCAO' from autobackup;

startup pfile = 'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITPRODUCAO.ORA';

restore spfile from 'C:\app\Administrator\virtual\FRA\PRODUCAO\AUTOBACKUP\2021_08_13 \O1_MF_S_1080472526_JKDZMZ0D_.BKP';
restore spfile from autobackup;
restore spfile validate;
restore controlfile validate;
restore controlfile from autobackup;

create pfile from spfile;
create spfile from pfile;

#-------------------------
# EXCLUIR BANCO DE DADOS
#-------------------------
dbca -deleteDatabase -sourceDB AG -sysDBAUserName SYS -sysDBAPassword nomanager -silent
ou
set ORACLE_SID=ag
sqlplus / as sysdba
startup mount exclusive restrict
o
startup mount exclusive restrict pfile = 'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITAG.ORA'
drop database;
exit
----

set DBID=3426120398; --LAB-AG
# ALTERAR O NOME DA INSTANCIA
sqlplus / as sysdba
shutdown immediate;
startup mount;
exit;
C:\> nid TARGET=sys/oracle12c@AG dbname=PRODUCAO SETNAME=YES
RESPONDER -> Y

orapwd file=PWDPRODUCAO password=oracle12c format=12 force=y entries=10
startup pfile = 'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITPRODUCAO.ORA';

-------

rman target /
report schema;

rman target /
shutdown immediate;
startup mount;

# Abrir uma novo shell cmd
set ORACLE_SID=<<OLD_SID>>
NID TARGET=/ DBNAME=<<NEW_SID>>

# RESTAURAR COM OUTRO NOME DE INSTANCIA

# PREPARAR A NOVA INSTANCIA
oradim -new -sid <<old sid name>>

Set Oracle_SID=<<old sid name>>
RMAN target /
```

