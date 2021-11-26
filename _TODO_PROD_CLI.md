## PROD-CLINICAS (172.24.8.19-IPESAUDE-BD02)

### INFORMAÇÕES GERAIS DO SISTEMA

| HOST                | DESCRIÇÃO                                                    |
| ------------------- | ------------------------------------------------------------ |
| NOME                | IPESAUDE-BD02                                                |
| IP                  | 172.24.8.19                                                  |
| DOMINIO             | ipes.se.gov.br                                               |
| SISTEMA OPERACIONAL | Windows Server 2016 (x64)                                    |
| PROCESSADOR         | Intel(R) Xeon(R) CPU ES-2697 v4 @ 2.30 GHz (8 Processadores) |
| MEMÓRIA             | 16 GB                                                        |
| DISCO (C:)          | Total: 199 GB<br/>Usado: 90 GB<br/>Livre: 109 GB             |
| DISCO(E:)           | Total: 200 GB                                                |



### RESUMO DAS TAREFAS A SEREM EXECUTADAS 

#### ALTERAÇÕES COM O BANCO ON-LINE

- [x] **ALERT.LOG:** Rotacionar de tempos em tempos
- [x] **TASK MANAGER:** Agendar tarefas no Windows
- [x] **DATAFILES:** AUTOEXTEND ON em IPESAUDEHOM_DAT01.DBF
- [x] **DATAFILES:** AUTOEXTEND ON em PORTAL_DAT01.DBF
- [x] **DATAFILES:** AUTOEXTEND ON em PORTAL_IDX01.DBF
- [x] **TEMPFILES:** AUTOEXTENDED ON em TEMP01.DBF
- [x] **REDOLOGS:** Aumentar o tamanho e coloca-los em grupos distintos
- [x] **FAST RECOVERY AREA:** Manutenção e Gerenciamento do crescimento
- [x] **RMAN:** Implementar scripts e configurações dos Backup/Restore
- [x] **RMAN:** Implementar Política de Retenção
- [x] **RMAN:** Iniciar o ciclo dos Backups FULL (Semanal)
- [x] **RMAN:** Iniciar o ciclo dos Backups INCREMENTAIS (Diários)
- [x] **RMAN:** Fazer acompanhamento dos Backups e Política de Retenção
- [x] **RMAN:** Corrigir possíveis erros

***OBS: Para implementar os Backups, o Banco tem que está em modo ARCHIVELOG***



#### ALTERAÇÕES COM O BANCO OFF-LINE

- [x] **ARCHIVELOGS:** Apagar lixos, configurar path
- [x] **ARCHIVELOGS:** Habilitar somente depois da observação dos REDO LOGS
- [x] **CONTROFILES:** Se possivel coloca-los em discos diferentes
- [ ] **MEMORIA:** Aumentar para 50% do total do servidor

***OBS: Verificar como seria a geração dos ARCHIVES com as novas alterações dos REDO LOGS***



## MÃO NA MASSA

### COLETANDO DADOS

```ini
# MOSTRA AS VIRIÁVEIS DE AMBIENTE-SISTEMA 
C:\> regedit

# PRINCIPAIS VARIÁVES DE SISTEMA
SET ORACLE_SID=CLINICAS
SET ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
SET ORACLE_HOME=C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\12.2.0\dbhome_1

# MOSTRA PARAMETROS DA FRA
show parameters db_recovery;
select * from V$RECOVERY_FILE_DEST;
DB_RECOVERY_FILE_DEST = %ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS
#DB_RECOVERY_FILE_DEST_SIZE = 9546M

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

| %ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS\CLINICAS           |
| ------------------------------------------------------------ |
| **\ONLINELOG**<br/>O1_MF_1_F8XOLQC2-.LOG<br/>O1_MF_2_F8XOLPMV-.LOG<br/>O1_MF_3_F8XOLPMC-.LOG<br/><br/>**\CONTROLFILE**<br/> O1_MF_F8XOLL82-CTL |
| **%ORACLE_BASE%\ORADATA\CLINICAS**                           |
| **\CONTROLFILE**<br/>O1_MF_F8XOLL7MIPESAUDE_DAT.-01.DBF<br/><br/>**\DATAFILE**<br/>IPESAUDE_DAT.-02.DBF<br/>IPESAUDEHOM_DAT01.DBF<br/>IPESAUDE_TMP.-01.DBF<br/>O1_MF_SYSAUX_F8XODOKP-.DBF<br/>O1_MF_SYSTEM_F8XOCL7X-.DBF<br/>O1_MF_TEMP_F8XOM0OF-.TMP<br/>O1_MF_UNDOTBS1_F8XOFGS8-.DBF<br/>O1_MF_USERS_F8XOG7Z4-.DBF<br/>PORTAL_DAT01.DBF<br/>PORTAL_IDX01.DBF<br/>PORTAL_TMP01.DBF<br/><br/>**\ONLINELOG**<br/>O1_MF_1_F8XOLOVO-.LOG<br/>O1_MF_2_F8XOLOVO-.LOG<br/>O1_MF_3_F8XOLOVO-.LOG |
| **%ORACLE_BASE%**                                            |
| \scripts                                                     |

### ALERT.LOG

```sql
# De tempos em tempos fazer o procedimento abaixo:
# PATH=C:\app\administrator\virtual\diag\rdbms\producao\producao\trace
1.Renomear o arquivo alert_clinicas.log para alert_clinicas.log.1
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

select * from dba_data_files where autoextensible = 'NO';
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
p1 %ORACLE_BASE%\ORADATA\CLINICAS\DATAFILE\IPESAUDEHOM_DAT01.DBF
p2 %ORACLE_BASE%\ORADATA\CLINICAS\DATAFILE\PORTAL_DAT01.DBF	    
p3 %ORACLE_BASE%\ORADATA\CLINICAS\DATAFILE\PORTAL_IDX01.DBF

alter database datafile p1 autoextend on;
alter database datafile p2 autoextend on;
alter database datafile p3 autoextend on;
```

### TEMPFILES

```sql
select * from dba_temp_files where autoextensible = 'NO';
p4 %ORACLE_BASE%\ORADATA\CLINICAS\DATAFILE\PORTAL_TMP01.DBF
alter database tempfile p4 autoextend on;
```

### REDO LOGS

```sql
# VERIFICAR OMF
show parameters 
db_create_file_dest;   # C:\app\Administrator\virtual\oradata
show parameters 
db_recovery_file_dest; # C:\app\Administrator\virtual\fast_recovery_area\clinicas
     
# VERIFICANDO OS GRUPOS DE REDOLOG FILES
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;
---
# SE POSSIVEL COLOCAR GRUPOS EM DISCOS DIFERENTES
# AUMENTAR O TAMANHO PARA 400M

set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
# GRUPO 1
%ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS\CLINICAS\ONLINELOG\O1_MF_1_F8XOLQC2_.LOG
%ORACLE_BASE%\ORADATA\CLINICAS\ONLINELOG\O1_MF_1_F8XOLOVO_.LOG

# GRUPO 2
%ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS\CLINICAS\ONLINELOG\O1_MF_2_F8XOLPMV_.LOG
%ORACLE_BASE%\ORADATA\CLINICAS\ONLINELOG\O1_MF_2_F8XOLOVO_.LOG

# GRUPO 3
%ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS\CLINICAS\ONLINELOG\O1_MF_3_F8XOLPMC_.LOG
%ORACLE_BASE%\ORADATA\CLINICAS\ONLINELOG\O1_MF_3_F8XOLOVO_.LOG

# ADICIONANDO OS NOVOS GRUPOS DE REDOLOGS
# COM OMF
ALTER DATABASE ADD LOGFILE GROUP 1 SIZE 400M;
ALTER DATABASE ADD LOGFILE GROUP 2 SIZE 400M;
ALTER DATABASE ADD LOGFILE GROUP 3 SIZE 400M;

# ADICIONANDO NOVOS GRUPOS DE REDOLOGS (SEM OMF)
alter database add logfile group 4 
('C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\REDO01A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\REDO01B.LOG') size 100M;

alter database add logfile group 5 
('C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\REDO02A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\REDO02B.LOG') size 100M;
                                    
alter database add logfile group 6 
('C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\REDO03A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\REDO03B.LOG') size 100M;

# DROPANDO REDO LOGS ANTIGOS
select group#, bytes/1024/1024 as MB, members, status from v$log ;

# Sempre verificar o status dos grupos 
# Se estiverem CURRENT (recebendo dados) ou 
# ACTIVE (gravando os dados nos archivelogs) não poderão ser deletados.

# APAGAR SOMENTE OS GRUPOS COM STATUS INACITVE
alter system switch logfile;
alter system checkpoint;

ALTER DATABASE DROP LOGFILE GROUP 1; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 2; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 3; --Inactive
```

### FAST RECOVERY AREA

```sql
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
show parameters db_recovery;
db_recovery_file_dest 'C:\app\Administrator\virtual\fast_recovery_area\clinicas '
db_recovery_file_dest_size '9546M'                                                   

select name from V$RECOVERY_FILE_DEST;
db_recovery_file_dest 'E:\FRA' 
db_recovery_file_dest_size '200G'                                     

alter system set DB_RECOVERY_FILE_DEST='E:\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=200G;

# FAZER ACOMPANHAMENTO DO CRESCIMENTO
#Manutenção e limpeza da FRA
show parameter db_recover;
select * from v$flash_recovery_area_usage;
```

### RMAN

```sql
# VARIAVEIS DE AMBIENTE
set ORACLE_SID=CLINICAS
set ORACLE_BASE=c:\app\Administrador\virtual
set ORACLE_HOME=c:\app\Administrador\virtual\product\12.2.0\dbhome_1

# CONFIGURAÇÕES DE PARAMETROS
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO BACKUPSET;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F';


# CONFIGURAÇÃO DOS CAMINHOS DOS BACKUP SET
Por default o backup vai para a FRA

# SCRIPTS DE BACKUPS
backup_full_prod_cli.bat
backup_diario_prod_cli.sql

# ESTRUTURA DE PASTAS PARA OS SCRIPTS E LOG DOS BACKUPS
%ORACLE_BASE%\scripts
%ORACLE_BASE%\scripts\log_backup
```

### ARCHIVELOG

```sql
# APAGAR TODOS OS LOGS DE ARCHIVELOGS ANTIGOS

# SETANDO O CAMINHO DOS ARCHIVELOGS
# Criar a pasta 'archivelogs' em C:\app\administrator\virtual (POUCO ESPAÇO)
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
show parameters log_archive_dest;
db_recovery_file_dest '%ORACLE_BASE%\fast_recovery_area\clinicas'

# DEFININDO O CAMINHO DOS ARCHIVES (Não sera usado)
# alter system set log_archive_dest_1='LOCATION=%ORACLE_BASE%\archivelogs' scope=spfile;

# COLOCANDO OS ARCHIVES NA FRA (Sera usado esse)
alter system set log_archive_dest_1='LOCATION=USE_DB_RECOVERY_FILE_DEST' scope=both;

# ATIVANDO MODO ARCHIVE
shutdown immediate;
startup mount;
archive log list;
alter database archivelog; 
alter database open;
archive log list;
alter system switch logfile;
```

### CONTROLFILES

```sql
# LOCAL ORIGINAL 
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
c1 %ORACLE_BASE%\ORADATA\CLINICAS\CONTROLFILE\O1_MF_F8XOLL7M_.CTL
c2 %ORACLE_BASE%\FAST_RECOVERY_AREA\CLINICAS\CLINICAS\CONTROLFILE\O1_MF_F8XOLL82_.CTL

# OBS: 
# Caso tenha outro disco, transferir um dos controls para esse dispositivo

# TRANSFERINDO O CONTROL FILE c1 para novo_c1
SELECT name FROM v$controlfile;
c1 %ORACLE_BASE%\ORADATA\CLINICAS\CONTROLFILE\O1_MF_F8XOLL7M_.CTL
   
# TRANSFERIR C1 PARA DISCO D:
novo_c1 'D:\ORADATA\CLINICAS\CONTROLFILES\O1_MF_F8XOLL7M_.CTL'
ALTER SYSTEM SET CONTROL_FILES=novo_c1,c2 scope=spfile;

shutdown immediate;
exit
mv c1 novo_c1
sqlplus / as sysdba
startup
```

### A FAZER EM CLINICAS HOMOLOGAÇÃO - 13/08/2021

```sql
# RESUMO DAS TAFERAS EM CLINICAS-PRODUÇÃO (172.24.8.19) 16/08/
 1. Autoextensible que faltam em alguns Datafiles
 2. Configurar variáves OMF
 3. Criação de uma nova FRA
 4. Criação de novos grupos de redo sendo um na FRA
 5. Aumentar o tamanho dos redologs
 *6. Aumentar a memoria SGA e PGA 
 *7. Mover um Controlfile para a nova FRA
 8. Habilitar o modo Archive
 9. Fazer o primeiro backup FULL com o RMAN
10. Acompanhamento diario dos backups FULL/INCREMENTAIS na FRA
11. Monitorar o tamanha da FRA e possíveis correções

---

--PRINCIPAIS VARIÁVEIS DE AMBIENTE
SET ORACLE_SID=PRODUCAO
SET ORACLE_BASE=C:\APP\ADMINISTRATOR\VIRTUAL
SET ORACLE_HOME=C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\12.2.0\dbhome_1

create pfile from spfile;

select * from v$session where osuser <> 'OracleServiceCLINICAS';
select * from v$database;
select * from v$version;

--TAMANHO DO BANCO
select tablespace_name,to_char(sum(bytes)/power(1024,3),'999G999D999') TAMANHO_GB 
  from dba_segments group by rollup(tablespace_name);
  
--ALTERAÇOES EM CLINICAS PRODUÇAO
--Autoextensible=yes
select * from dba_data_files where autoextensible = 'NO';
select * from dba_temp_files where autoextensible = 'NO';
---
--VERIFICANDO OMF
show parameters db_create_file_dest;
show parameters db_recovery_file_dest;
show parameters spfile;

--VERIFICANDO DATAFILES 
select name from v$controlfile union all
select member as name from v$logfile union all
select name from v$tempfile union all
select name from v$dbfile order by 1;
--
select * from v$controlfile;
select * from v$logfile order by member;
select * from v$tempfile;
select * from v$dbfile;

--CONFIGURAÇOES DA FRA
show parameter db_recover;
select * from v$flash_recovery_area_usage;
select * from v$recovery_file_dest;
---
show parameter db_create;
show parameter control_file;
select * from v$parameter where name = 'control_files';
select * from v$parameter where name like '%db_create%';

alter system set DB_RECOVERY_FILE_DEST_SIZE=200G;
alter system set DB_RECOVERY_FILE_DEST='E:\FRA' SCOPE=BOTH;

--HABILITANDO ARCHIVELOG
show parameters log_archive_dest;
archive log list; --USE_DB_RECOVERY_FILE_DEST
select dest_name,destination from v$archive_dest;
---
shutdown immediate;
startup mount;
archive log list;
alter database archivelog; --noarchivelog
alter database open;

--Gerar os primeiros archivelogs 
alter system switch logfile;
select archiver from v$instance;

--CRIANDO NOVOS GRUPOS DE REDO - OMF
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;

alter database add logfile group 4 size 400M;
alter database add logfile group 5 size 400M;
alter database add logfile group 6 size 400M;

alter system switch logfile;
alter system checkpoint;

select group#, bytes/1024/1024 as MB, members, status from v$log ;
ALTER DATABASE DROP LOGFILE GROUP 1; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 2; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 3; --Inactive

--APARECE O CAMPO IS_RECOVERY_DEST_FILE
select * from V$CONTROLFILE;
select * from V$LOGFILE; 
select * from V$ARCHIVED_LOG;
select * from V$DATAFILE_COPY; 
select * from V$BACKUP_PIECE;

--MOVENDO O CONTROLFILE DA FRA ANTIGA PARA A NOVA
SELECT * FROM v$controlfile;
ALTER SYSTEM SET CONTROL_FILES='C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\CONTROLFILE\O1_MF_F8XOLL7M_.CTL',
'E:\FRA\CLINICAS\CONTROLFILE\O1_MF_F8XOLL82_.CTL' scope=spfile;

# VERIFICAR DEU ERRO DE LEITURA EM FRA
--'C:\APP\ADMINISTRATOR\VIRTUAL\FRA\CLINICAS\CONTROLFILE\O1_MF_F8XOLL82_.CTL' scope=spfile;

--CONFIGURAÇOES DE MEMORIA (TOTAL DO HOST=16G)
select * from v$parameter where name like '%sga%';
select * from v$parameter where name like '%pga%';
show sga;
show parameter target;
SELECT * FROM V$MEMORY_TARGET_ADVICE;
select * from v$memory_target_advice order by memory_size;
---
alter system set sga_target = 8G scope=spfile; 
alter system set sga_max_size = 8G scope=spfile; 
--
alter system set pga_aggregate_target = 3G scope=spfile; 
alter system set pga_aggregate_limit = 6G scope=spfile; --Setar com o dobro de pga_aggregate_target
---   
alter system set memory_max_target = 0 scope=spfile;
alter system set memory_target = 0 scope=spfile;

```

