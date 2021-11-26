

## PROD-ITABS (172.24.8.41-IPESAUDE-ABS01)

### INFORMAÇÕES GERAIS DO SISTEMA

| HOST                | DESCRIÇÃO                                                    |
| ------------------- | ------------------------------------------------------------ |
| NOME                | IPESAUDE-ABS01                                               |
| IP                  | 172.24.8.41                                                  |
| DOMINIO             | ipes.se.gov.br                                               |
| SISTEMA OPERACIONAL | Windows Server 2016 (x64)                                    |
| PROCESSADOR         | Intel(R) Xeon(R) CPU ES-2697 v4 @ 2.30 GHz (4 Processadores) |
| MEMÓRIA             | 8 GB                                                         |
| DISCO (C:)          | Total: 99 GB<br/>Usado: 41 GB<br/>Livre:  58 GB              |
| DISCO (E:)          | Total: 200 GB                                                |



### RESUMO DAS TAREFAS A SEREM EXECUTADAS 

#### ALTERAÇÕES COM O BANCO ON-LINE

- [x] **ALERT.LOG:** Rotacionar de tempos em tempos
- [x] **TASK MANAGER:** Agendar tarefas no Windows
- [x] **DATAFILES:** AUTOEXTEND ON em ITABS_01.DBF
- [ ] **REDOLOGS:** Somente Multiplexar (Não geram muitos ARCHIVESLOGS) 
- [ ] **FAST RECOVERY AREA:** Manutenção e Gerenciamento do crescimento
- [x] **RMAN:** Implementar scripts e configurações dos Backup/Restore
- [x] **RMAN:** Implementar Política de Retenção
- [ ] **RMAN:** Iniciar o ciclo dos Backups FULL (Semanal)
- [ ] **RMAN:** Iniciar o ciclo dos Backups INCREMENTAIS (Diários)
- [ ] **RMAN:** Fazer acompanhamento dos Backups e Política de Retenção
- [ ] **RMAN:** Corrigir possíveis erros

***OBS: Para implementar os Backups, o Banco tem que está em modo ARCHIVELOG***



#### ALTERAÇÕES COM O BANCO OFF-LINE

- [ ] **ARCHIVELOGS:** Apagar lixos, configurar path
- [ ] **ARCHIVELOGS:** Habilitar somente depois da observação dos REDO LOGS
- [ ] **CONTROFILES:** Se possivel coloca-los em discos diferentes

***OBS: Verificar como seria a geração dos ARCHIVES com as novas alterações dos REDO LOGS***



## MÃO NA MASSA

### COLETANDO DADOS

```ini
# MOSTRA AS VIRIÁVEIS DE AMBIENTE-SISTEMA 
C:\> regedit

# PRINCIPAIS VARIÁVEIS 
select * from dba_directories;
SET ORACEL_SID=ITABS
SET ORACLE_BASE=C:\APP\ADMINISTRATOR\VIRTUAL
SET ORACLE_HOME=C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\12.2.0\dbhome_1

# MOSTRA PARAMETROS DA FRA
show parameters db_recovery;
select * from V$RECOVERY_FILE_DEST;
DB_RECOVERY_FILE_DEST = %ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS
DB_RECOVERY_FILE_DEST_SIZE = 9546M

# MOSTRA O DESTINO DO ARQUIVAMENTO
sqlplus as sysdba
archive log list;
archive destination USE_DB_RECOVERY_FILE_DEST
---
select dest_name,destination from v$archive_dest;
LOG_ARCHIVE_DEST_1 = USE_DB_RECOVERY_FILE_DEST

# MOSTRA O DESTINO DOS DATAFILES
show parameters db_create_file_dest;

# VERIFICANDO DATAFILES
select * from v$controlfile;
select * from v$tempfile;
select * from v$dbfile;
select * from v$logfile;

```

### ESTRUTURA DE ARMAZENAMENTO

| \ORADATA\ITABS                                               |
| ------------------------------------------------------------ |
| CONTROL01.CTL<br/>ITABS_01.DBF<br/>REDO01.LOG<br/>REDO02.LOG<br/>REDO03.LOG<br/>SYSAUX01.DBF<br/>SYSTEM01.DBF<br/>TEMP01.DBF<br/>UNDOTBS01.DBF<br/>USERS01.DBF |
| **%ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS**                   |
| CONTROL02.CTL                                                |
| **FALTA DEFINIR A PASTA DOS SCRIPTS**                        |
| \rman_scripts                                                |

### ALERT.LOG

```sql
# De tempos em tempos fazer o procedimento abaixo:
# PATH=C:\app\administrator\virtual\diag\rdbms\producao\producao\trace
1.Renomear o arquivo alert_clinicas.log para alert_itabs.log.1
2.Compactar o arquivo alert_clinicas.log.1
3.Proceder dessa maneira com as proximas ROTAÇÕES: log.2 log.3 ...

# OPCIONAL
# Criar uma tabela ALERTLOG no schema para acessar o log via SELECT 
# no caso do NOTEPAD.EXE não conseguir ler o alert.log
```

### TASK MANAGER

```ini
# TASK MANAGER
C:\> taskschd  

# LISTA DE AGENDAMENTOS
1.Agendar Rotação & Compactação dos ALERTs LOGs quando atingir um determindado VALOR
2.Agendar Backup Incremental nivel 0 FULL (Semanal)
3.Agendar Backup Incremental nivel 1 INC  (Diarios) 
4.Email de alerta dos Backups incompletos
5.Email de alerta Tablespace cheios
```

### DATAFILES

```sql
# ALTERAR PARA AUTOEXTEND ON, os seguintes datafiles:
# Logar como sys as sysdba

set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
select * from dba_data_files where autoextensible = 'NO';
alter database datafile 'C:\ORADATA\ITABS\ITABS_01.DBF' autoextend on;
```

### REDO LOGS

```sql
# VERIFICAR OMF
show parameters db_create_file_dest; # NÃO DEFINIDO
show parameters db_recovery_file_dest;
# DB_RECOVERY_FILE_DEST = %ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS
# DB_RECOVERY_FILE_DEST_SIZE = 9546M

# VERIFICANDO OS GRUPOS DE REDOLOG FILES
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;
---

# SE POSSIVEL COLOCAR GRUPOS EM DISCOS DIFERENTES
# NAO PRECISA AUMENTAR O TAMANHO DEIXAR COM 200M 

set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
GRUPO 1 C:\ORADATA\ITABS\REDO01.LOG
GRUPO 2 C:\ORADATA\ITABS\REDO02.LOG
GRUPO 3 C:\ORADATA\ITABS\REDO03.LOG

# ADICIONAR MEMBROS A UM GRUPO EXISTENTE
ALTER DATABASE ADD LOGFILE MEMBER
'%ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS\ITABS\ONLINELOG\REDO01B.LOG' TO GROUP 1;  
ALTER DATABASE ADD LOGFILE MEMBER
'%ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS\ITABS\ONLINELOG\REDO02B.LOG' TO GROUP 2;  
ALTER DATABASE ADD LOGFILE MEMBER
'%ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS\ITABS\ONLINELOG\REDO03B.LOG' TO GROUP 3;  
```

### FAST RECOVERY AREA

```sql
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
show parameters db_recovery;

select * from V$RECOVERY_FILE_DEST;
db_recovery_file_dest 'E:\FRA'
db_recovery_file_dest_size '200G'

# ALTERANDO A FRA
alter system set DB_RECOVERY_FILE_DEST='E:\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=200G SCOPE=BOTH;

# ACOMPANHAR O CRESCIMENTO
# Manutenção e limpeza da FRA

show parameter db_recover;
select * from v$flash_recovery_area_usage;
```

### RMAN

```sql
$ rman target sys/senhaitabs@itabs

# VARIAVEIS DE AMBIENTE
set ORACLE_SID=ITABS
set ORACLE_BASE=c:\app\Administrador\virtual
set ORACLE_HOME=c:\app\Administrador\virtual\product\12.2.0\dbhome_1

# CONFIGURAÇÕES DE PARAMETROS
CONFIGURE RETENTION POLICY TO REDUNDANCY 1;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;

CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F';

# CONFIGURAÇÃO DOS CAMINHOS DOS BACKUP SET
Por default o backup vai para a FRA

# SCRIPTS DE BACKUPS
backup_full_prod_itabs.bat
backup_diario_prod_i.sql

# ESTRUTURA DE PASTAS PARA OS SCRIPTS E LOG DOS BACKUPS
%ORACLE_BASE%\scripts
%ORACLE_BASE%\scripts\log_backup
```

### ARCHIVELOG

```sql
# APAGAR TODOS OS LOGS DE ARCHIVELOGS ANTIGOS

set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'

# MOSTRA O DESTINO DOS ARCHIVELOGS
sqlplus as sysdba
archive log list;
archive destination USE_DB_RECOVERY_FILE_DEST
---
select dest_name,destination from v$archive_dest;
LOG_ARCHIVE_DEST_1 = USE_DB_RECOVERY_FILE_DEST

# COLOCANDO OS ARCHIVES NA FRA (Ja esta configurado)
alter system set log_archive_dest_1='LOCATION=USE_DB_RECOVERY_FILE_DEST' scope=both;

# ATIVANDO MODO ARCHIVE
shutdown immediate;
startup mount;
archive log list;
alter database archivelog; 
alter database open;
archive log list;
```

### CONTROLFILES

```sql
# LOCAL ORIGINAL 
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'

c1 C:\ORADATA\ITABS\CONTROL01.CTL
c2 %ORACLE_BASE%\FAST_RECOVERY_AREA\ITABS\CONTROL02.CTL

# OBS: 
# Caso tenha outro disco, transferir um dos controls para esse dispositivo

# TRANSFERINDO O CONTROL FILE c1 para novo_c1
SELECT name FROM v$controlfile;
c1 C:\ORADATA\ITABS\CONTROL01.CTL
   
# TRANSFERIR C1 PARA DISCO D:
novo_c1 'D:\ORADATA\ITABS\CONTROL01.CTL'
ALTER SYSTEM SET CONTROL_FILES=novo_c1,c2 scope=spfile;

shutdown immediate;
exit
mv c1 novo_c1
sqlplus / as sysdba
startup
```

#### CRIANDO BANCO ITABS EM OUTRA MAQUINA

```sql
# VER O CHARSET ITABS PRODUÇÃO
select * from NLS_DATABASE_PARAMETERS;

NLS_NCHAR_CHARACTERSET	AL16UTF16
NLS_CHARACTERSET	WE8MSWIN1252
lsnrctl status
rman target /
sqlplus / as sysdba
ipconfig
ping 172.24.8.9
ping 172.24.8.19
ping 172.24.8.41
ping 172.24.9.100/IPESAUDE-BDH03
tnsping ITABS 20

# COPIAR O DUMPFILE
Administrator/senha \\172.24.8.41\e$

select * from dba_segments;

CREATE TABLESPACE "ITABS" DATAFILE 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\ITABS\DATAFILE\ITABS_01.DBF' SIZE 512M AUTOEXTEND ON;
DROP TABLESPACE ITABS INCLUDING CONTENTS AND DATAFILES;

SELECT * FROM dba_directories;
CREATE OR REPLACE DIRECTORY DATA_PUMP AS 'E:\DPUMP';
GRANT READ, WRITE ON DIRECTORY DATA_PUMP TO ITABS; --Nã
show parameters db_create_file_dest;

# PARANDO DATA PUMP
select * from dba_datapump_jobs; --Pega o nome do job
impdp system/senha@instancia attach=nome_do_job
import> kill_job
ou
expdp system/senha@instancia attach=nome_do_job
export> kill_job
C

# ITABS_EXPDP.PAR
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

# ITABS_EXPDP.BAT
cd C:\Backup
del E:\DPDUMP\itabs.dmp
del E:\DPDUMP\itabs_exp.log
call expdp parfile=itabs.par

# ITABS_IMPDP.PAR
USERID=system/oracle12c@itabs
SCHEMAS=CAMPANHAMENSAGENS,PORTALBENEF,GESTAO,AGENDAIPES,GDPORTAL
DUMPFILE=itabs.dmp
LOGFILE=itabs_impdp.log
DIRECTORY=DATA_PUMP
--TABLE_EXISTS_ACTION=[SKIP | APPEND | TRUNCATE | REPLACE]

# DROP EM TODOS OS SCHEMAS
select * from v$instance; --Conferir o host_name
drop user CAMPANHAMENSAGENS cascade;
drop user PORTALBENEF cascade;
drop user GESTAO cascade;
drop user AGENDAIPES cascade;
drop user GDPORTAL cascade;

# Executar do prompt
cd C:\app\Administrador\virtual\dp_scripts
call impdp parfile=itabs_impdp.par

select * from dba_segments where owner in ('CAMPANHAMENSAGENS','PORTALBENEF','GESTAO','AGENDAIPES','GDPORTAL');
select * from dba_objects where owner in ('CAMPANHAMENSAGENS','PORTALBENEF','GESTAO','AGENDAIPES','GDPORTAL') order by created desc;
select owner,count(*) from dba_segments where owner in ('CAMPANHAMENSAGENS','PORTALBENEF','GESTAO','AGENDAIPES','GDPORTAL') GROUP BY ROLLUP(owner);
select tablespace_name,count(*) from dba_segments where owner in ('CAMPANHAMENSAGENS','PORTALBENEF','GESTAO','AGENDAIPES','GDPORTAL') GROUP BY ROLLUP(tablespace_name);

# ITABS_IMPDP.PAR
USERID=system/oracle12c@itabs
SCHEMAS=CAMPANHAMENSAGENS,PORTALBENEF,GESTAO,AGENDAIPES,GDPORTAL
DUMPFILE=itabs.dmp
LOGFILE=itabs_impdp.log
DIRECTORY=DATA_PUMP_DIR
PARALLEL=4
REMAP_TABLESPACE=ITABS:ITABS
REMAP_TABLESPACE=USERS:ITABS
REMAP_TABLESPACE=SYSTEM:ITABS

# CONFERENCIA POR SCHEMA
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

# TODOS OS SCHEMAS
select object_type,count(*) from dba_objects 
 where owner in ('AGENDAIPES','GESTAO','PORTALBENEF','GDPORTAL','CAMPANHAMENSAGENS')
 group by object_type union all
select object_type,count(*) from dba_objects@PROD_ITABS where owner in ('AGENDAIPES','GESTAO','PORTALBENEF','GDPORTAL','CAMPANHAMENSAGENS') 
 group by object_type order by 1;

# POS IMPORTE DATA PUMP
CREATE PUBLIC DATABASE LINK PROD_ITABS
CONNECT TO system IDENTIFIED BY manager$itabs 
USING '172.24.8.41:1521/itabs';

CREATE GLOBAL TEMPORARY TABLE "GESTAO"."TB_MATERIAL_TEMP" (
  "ID" NUMBER(10,0), "MATERIAL" VARCHAR2(200 BYTE), 
  "ID_MARCA" NUMBER(5,0), "SALDO_LOTE" NUMBER(24,3), 
  "SALDO_MATERIAL" NUMBER(24,3)) 
  ON COMMIT DELETE ROWS;
```

