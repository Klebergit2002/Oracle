##### PROBLEMA COM INDICES

```sql
ANALYZE INDEX IDX_GAM_ACCT VALIDATE STRUCTURE;
SELECT name, height,lf_rows,lf_blks,del_lf_rows FROM INDEX_STATS;
OBS: 
Se height > 4 REBUILD 
Se del_lf_row > 20% REBUILD (Linhas de folhas excluidas)

ALTER INDEX ... COALESCE
ALTER INDEX ... REBUILD OU REBUILD ONLINE (USA LOCK)
---
SELECT COMM
FROM(
SELECT 1 AS ID,
ROWNUM AS ID2,
'ANALYZE INDEX ' ||INDEX_NAME || ' VALIDATE STRUCTURE ; ' AS COMM
FROM DBA_INDEXES
WHERE OWNER = '&user'

```

##### PROBLEMA COM SENHA EXPIRADA

```sql
# USER SSERVER DE HOMO-AG
select * from dba_users;
ALTER USER SSERVER IDENTIFIED BY "ssvrus10g!" ACCOUNT UNLOCK; (FUNCIONOU)

ALTER USER SSERVER IDENTIFIED BY "ssvrus10g!"
ALTER USER SSERVER ACCOUNT UNLOCK;
ALTER USER SSERVER PASSWORD EXPIRE;
```

##### VERIFICANDO ERROS/ALERTAS

```sql
adrci
show alert
```

##### PROBLEMA COM TEMPFILES

```sql
shutdown immediate
startup mount
alter database tempfile 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF' offline;
alter database tempfile 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF' drop including datafiles;
alter database open;

DROP TABLESPACE temp2 INCLUDING CONTENTS AND DATAFILES;
CREATE TEMPORARY TABLESPACE TEMP TEMPFILE 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF' SIZE 10485760000; 

---

Alter Database Rename File 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\TEMP02.DBF' 
   to 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF';
alter database tempfile 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\PRODUCAO\DATAFILE\TEMP02.DBF' online;
----

# RESOLVIDO
ORA-01157 
ORA-01110:202 do arquivo de dados:
'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_TMP01.DBF'
alter system check datafiles;
----
SELECT tablespace_name, file_name, bytes/1024/1024 FROM dba_temp_files WHERE tablespace_name = 'TEMP';

DROP TABLESPACE PORTAL_TMP INCLUDING CONTENTS AND DATAFILES;

CREATE TEMPORARY TABLESPACE PORTAL_TMP TEMPFILE 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\PORTAL_TMP01.DBF' SIZE 512 M REUSE AUTOEXTEND ON; 

```

##### SINCRONIZANDO O BACKUP

```sql
catalog start with 'C:\app\Administrador\virtual\fast_recovery_area\producao\producao';
crosscheck backup;
crosscheck copy;
crosscheck backup of controlfile;
crosscheck archivelog all;

- delete expired backup;
- delete obsolete device type disk;

list backup;

report schema;
report need backup;
report obsolete;

show controlfile autobackup;
```

##### PROBLEMA COM O LISTENER

```sql
# Alterar os arquivos tnsnames.ora, listener.ora

ERROR: ORA-12505:
Solução:
Verificar a variavel TNS_ADMIN=C:\app\Administrador\virtual\product\12.2.0\dbhome_1\network\admin
e/ou
Colocar o nome da maquina no arquivo %ORALCE_HOME%\network\admin\listener.ora
HOST = AGINF-PC-167591

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = AGINF-PC-167591)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )


ERROR: ORA-12560: TNS:erro de adaptador de protocolo
Solução:
Servicos do windows
Iniciar OracleServicePRODUCAO
sqlplus / as sysdba
startup;

---

show parameters local_listener
alter system set LOCAL_LISTENER='(ADDRESS=(PROTOCOL=TCP)(HOST=AGINF-PC-167591)(PORT=1521))'scope=both;
alter system set LOCAL_LISTENER='(ADDRESS=(PROTOCOL=TCP)(HOST=172.24.9.12467591)(PORT=1521))'scope=both;
# AGINF-PC-167591 (172.24.8.171 172.24.9.124
---
lsnrctl status -> ver o caminho do tnsnames.ora
- tnsnames.ora
- LISTENER_PRODUCAO =
-  (ADDRESS = (PROTOCOL = TCP)(HOST = AGINF-PC-167591)(PORT = 1521))

ipconfig
lsnrctl status
lsnrctl stop
lsnrctl start
tnsping PRODUCAO 20
tnsping 172.24.8 20

#ORA-01078: failure in processing system parameters
# ERRO O/S-Error: (OS 5) Access is denied
Right click on service
Select 'properties'
Select 'logon'
Change the default user ID to an Oracle user with Windows administrator privileges

```

##### MODOS DE CRIAÇÃO DE PFILE E SPFILE

```sql
# Tem que ser via sqlplus
set ORACLE_SID=PRODUCAO
sqlplus / as sysdba;

startup PFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITPRODUCAO.ORA';

CREATE SPFILE FROM MEMORY;
CREATE PFILE FROM MEMORY;
---
create spfile from pfile; # Cuidado pra nao perder o spfile atual
create pfile from spfile; # Mais comum

create pfile='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\dbs\pfile_producao.ora' from memory;

# Agora podemos tambem criar o PFILE como SPFILE informando inclusive a pasta
C:\APP\ADMINISTRADOR\VIRTUAL\PRODUCT\12.2.0\DBHOME_1\DATABASE

create PFILE from  SPFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\spfileclinicas.ora';

create PFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\pfile_clinicas.ora' from  SPFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\spfileclinicas.ora';
                
create  SPFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\spfile_producao.ora' from  PFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\dbs\pfile_producao.ora';

# Mas se o banco tentar iniciar e aparecer algo como :
LRM-00109: could not open parameter file '/u01/app/oracle/product/11.2.0.4/db_1/dbs/init<<NOME SEU BANCO>>.ora'

# A solução é abrir o banco forçando a utilização de algum arquivo com a configuracao tipo:
startup PFILE='C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\initproducao.ora';

SHOW PARAMETER spfile;
select dbid,name from v$database;

# CAMINHO DO ALERT.LOG
C:\app\Administrador\virtual\diag\rdbms\producao\producao\trace

```

##### RECUPERANDO O SPFILE

```sql
# ESSE PROCESSO DE COPIA E EDIÇÃO FUNCIONOU
- Copiar o spfile para init.ora
- Editar o init.ora tirar todos os caracteres que não são textos puros
- Salvar o init.ora

startup pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\init.ora';

show parameters control;
show parameters instance; 
show parameters db_name;

---

CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO 'c:\app\Administrador\virtual\FRA\ORCL\%F';

run {
allocate channel d1 type disk;
restore controlfile from autobackup 
release channel d1;
}

# restore spfile from autobackup recovery area='c:\app\administrador\virtual\FRA' db_name=orcl;

# create spfile from pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\initorcl.ora';
'c:\app\Administrador\virtual\FRA\ORCL\BACKUPSET\2021_04_23\O1_MF_NCNNF_CONTROLS_0_J86SQS45_';

# Recatalogar o novo caminho
catalog start with 'c:\app\Administrador\virtual\FRA\ORCL\AUTOBACKUP\2021_04_23\';
catalog recovery area;
catalog db_recovery_file_dest;
```

##### PROBLEMA NO CONTROL FILE

```sql
# Verifiquei no alert.log (control01.ctl nao foi possivel abrir o arquivo)
show parameter control_files;
alter system set control_files='c:\app\adminstrador\virtual\oradata\orcl\control01.ctl' scope=spfile;
---
SET DBID = 1569221745;
startup force nomount;
restore spfile from autobackup recovery area='c:\app\administrador\virtual\FRA' db_name=orcl;
startup force;
---
create spfile from pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\initorcl.ora';
create pfile='c:\app\administrator\virtual\product\12.2.0\dbhome_1\database\initprod.ora' from spfile;

startup mount pfile='c:\app\administrator\virtual\product\12.2.0\dbhome_1\database\initorcl.ora';

# OUTRA MANEIRA
create pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\init.ora';

# Edit o init.ora e mude os parametros para sua instancia
startup pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\initorcl.ora';
create spfile from pfile='c:\app\administrador\virtual\product\12.2.0\dbhome_1\database\initorcl.ora';
```

