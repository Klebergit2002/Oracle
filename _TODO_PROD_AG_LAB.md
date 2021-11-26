### PROD-AG-LAB (172.24.8.171)

##### RESUMO

```ini
# OBS: Esse Banco não esta utilizando o padrão OMF
ALERT.LOG: Rotacionar de tempos em tempos
TAREFAS: Agendamento de tarefas no Windows
CONTROLFILES: Se possivel mover o control03.ctl para outro disco
DATAFILES: SAUDEPRO_DAT08.DBF e SAUDEPRO_IDX09.DBF alterar para autoextend on
TEMPFILES (TEMP02.DBF): Alterar para autoextend on
REDO LOG: Multiplexar e aumentar o tamanho

# Observar como seria a geração dos ARCH com os novos parametros do REDO LOGs  
ARCHIVELOGS: Apagar lixos, configurar path
ARCHIVELOGS: Habilitar somente depois da observação dos REDO LOGs
```

##### ROTACIONAR ALERT.LOG

```ini
# De tempos em tempos fazer o procedimento abaixo:
# PATH=C:\app\administrator\virtual\diag\rdbms\producao\producao\trace
1.Renomear o arquivo alert_producao.log para alert_producao.log.1
2.Compactar o arquivo alert_producao.log.1
3.Proceder dessa maneira com as proximas ROTAÇÕES: log.2 log.3 ...

# Criar uma tabela ALERTLOG no schema para acessar o log via SELECT 
# no caso do NOTEPAD.EXE não conseguir ler o alert.log

# VERIFICAR O ERRO
Errors in file C:\APP\ADMINISTRATOR\VIRTUAL\diag\rdbms\producao\producao\trace\producao_j000_10900.trc:
ORA-12012: error on auto execute of job "SYS"."ORA$AT_OS_OPT_SY_33435"
ORA-20001: Statistics Advisor: Invalid task name for the current user
ORA-06512: at "SYS.DBMS_STATS", line 47207
ORA-06512: at "SYS.DBMS_STATS_ADVISOR", line 882
ORA-06512: at "SYS.DBMS_STATS_INTERNAL", line 20059
ORA-06512: at "SYS.DBMS_STATS_INTERNAL", line 22201
ORA-06512: at "SYS.DBMS_STATS", line 47197
2021-05-10T02:00:00.157789-03:00
```

##### AGENDAMENTO DE TAREFAS - WINDOWS

```ini
# TASK MANAGER
taskschd  

# LISTA DE AGENDAMENTOS
1.Agendar Rotação & Compactação dos ALERTs LOGs quando atingir um determindado VALOR
2.Agendar Backup Incremental nivel 0 FULL # OK
3.Agendar Backup Incremental nivel 1 DIARIOS # OK
4.Email de alerta dos Backups incompletos
5.Email de alerta Tablespace cheios
```

##### MOVENDO A FRA

```sql
# MAPEAR OS LOCAIS DOS SEGUNTES DATAFILES
# ORACLE_SID=PRODUCAO
# ORACLE_BASE=C:\APP\ADMINISTRATOR\VIRTUAL

# CONTROL FILES (ORIGINAL)
# %ORACLE_BASE%\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\CONTROLFILE\
# %ORACLE_BASE%\ORADATA\PRODUCAO\CONTROLFILE\

# ARQUIVOS DE DADOS (ORIGINAL)
# %ORACLE_BASE%\ORADATA\PRODUCAO\DATAFILE

# REDOLOGS (ORIGINAL)
# %ORACLE_BASE%\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG
# %ORACLE_BASE%\ORADATA\PRODUCAO\ONLINELOG

# ARCHIVELOGS (ORIGINAL)
# %ORACLE_BASE%\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ARCHIVELOG

---

# CONFIRMANDO SE ESTA NO BANCO CORRETO
ipconfig
set ORACLE_SID=PRODUCAO
sqlplus / as sysdba
select DBID from v$database;

# CRIAR A NOVA PASTA "FRA" COM AS PERMISSÕES NECESSÁRIAS
# sqldeveloper
show parameters db_recovery;
select name from V$RECOVERY_FILE_DEST;
alter system set DB_RECOVERY_FILE_DEST='C:\app\Administrator\virtual\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=750G;

# REINICIAR A INSTANCIA NO MODO MOUNT
sqlplus / as sysdba
shutdown immediate;
startup mount;
select * from v$restore_point;
drop restore point <NOME> se existir
alter database flashback off; # PARA RECONHECER A NOVA FRA
alter database flashback on;  # PARA RECONHECER A NOVA FRA
alter database open;

# MOVER ARQUIVOS (BACKUPSET,ARCHIVELOGS,AUTOBACKUP)
rman target /
BACKUP AS COPY ARCHIVELOG ALL DELETE INPUT;
BACKUP DEVICE TYPE DISK BACKUPSET ALL DELETE INPUT;

# MOVER DATAFILECOPY PARA CADA ARQUIVO
BACKUP AS COPY DATAFILECOPY <DATFILE> DELETE INPUT;

# OPCIONAL-MOVER ARQUIVOS (BACKUPSET,ARCHIVELOGS,AUTOBACKUP)
set serveroutput on; 
declare   
cursor dfc is select name from v$datafile_copy
              where status = 'A'
              and is_recovery_dest_file = 'YES';
 begin   
 dbms_output.put_line('run');   
 dbms_output.put_line('{');     
 dbms_output.put_line('backup as copy archivelog all delete input;');
 dbms_output.put_line('backup device type disk backupset all delete  input;');
 for dfcrec in dfc loop     dbms_output.put_line('backup as copy datafilecopy ''' ||
            dfcrec.name || '''delete input;');   
 end loop;   
 dbms_output.put_line('}'); 
end;
/ 

# MOVER CONTROLFILE DA VELHA FRA (NOMOUNT)
shutdown immediate;
startup nomount;
# RESTORE CONTROLFILE FROM 'filename_of_old_control_file';
RESTORE CONTROLFILE FROM 'C:\app\Administrator\virtual\fast_recovery_area\producao
\producao\controlfile\O1_MF_HP6Y9YXG_.CTL'
alter database open;

# MOVER OS REDOLOGS
# ADD NOVOS GRUPOS REDOLOGS NA NOVA FRA PARA CADA GRUPO 
# DEPOIS APAGAR OS ANTIGOS: OBSERVAR O STATUS INACTIVE
SQL> alter database add logfile size 300M;
SQL> alter database drop logfile 'nameof the old redo log';

---
# RECATALOGAR
RMAN> CATALOG START WITH '/usr4/oracle/fast_recovery_area'; OU
RMAN> CATALOG RECOVERY AREA [NOPROMPT]; OU
RMAN> CATALOG DB_RECOVERY_FILE_DEST [NOPROMPT];
---
RMAN> CROSSCHECK BACKUP;
RMAN> CROSSCHECK ARCHIVELOG ALL;
RMAN> DELETE EXPIRED BACKUP;
RMAN> DELETE EXPIRED ARCHIVELOG ALL;

```

##### CONTROLFILES & SPFILE

```sql
# LOCAL ORIGINAL
# C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\CONTROLFILE\
# C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\CONTROLFILE\
# OBS: 
# Caso tenha outro disco, transferir control03.ctl para esse dispositivo
# Vide Controlfiles.md

# SPFILE & PFILE
create pfile from spfile (mais usado)
create spfile from pfile (cuidado p/não desatualizar o spfile)
```

##### ARQUIVOS DE DADOS & TEMP

```sql
# ALTERAR PARA AUTOEXTEND ON, os seguintes datafiles:
# Logar como sys as sysdba

# p1 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF' 
# p2 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF'
alter database datafile p1 autoextend on;
alter database datafile p2 autoextend on;

# p3 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\TEMP02.DBF'	
alter database tempfile p3 autoextend on;
```

##### GRUPOS DE REDOLOGS

```sql
# MULTIPLEXAR OS REDOLOGS
# TAMANHO: 200 MB
# ARQUIVOS ATUAIS
# E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO01.LOG
# E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO02.LOG
# E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO03.LOG
---
# CRIAR NOVOS GRUPOS
# CRIAR OS NOVOS GRUPOS COM 400MB E APAGAR OS ANTIGOS
# TAMANHO: 400 MB

# VISUALIZAR OS GRUPOS E DATAFILES DE REDO LOG
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;

# CRIAR A PASTA ONLINELOG EM 
C:\app\administrator\virtual\product\ORADATA\PRODUCAO

# ADICIONANDO NOVOS GRUPOS DE REDOLOGS
alter database add logfile group 1 
('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO01A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO01B.LOG') size 200M;

alter database add logfile group 2 
('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO02A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO02B.LOG') size 200M;
                                    
alter database add logfile group 3 
('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO03A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO03B.LOG') size 200M;
                                          
# APAGAR OS GRUPOS ANTIGOS 
# OBS: Apagar somente os grupos com status inactive
select group#, status from v$log;
alter system switch logfile;
alter system checkpoint;
                                    
alter database drop logfile group 4;
alter database drop logfile group 5;
alter database drop logfile group 6;
```

##### ARCHIVELOG

```sql
# APAGAR TODOS OS LOGS DE ARCHIVELOS ANTIGOS

# SETANDO O CAMINHO DOS ARCHIVELOGS
# Criar a pasta 'archivelogs' em C:\app\administrator\virtual (POUCO ESPAÇO)
show parameters log_archive_dest;
alter system set log_archive_dest_1='LOCATION=C:\app\administrator\virtual\archivelogs' scope=spfile;

# ATIVANDO MODO ARCHIVE
shutdown immediate;
startup mount;
archive log list;
alter database archivelog; 
alter database open;
archive log list;
```

##### RMAN

```sql
# VARIAVEIS DE AMBIENTE
set ORACLE_SID=PRODUCAO
set ORACLE_HOME=c:\app\Administrador\virtual\product\12.2.0\dbhome_1
set ORACLE_BASE=c:\app\Administrador\virtual

# CONFIGURAÇÕES DE PARAMETROS
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F';

# CONFIGURAÇÃO DOS CAMINHOS DOS BACKUP SET

# SCRIPTS DE BACKUPS
backup_full_prod_ag.bat
backup_diario_prod_ag.sql

# ESTRUTURA DE PASTAS PARA OS SCRIPTS E LOG DOS BACKUPS
%ORACLE_BASE%\scripts
%ORACLE_BASE%\scripts\log_backup
```

##### TABLESPACES (08/05/2021)

```ini
# NAME			ALLOC	FREE	USED	%F	%U	MAX
TS_WORKFLOW		662		140		522		21	79	32767
USERS			684		669		15		98	2	32768
SYSTEM			1240	107		1133	9	91	32768
SYSAUX			4870	275		4595	6	94	32768
TEMP2			10000	10000	0		100	0	0
UNDOTBS1		27270	25844	1426	95	5	32768
TEMP			32767	32759	8		100	0	32768
SAUDEPRO_IDX	100158	4558	95600	5	95	282616
SAUDEPRO_DAT	167220	3335	163885	2	98	282616
```

