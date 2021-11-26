### STANDBY MANUAL COM ARCHIVE LOGS - NERV

#### Banco de Dados de Produção

```bash
# NOME DA INSTANCIA - PROD
$ ps aux | grep pmon
$ export ORACLE_SID=PROD
$ rman target /

# COLOCAR O BANCO EM MODO ARCHIVELOG
RMAN> shutdown immediate;
RMAN> startup mount;
RMAN> sql 'alter database archivelog';
RMAN> alter database open;
RMAN> backup database;
RMAN> exit;

# COPIAR PARA O BANCO STANDBY
# WINDOWS \\172.24.8.19\c$  # Administrator/senha 

# 1 SPFILE E ORAPWD
# CAMINHO NO WINDOWS
\\172.24.8.19\c$\app\Administrator\virtual\product\12.2.0\dbhome_1\database
PWDclinicas.ora
SPFILECLINICAS.ORA
INITCLINICAS.ORA

scp /u01/app/oracle/product/11.2.0.4/db_1/dbs/spfilePROD.ora standby: /u01/app/oracle/product/11.2.0.4/db_1/dbs/spfilePROD.ora 

# 2 CONTROLFILE E BACKUP FULL/INCREMENTAL
# WINDOWS
\\172.24.8.19\e$\FRA\CLINICAS\AUTOBACKUP\2021_10_07

scp /u01/FRA/PROD/backupset/2014_06_10/control*.bkp standby: /home/oracle 
scp /u01/FRA/PROD/backupset/2014_06_10/backupfull*.bkp standby: /home/oracle

# IR PARA O HELP: Banco de Dados StandBy 
# logo abaixo 
-----------------------------------------

# SIMULAR A GERAÇÃO DE ALGUNS ARCHIVES
$ sqlplus / as sysdba
SQL> alter system switch logfile;
SQL> /
SQL> /
SQL> exit;
$ rman target /
RMAN> list archivelog all;
RMAN> exit;

# COPIAR OS ARCHIVES PARA STANDBY MANTENDO A INTEGRIDADE DO BANCO STANDBY
$ scp /u01/FRA/PROD/archivelog/2014_06_10/* standby:/home/oracle
# VAI PARA O BANCO STANDBY
```

#### Banco de Dados StandBy - Nerv (DATA-GUARD de pobre)

```sql
# SOMENTE O RDBMS INSTALADO SEM INSTANCIA
$ ls $ORACLE_HOME
$ export ORACLE_SID=PROD
$ ps aux | grep pmon

# WINDOWS
oradim -new -sid CLINICAS
oradim -delete -sid CLINICAS
services.msc

# Essa senha é somente para acesso via prompt de comando
# orapwd file=PWDclinicas.ora password=IPESAUDE format=12 force=y entries=10
Copiar o PWDclinicas.ora para o STANDBY

sqlplus / as sysdba
startup nomount PFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITCLINICAS.ORA';
create spfile from pfile;

# CONFERINDO A COPIA DO SPFILE
$ ls -lh $ORACLE_HOME/dbs

$ rman target /
RMAN> startup nomount; # Falhou por que falta criar alguns diretórios do Oracle
$ strings $ORACLE_HOME/dbs # Mostra arquivos imprimiveis do spfile (linux)
$ mkdir -p /u01/app/oracle
$ mkdir -p /u01/app/oracle/admin/PROD/adump
$ mkdir -p /u01/oradata/PROD/
$ mkdir -p /u01/FRA/PROD/
$ rman target /
RMAN> startup nomount;
RMAN> shutdown abort; # So para subir tudo limpinho
RMAN> exit;
$ cd /home/oracle
$ ls -lh # Pegar o nome do controlfile
$ rman target / 
RMAN> startup nomount
RMAN> restore controlfile from '/home/oracle/control.bkp';

# WINDOWS
restore controlfile from 'C:\DBA-Kleber\O1_MF_NCNNF_CONTROLS_1_JP1Y9ZFT_.BKP';
      
RMAN> alter database mount;
RMAN> catalog start with '/home/oracle/';
RMAN> catalog start with 'C:\DBA-Kleber\Clinicas-Standby';

# SCRIPT PARA LIMPAR DO CATALOGO
C:\app\Administrador\virtual\product\12.2.0\dbhome_1\rdbms\admin\prgrmanc.sql
sqlplus sys/nomanager@c
SQL> @?/rdbms/admin/prgrmanc


# REDE: tem que usar o nome completo do mapeamento
catalog start with '\\172.24.8.7\c$\Backups\Bkp_ORACLE\IPESAUDE-BD02\FRA\CLINICAS';
---

select open_mode from v$database;
select status from v$instance;

#restore archivelog from logseq 134837 until logseq 134857;

report schema;
report need backup;
report obsolete;
---
list backup of archivelog all summary;
list backup of controlfile;
---
list backupset;
list backupset of database;
list backupset of archivelog all;

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

???
RMAN> catalog start with 'C:\app\Administrador\virtual\FRA\CLINICAS';
RMAN> catalog recovery area;
-----

Alter Database Rename File 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSTEM_F8XOCL7X_.DBF' to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSTEM_F8XOCL7X_.DBF';

# PATH DIFERENTES 

run {

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSTEM_F8XOCL7X_.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSTEM_F8XOCL7X_.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSAUX_F8XODOKP_.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_SYSAUX_F8XODOKP_.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_UNDOTBS1_F8XOFGS8_.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_UNDOTBS1_F8XOFGS8_.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_USERS_F8XOG7Z4_.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_USERS_F8XOG7Z4_.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_TMP._01.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_TMP._01.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._01.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._01.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_DAT01.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_DAT01.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_IDX01.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_IDX01.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDEHOM_DAT01.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDEHOM_DAT01.DBF';

Set Newname For Datafile 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._02.DBF' To 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._02.DBF';

Set Newname For Datafile  'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_TEMP_F8XOM0OF_.TMP' TO
 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\O1_MF_TEMP_F8XOM0OF_.TMP';

Set Newname For Datafile 
 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_TMP01.DBF' TO
 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_TMP01.DBF';

restore database;
switch datafile all;
switch datafile all;
}

recover database;

#SQL>recover database using backup controlfile until cancel;

recover database;
recover database until time "to_date('18-09-2009 13:00:00','DD-MM-YYYY HH24:MI:SS')";

run {
  set until time '2009-02-02:10:30:00';
  restore database;
  recover database;
}  


# OBS: O banco tem que abrir em read only, senão o proximo 
# recover database não é possível

RMAN> backup validate database;
RMAN> recover database;
RMAN> alter database open read only;

# SIMULAR GERAÇÃO DE ALGUNS ARQUIVES EM PRODUÇÃO
RMAN> catalog start with '/home/oracle/';
RMAN> recover database;
RMAN> sql 'alter database open read only';

# PARA MANTER O BANCO STADNBY ATUALIZADO 
# COPIAR OS ARCHIVES TODOS OS DIAS

shutdown immediate;
startup mount;
catalog start with 'C:\DBA-Kleber\Clinicas-Standby';
restore database preview;
recover database preview;
restore database preview summary;

-----
run {
  restore archivelog all;
  restore archivelog all from sequence 134838 until sequence 13486 preview;
  restore archivelog from SCN 56789;
};

restore archivelog all preview;
restore achivelog from SCN 56789 preview;
restore database preview;


```

