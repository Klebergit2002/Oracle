#### COLETANDO DADOS PARA MIGRAÇÃO 10g --> 11g release 2.0.4

```sql
# VERIFICAR CHARACTERSET DO BANCO ORIGEM (ANTIGO)
--Variavel de ambiente
NLS_LANG=BRAZILIAN PORTUGUESE_BRAZIL.WE8ISO8859P1

--Principais charsets
NLS_CHARACTERSET		WE8ISO8859P1
NLS_NCHAR_CHARACTERSET	AL16UTF16

--------------------------
--NLS_DATABASE PARAMETROS
--------------------------
NLS_LANGUAGE			BRAZILIAN PORTUGUESE
NLS_TERRITORY			BRAZIL
NLS_CURRENCY			R$
NLS_ISO_CURRENCY		BRAZIL
NLS_NUMERIC_CHARACTERS	,.
NLS_CHARACTERSET		WE8ISO8859P1
NLS_CALENDAR			GREGORIAN
NLS_DATE_FORMAT			DD/MM/RR
NLS_DATE_LANGUAGE		BRAZILIAN PORTUGUESE
NLS_SORT				WEST_EUROPEAN
NLS_TIME_FORMAT			HH24:MI:SSXFF
NLS_TIMESTAMP_FORMAT	DD/MM/RR HH24:MI:SSXFF
NLS_TIME_TZ_FORMAT		HH24:MI:SSXFF TZR
NLS_TIMESTAMP_TZ_FORMAT	DD/MM/RR HH24:MI:SSXFF TZR
NLS_DUAL_CURRENCY		Cr$
NLS_COMP				BINARY
NLS_LENGTH_SEMANTICS	BYTE
NLS_NCHAR_CONV_EXCP		FALSE
NLS_NCHAR_CHARACTERSET	AL16UTF16
NLS_RDBMS_VERSION		10.2.0.4.0
---
select * from nls_database_parameters;
select * from nls_instance_parameters;
select * from nls_session_parameters;
select * from v$version;

#-------------------------------------
# DATAFILES/TABLESPACES/SCHEMAS/USERS
#-------------------------------------
SIPES_DATA
SIPES_INDEX
SIPES_DATA_HIST
SIPES_DATA_TEMP
SIPES_INDEX_TEMP
SIPES_DATA_16K
--
CONSIG 			CONSIG_DATA
PLAN_DADOS 		PLAN_DATA
RECAD_DADOS 	RECAD_DATA
CESSAO_DADOS 	CESSAO_DATA
SADV3 			SADV3_DATA
FHS_DADOS 		FHS_DATA
TEMP

select * from dba_data_files order by tablespace_name,file_name;

select file_name,tablespace_name, bytes/1024/1024/1024
  from dba_data_files order by tablespace_name,file_name;
  
/ora/database/oradisk03/sead/cessao_dados_01.dbf	CESSAO_DADOS	 200M
/ora/database/oradisk03/sead/consig_01.dbf	        CONSIG	        3072M
/ora/database/oradisk03/sead/fhs_dados_01.dbf	    FHS_DADOS	     100M
/ora/database/oradisk03/sead/plan_dados_01.dbf	    PLAN_DADOS	    1024M
/ora/database/oradisk03/sead/recad_dados_01.dbf	    RECAD_DADOS	    2048M
/ora/database/oradisk03/sead/sadv3_01.dbf	        SADV3	         100M
---
/ora/database/oradisk03/sead/sipes_data_01.dbf	    SIPES_DATA	     7G (01..16) = 112G
/ora/database/oradisk03/sead/sipes_data_hist_01.dbf	SIPES_DATA_HIST	10G (01..04) =  40G
/ora/database/oradisk03/sead/sipes_data_temp_01.dbf	SIPES_DATA_TEMP	 5G
/ora/database/oradisk03/sead/sipes_data_temp_02.dbf	SIPES_DATA_TEMP	 2G
/ora/database/oradisk03/sead/sipes_data_16k_01.dbf	SIPES_DATA_16K	 3G
/ora/database/oradisk03/sead/sipes_data_16k_02.dbf	SIPES_DATA_16K	 3G

/ora/database/oradisk03/sead/sipes_index_01.dbf	    SIPES_INDEX	     5G
/ora/database/oradisk03/sead/sipes_index_02.dbf	    SIPES_INDEX	     5G
/ora/database/oradisk03/sead/sipes_index_03.dbf	    SIPES_INDEX	     5G

/ora/database/oradisk03/sead/sipes_index_temp_01.dbf SIPES_INDEX_TEMP 2G

select file_name,tablespace_name, bytes/1024/1024/1024
  from dba_data_files order by tablespace_name,file_name;

select distinct owner,tablespace_name from dba_indexes order by 1,2;

#----------------------------
# INDEXES
#----------------------------
select * from dba_segments;

select SUM(bytes/1024/1024/1024) from dba_segments where segment_type='INDEX' and owner = 'ADMSFP';

select segment_type,sum(bytes/1024/1024/1024) from dba_segments 
where owner in ('ADMSFP','CESSAO','CONSIG','ESMARTINS','ESOCIAL','FHS','LSNETO','PLAN','RECAD','SADV3','USR1','WFGOMES') 
group by segment_type;

select owner,segment_type, sum(bytes/1024/1024/1024) from dba_segments 
where owner in ('ADMSFP','CESSAO','CONSIG','ESMARTINS','ESOCIAL','FHS','LSNETO','PLAN','RECAD','SADV3','USR1','WFGOMES') 
group by owner,segment_type order by owner, 3 desc;

select * from dba_segments where segment_type like '%INDEX%' 
and owner in ('ADMSFP','CESSAO','CONSIG','ESMARTINS','ESOCIAL','FHS','LSNETO','PLAN','RECAD','SADV3','USR1','WFGOMES') 

#-----------------------------------------------
# MOVER TABLES,INDEXES E LOBS OUTRO TABLESPACE
#-----------------------------------------------
SELECT 'ALTER TABLE CONSIG.' || table_name || ' MOVE TABLESPACE nome_do_novo_tablespace;'
  FROM dba_tables WHERE owner = 'CONSIG';

SELECT 'ALTER INDEX CONSIG.' || index_name || ' REBUILD TABLESPACE nome_do_novo_tablespace;'
  FROM dba_indexes WHERE owner = 'CONSIG' AND index_type != 'LOB';

SELECT 'ALTER TABLE CONSIG.' || table_name ||
  ' MOVE LOB( ' || COLUMN_NAME ||
  ' ) STORE AS (TABLESPACE nome_do_novo_tablespace);'
  FROM dba_tab_columns
  WHERE
    owner = 'CONSIG' AND
    data_type LIKE '%LOB';

# ------------------
# CONECTAR SSH -X 
# ------------------
$ ssh -X oracle/12345678@172.28.64.64
dbca
ou
# ORACLE 11g
dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbname seed -sid seed -responseFile NO_VALUE -characterSet WE8ISO8859P1 -sysPassword oracle11g -systemPassword o -databaseType MULTIPURPOSE -memoryPercentage 30 -totalMemory 800 -redoLogFileSize 20 -emConfiguration NONE
--
-characterSet AL32UTF8 \
-sampleSchema true
NLS_NCHAR_CHARACTERSET	AL16UTF16

# CREATE TABLESPACE
CREATE BIGFILE TABLESPACE Exemplo
datafile '/u02/oracle/data/exemplo01.dbf' AUTOEXTEND ON NEXT 1G;

CREATE BIGFILE TABLESPACE "SIPES" DATAFILE '/ora/database/oradisk03/sead/sipes01.dbf' size 1G AUTOEXTEND ON NEXT 1GM maxsize 2G NOLOGGING;
```

#### INSTALAÇÃO DO RDBMS-ORACLE 11g release 2.0.4

```sql
# CONSIDERANDO QUE O S.O. ESTA INSTALADO E CONFIGURADO
# VER A VERSÃO DO ORACLE-LINUX INSTALADA
IP: 172.28.64.64
login: root
senha: PJS3pl4G

cat /etc/oracle-release
Oracle Linux Server release 7.9

# INSTALAR OS PRE-REQUISITOS PARA O ORACLE
yum search oracle-rdbms*
yum install oracle-rdbms-server-11gR2-preinstall
yes
yes

# INSTALAR PACOTES DE APOIO
yum install tree mc unzip zip htop

# DIRETORIOS PARA O ORACLE
mkdir -p /u01/app/oracle/product/11.2.0/db_1
mkdir -p /u02/fra
chown -R oracle:oinstall /u01 /u02
chmod -R 755 /u01 /u02
---
find /u01
find /u01 /u02
tree -L 3 -A -d u01
tree -L 2 -A -d /u01 /u02

# HABILITAR ssh -X
nano /etc/ssh/sshd_config
ForwardX11 yes
ou
X11Forwarding yes

# INSTALACAO DO ORACLE
- Instalar como usuario oracle
- Filezilla para transferir os arquivos
- Definir as variaveis de ambiente no .bash_profile de oracle

env-db-sead.sh # Usar somente como referencia

# VARIAVEIS DE AMBIENTE PARA O ORACLE
# Adicionar em .bash_profile

export TMP=/tmp
export ORACLE_HOSTNAME=ol7
export ORACLE_UNQNAME=sead

export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export ORACLE_SID=sead

PATH=/usr/sbin:$PATH:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib

export NLS_LANG='BRAZILIAN PORTUGUESE_BRAZIL.WE8ISO8859P1'
export LANG=en_US.UTF-8
#TNS_ADMIN=/u01/app/oracle/product/11.2.0/db_1/network/admin

alias dba='sqlplus "/ as sysdba"'

umask 022

---

# ALIASES
alias tns='cd $ORACLE_HOME/network/admin'
alias envo='env | grep ORACLE'

---

mkdir install
cp p13390677_112040_Linux-x86-64_1of7.zip /home/oracle/install
cp p13390677_112040_Linux-x86-64_2of7.zip /home/oracle/install

cd /home/oracle/install
unzip -oq p133*.zip

cd /home/oracle/install
chown -R oracle:oinstall *.zip 
Verificar todas as permissoes (oracle:oinstall)
--
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -X oracle@172.28.64.64

cd /home/oracle/install/database
./runinstaller

---
# Error in invoking target 'agent nmhs' of makefile
# $ORACLE_HOME/sysman/lib/ins_emagent.mk
Editar $ORACLE_HOME/sysman/lib/ins_emagent.mk
Trocar 
$(MK_EMAGENT_NMECTL) por 
$(MK_EMAGENT_NMECTL) -lnnz11
Salvar o arquivo
Click no botao Retry
---
Editar /etc/oratab 
sead:/u01/app/oracle/product/11.2.0/db_1:Y

grep -iw error /u01/app/oraInventory/logs/installActions2021-10-29_06-43-10PM.log

# LOGS INVENTORY
grep -iw error /u01/app/oraInventory/logs/installActions2021-10-25_03-30-43PM.log

# DESINSTALANDO O ORACLE 11g
# NAO FUNCIONOU
./runInstaller -deinstall -home /u01/app/oracle/product/11.2.0.4/dbhome_1

# FUNCIONOU
$ORACLE_HOME/deinstall/deinstall
Depois tem que apagar duas pastas como root
O Script mostra no final do processo.

```

