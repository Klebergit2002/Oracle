## PROD-AG (172.24.8.9-IPESAUDE-BD03)

### INFORMAÇÕES GERAIS DO SISTEMA

| HOST                | DESCRIÇÃO                                                    |
| :------------------ | :----------------------------------------------------------- |
| NOME                | IPESAUDE-BD03                                                |
| IP                  | 172.24.8.9                                                   |
| DOMINIO             | ipes.se.gov.br                                               |
| SISTEMA OPERACIONAL | Windows Server 2016 (x64)                                    |
| PROCESSADOR         | Intel(R) Xeon(R) CPU ES-2697 v4 @ 2.30 GHz (10 Processadores) |
| MEMÓRIA             | 60 GB                                                        |
| DISCO (C:)          | Total: 199 GB<br/>Usado: 70 GB<br/>Livre: 129 GB             |
| DISCO (E:)          | Total: 999 GB<br/>Usado: 557 GB<br/>Livre: 442 GB            |
| DISCO (F:)          | Total: 200 GB                                                |



### RESUMO DAS TAREFAS A SEREM EXECUTADAS 

#### ALTERAÇÕES COM O BANCO ON-LINE

- [x] **ALERT.LOG:** Rotacionar de tempos em tempos
- [x] **TASK MANAGER:** Agendar tarefas no Windows
- [x] **DATAFILES:** AUTOEXTEND ON em SAUDE_DAT08.DBF
- [x] **DATAFILES:** AUTOEXTEND ON em SAUDE_IDX09.DBF
- [x] **DATAFILES:** AUTOEXTEND ON em TEMP02.DBF
- [x] **REDOLOGS:** Multiplexar e aumentar o tamanho
- [x] **FAST RECOVERY AREA:** Manutenção e Gerenciamento do crescimento
- [x] **RMAN:** Implementar scripts e configurações dos Backup/Restore
- [x] **RMAN:** Implementar Política de Retenção
- [x] **RMAN:** Iniciar o ciclo dos Backups FULL (Semanal)
- [x] **RMAN:** Iniciar o ciclo dos Backups INCREMENTAIS (Diários)
- [x] **RMAN:** Fazer acompanhamento dos Backups e Política de Retenção
- [ ] **RMAN:** Corrigir possíveis erros

***OBS: Para implementar os Backups, o Banco tem que está em modo ARCHIVELOG***



#### ALTERAÇÕES COM O BANCO OFF-LINE

- [x] **ARCHIVELOGS:** Apagar lixos, configurar path
- [x] **ARCHIVELOGS:** Habilitar somente depois da observação dos REDO LOGS
- [ ] **CONTROFILES:** Se possivel mover o CONTROL03.CTL para outro disco

***OBS: Verificar como seria a geração dos ARCHIVES com as novas alterações dos REDO LOGS***



- [x] Inclusão de mais um disco (F:) de 200 GB para FRA
- [x] Configuração e implementação da FRA no novo disco F:
- [ ] Diminuir o tamanho dos Redologs, se necessário
- [ ] Aumentar a memória SGA e PGA, se necessário - AMM (Automatic Memory Management)



## MÃO NA MASSA

### COLETANDO DADOS

```ini
# MOSTRA AS VARIÁVEIS DE AMBIENTE-SISTEMA
C:\> regedit

# PRINCIPAIS VARIÁVEIS DE AMBIENTE
SET ORACLE_SID=PRODUCAO  -- Não esta no registro do Windows
SET ORACLE_BASE=C:\APP\ADMINISTRATOR\VIRTUAL
SET ORACLE_HOME=C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\12.2.0\dbhome_1

# MOSTRA PARAMETROS DA FRA
show parameters db_recovery;
select * from V$RECOVERY_FILE_DEST;
DB_RECOVERY_FILE_DEST = E:\RECOVERY\PRODUCAO\FLASH_RECOVERY_AREA
DB_RECOVERY_FILE_DEST_SIZE = 200G

# ALTERANDO NOME/CAMINHO/TAMANHO DA FRA
select name from V$RECOVERY_FILE_DEST;
alter system set DB_RECOVERY_FILE_DEST='F:\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=200G;

# MOSTRA O DESTINO DO ARQUIVAMENTO
sqlplus sys/****@producao as sysdba
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

# DATAPUMP
Restrictions
To use this feature, database compatibility must be set to 12.0.0 or later.
This feature requires that the Oracle Advanced Compression option be enabled.

expdp fabio@bd directory=DADOS dumpfile=expdp_fabio_%u.dmp 
   logfile=expdp_fabio.log parallel=4 COMPRESSION=ALL COMPRESSION_ALGORITHM=BASIC
 
```

***OBS: Esse Banco não esta utilizando o padrão OMF***

### ESTRUTURA DE ARMAZENAMENTO

| %ORACLE_BASE%\PRODUCT\ORADATA\PRODUCAO\CONTROLFILES          |
| ------------------------------------------------------------ |
| CONTROL02.CTL                                                |
| **E:\ORADATA\PRODUCAO\CONTROLFILES**                         |
| CONTROL01.CTL<br/>CONTROL03.CTL                              |
| **E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE**              |
| SAUDEPRO_DAT01.DBF<br/>SAUDEPRO_DAT02.DBF<br/>SAUDEPRO_DAT03.DBF<br/>SAUDEPRO_DAT04.DBF<br/>SAUDEPRO_DAT05.DBF<br/>SAUDEPRO_DAT06.DBF<br/>SAUDEPRO_DAT07.DBF<br/>SAUDEPRO_DAT08.DBF<br/>SAUDEPRO_DAT09.DBF<br/><br/>SAUDEPRO_IDX01.DBF<br/>SAUDEPRO_IDX02.DBF<br/>SAUDEPRO_IDX03.DBF<br/>SAUDEPRO_IDX04.DBF<br/>SAUDEPRO_IDX05.DBF<br/>SAUDEPRO_IDX06.DBF<br/>SAUDEPRO_IDX07.DBF<br/>SAUDEPRO_IDX08.DBF<br/>SAUDEPRO_IDX09.DBF<br/><br/>TEMP02.DBF<br/>TS_WORKFLOW01.DBF |
| **E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO**     |
| REDO01.LOG<br/>REDO02.LOG<br/>REDO03.LOG<br/>SYSAUX01.DBF<br/>SYSTEM01.DBF<br/>UNDOTBS01.DBF<br/>USERS01.DBF<br/>TEMP01.DBF |
| **%ORACLE_BASE%**                                            |
| \scripts                                                     |

### ALERT.LOG

```ini
# De tempos em tempos fazer o procedimento abaixo:
# PATH=C:\app\administrator\virtual\diag\rdbms\producao\producao\trace
1.Renomear o arquivo alert_producao.log para alert_producao.log.1
2.Compactar o arquivo alert_producao.log.1
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
2.Agendar Backup Incremental nivel 0 FULL 
3.Agendar Backup Incremental nivel 1 DIARIOS 
4.Email de alerta dos Backups incompletos
5.Email de alerta Tablespace cheios
```

### DATAFILES COM AUTOEXTENDED OFF

```sql
# ALTERAR PARA AUTOEXTEND ON, os seguintes datafiles:
# Logar como sys as sysdba

select * from dba_data_files where autoextensible = 'NO';
# p1 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF' 
# p2 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF'

select * from dba_temp_files where autoextensible = 'NO';
# p3 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\TEMP02.DBF'

alter database datafile p1 autoextend on;
alter database datafile p2 autoextend on;
alter database tempfile p3 autoextend on;
```

### REDO LOGS

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

# CRIAR A PASTA ONLINELOG EM (NÃO FOI CRIADA)
C:\app\administrator\virtual\product\ORADATA\PRODUCAO

# ADICIONANDO NOVOS GRUPOS DE REDOLOGS
alter database add logfile group 1 
('E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO01A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO01B.LOG') size 200M;

alter database add logfile group 2 
('E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO02A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO02B.LOG') size 200M;
                                    
alter database add logfile group 3 
('E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\PRODUCAO\REDO03A.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\REDO03B.LOG') size 200M;

# APAGAR OS GRUPOS ANTIGOS 
# OBS: Apagar somente os grupos com status inactive
select group#, status from v$log;
alter system switch logfile;
alter system checkpoint;
                                    
alter database drop logfile group 4;
alter database drop logfile group 5;
alter database drop logfile group 6;
```

### FAST RECOVERY AREA

```sql
set ORACLE_BASE='C:\APP\ADMINISTRATOR\VIRTUAL'
show parameters db_recovery;

db_recovery_file_dest 'F:\FRA' 
db_recovery_file_dest_size '200G'                                     

select name from V$RECOVERY_FILE_DEST;
alter system set DB_RECOVERY_FILE_DEST='F:\FRA' SCOPE=BOTH;
alter system set DB_RECOVERY_FILE_DEST_SIZE=200G;

# FAZER ACOMPANHAMENTO DO CRESCIMENTO
- Manutenção e limpeza da FRA

show parameter db_recover;
select * from v$flash_recovery_area_usage;
```

### RMAN

```sql
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
backup_full_prod_ag.bat
backup_diario_prod_ag.sql

# ESTRUTURA DE PASTAS PARA OS SCRIPTS E LOG DOS BACKUPS
%ORACLE_BASE%\rman_scripts
%ORACLE_BASE%\rman_scripts\log_backup
```

### ARCHIVELOG

```sql
# APAGAR TODOS OS LOGS DE ARCHIVELOGS ANTIGOS

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

### CONTROLFILES

```sql
# LOCAL ORIGINAL
# CONTROL02.CTL C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\CONTROLFILES\
# CONTROL01.CTL E:\ORADATA\PRODUCAO\CONTROLFILES\
# CONTROL03.CTL E:\ORADATA\PRODUCAO\CONTROLFILES\

# OBS: 
# Caso tenha outro disco, transferir control03.ctl para esse dispositivo
# TRANSFERINDO O CONTROL FILE c3 para novo_c3
SELECT name FROM v$controlfile;
c3 E:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL03.CTL
   
# TRANSFERIR C3 PARA DISCO D:
novo_c3 'D:PATH\CONTROL03.CTL'
ALTER SYSTEM SET CONTROL_FILES=c1,c2,novo_c3 scope=spfile;

shutdown immediate;
exit
mv c3 novo_c3
sqlplus / as sysdba
startup
```

