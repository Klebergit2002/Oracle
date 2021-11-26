#### REALOCAR A FRA																										

```sql
# ALTERAR O DESTINO E TAMANHO DA NOVA FRA
sqlplus / as sysdba
alter system set DB_RECOVERY_FILE_DEST='C:\app\Administrator\virtual\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=150G SCOPE=BOTH;

# Copiar os arquivos para a nova FRA (Pelo S.O.) 

# TESTAR O NOVO LOCAL
alter system switch logfile;
alter system checkpoint;

# SE FOR REMOVER OS ARQUIVOS DA ANTIGA FRA PARA A NOVA
RMAN> CATALOG START WITH 'C:\app\Administrator\virtual\FRA'; ou
RMAN> CATALOG RECOVERY AREA [NOPROMPT]; ou
RMAN> CATALOG DB_RECOVERY_FILE_DEST [NOPROMPT];

# DEPOIS LIMPAR O CATALOGO DO REGISTRO ANTIGO
RMAN> CROSSCHECK BACKUP;
RMAN> CROSSCHECK ARCHIVELOG ALL;
RMAN> DELETE EXPIRED BACKUP;
RMAN> DELETE EXPIRED ARCHIVELOG ALL;

# VERIFICANDO O DESTINO DOS ARCHIVES
select dest_name,destination from v$archive_dest;
#LOG_ARCHIVE_DEST_1 = USE_DB_RECOVERY_FILE_DEST

# MOVER O CONTROLFILE PARA A NOVA FRA
ALTER SYSTEM SET CONTROL_FILES=c1,c2,novo_c3 scope=spfile;

# OBS: A FRA não reconheceu
ALTER SYSTEM SET CONTROL_FILES='C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\CONTROLFILE\O1_MF_HP6Y9YVR_.CTL','C:\APP\ADMINISTRATOR\VIRTUAL\FRA\PRODUCAO\CONTROLFILE\O1_MF_HP6Y9YXG_.CTL' scope=spfile;

SQL> shutdown immediate;
SQL> exit
# Copiar o controlfile pelo SO para o novo local  
$ sqlplus / as sysdba
SQL> startup

# MOVER REDOLOGS PARA A NOVA FRA
# REALOCAR
select * from v$log;
select * from v$logfile;
select group#, bytes/1024/1024 as MB, members, status from v$log ;
select * from v$history;
---
shutdown immediate;
# Copiar os redos para a nova localização
startup mount;

alter database rename file 
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO01B.LOG',
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO02B.LOG', 
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO03B.LOG'
TO  
'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO01B.LOG',
'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO02B.LOG', 
'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\PRODUCAO\ONLINELOG\REDO03B.LOG';

alter database open;
---
# Para utilizar os novos redo logs
alter system switch logfile;   # Algumas vezes para ativar os novos redo logs

CATALOG START WITH 'C:\app\Administrador\virtual\FRA';
CATALOG RECOVERY AREA NOPROMPT;
CATALOG BACKUPPIECE '?/oradata/01dmsbj4_1_1.bcp';
CATALOG DATAFILECOPY '?/oradata/users01.bak' LEVEL 0
CATALOG ARCHIVELOG '?/oradata/archive1_30.dbf', '?/oradata/archive1_31.dbf', 
                   '?/oradata/archive1_32.dbf';
                   
# MOVER OS BACKPSET,ARCHIVELOG,AUTOBACKUP PARA A NOVA FRA
rman target /
RMAN> BACKUP AS COPY ARCHIVELOG ALL DELETE INPUT;

# Não funcionou
RMAN> BACKUP DEVICE TYPE DISK BACKUPSET ALL DELETE INPUT;
RMAN> BACKUP AS COPY DATAFILECOPY DELETE INPUT;
RMAN> BACKUP AS COPY DATAFILECOPY <DATAFILE> DELETE INPUT;

# APARECE O CAMPO IS_RECOVERY_DEST_FILE
select * from V$CONTROLFILE;
select * from V$LOGFILE; 
select * from V$ARCHIVED_LOG;
select * from V$DATAFILE_COPY; 
select * from V$BACKUP_PIECE;

# SÓ PARA QUEM USA FLSHBACK
# alter database flashback off;
# alter database flashback on;

```
