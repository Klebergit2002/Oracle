##### ORACLE 12c NO ORACLE LINUX 7.7 

```shell
# Baixar o Oracle Linux 7.7 
https://edelivery.oracle.com
kleber2002@gmail.com/Xadrez@2002

# Baixar o ORACLE
LINUXX64_193_db_home.zip

# Iniciar a partir da ISO
-Linguagem English
-Configurações:
-Date e Time
-Software: 
-  Minimal install
-Network & Host name: 
-    ol7-dba:localdomain
-    Ethernet em ON

# Begin instalation
-Colocar a senha: Root password
-Reboot
-Entrar como root:
-Pegar o ip addr para conectar via ssh
-No windows usar o mobaXterm

# VER A VERSÃO DO ORACLE-LINUX INSTALADA
cat /etc/oracle-release

# INSTALAR OS PRE-REQUISITOS PARA O ORACLE
yum install mc unzip zip
yum install oracle-database-preinstall-19c
yum install oracle-rdbms-server-12cR1-preinstall 
yum install oracle-rdbms-server-11gR2-preinstall
yes
yes
---
# TROCAR PARA IP ESTATICO
nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
-Alterar: BOOTPROTO="dhcp" para "static"
-Adicionar: IPADDR=IP da sua maquina
-Reboot
---
nano /etc/hosts
-Adicionar a linha:
-Ip da maquina     ol7-dba.localdomain     ol7-dba
-Para pingar pelo nome da maquina
---
passwd oracle 
nano /etc/selinux/config
-Alterar SELINUX=disable
---
systemctl stop firewalld
systemctl disable firewalld
---
# Criação dos diretorios para o Oracle
mkdir -p /u01/app/oracle/product/11.2.0.4/dbhome_1
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
mkdir -p /u02/oradata
chown -R oracle:oinstall /u01 /u02
chmod -R 755 /u01 /u02
---
find /u01
tree -L 3 -A -d /ora
tree -L 2 -d /u01

# HABILITAR ssh -X
nano /etc/ssh/sshd_config
ForwardX11 yes
ou
X11Forwarding yes

# INSTALACAO DO ORACLE
-Instalar com usuario oracle

-Definir as variaveis de ambiente
.bash_provile do usuario oracle

-Entrar no ORACLE_HOME
cd $ORACLE_HOME
-Usar o filezilla para copiar o instalador
-No oracle 19c tem que descompactar no ORACLE_HOME
unzip -oq 19cLINUX....zip
./runinstaller
```

##### INSTALAÇÃO NO MODO SILENT

```shell
#######PRE_CONFIGIRAÇÃO#########
-- desabilitando firewall
systemctl stop firewalld
systemctl disable firewalld

--configuração interface de rede
--Configurar no instalador grafico
nmcli device status
ip addr show dev enp0s3

-- ajustar /etc/hosts -> Para pingar pelo nome
10.0.2.15 ol77 ol77.localdomain
ip addr show

-- instalando pacote preinstall
yum search preinstall
yum install oracle-database-server-12cR2-preinstall.x86_64
yum install oracle-rdbms-server-11gR2-preinstall

-- criando user oracle 
passwd oracle

-- grupos do S.O
groupadd -g oinstall
groupadd -g dba
groupadd -g oper
groupadd -g backupdba
groupadd -g dgdba
groupadd -g kmdba
groupadd -g racdba

-- atribuindo grupos ao oracle 
usermod -g oinstall -G dba,oper,backupdba,dgdba,kmdba,racdba oracle

-- Diretorio do instalador e script de variaveis de ambiente
--CRIAR ARQUIVO env-db-SID.sh
chmod +x env-db-orcl.sh
/home/oracle
  env-db-orcl.sh
/home/oracle/stage
  p13390677_112040_Linux-x86-64_1of7.zip
  p13390677_112040_Linux-x86-64_2of7.zip
  
-- criando diretorio base e atribuindo permissão
mkdir -p /u01/app/oracle 
chown -R oracle.oinstall /u01 

cd /home/oracle/stage
unzip p13390677_112040_Linux-x86-64_1of7.zip
unzip p13390677_112040_Linux-x86-64_2of7.zip

cd /home/oracle/stage/database/response
cp *.rsp /home/oracle
cd /home/oracle
nano db_install.rsp
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=ol77
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oraInventory
ORACLE_HOME=/u01/app/oracle/product/11.2.0.4/db_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
DECLINE_SECURITY_UPDATES=true
oracle.installer.autoupdates.option=SKIP_UPDATES

#####INSTALAÇÃO MODO SILENT########
-- pacote screen para recuperar sessions
yum install screen
screen (abre sessão)
screen -r 

-- comando de instalação SGBD
arquivo: db_install.rsp
cd /home/oracle/stage/database
. ~/env-db-orcl.sh

screen
./runInstaller -ignoreSysPrereqs -showProgress -silent -responseFile  /home/oracle/db_install.rsp
# OBS: Depois do rodar os scripts como root checar o arquivo de log

-- comando criação listener (NETCA)
arquivo: netca.rsp
# OBS: Não precisa alterar nada no netca.rsp
netca -silent -responsefile /home/oracle/netca.rsp
lsnrctl status

-- Banco de dados (DBCA)
Pode ser deito de duas formas
- arquivo: dbca.rsp

- ou passando todos os atributos
# ORACLE 12c
dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbname orcl -sid 
orcl -responseFile NO_VALUE -characterSet WE8ISO8859P1 -sysPassword suporte -systemPassword suporte -createAsContainerDatabase false -databaseType MULTIPURPOSE -memoryMgmtType auto_sga -totalMemory 800 -redoLogFileSize 300 -emConfiguration NONE

# ORACLE 11g
dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbname orcl -sid orcl -responseFile NO_VALUE -characterSet WE8ISO8859P1 -sysPassword oracle11g -systemPassword oracle11g -databaseType MULTIPURPOSE -memoryPercentage 30 -totalMemory 800 -redoLogFileSize 50 -emConfiguration NONE
--
-characterSet AL32UTF8 \
-sampleSchema true

```

##### env-db-SID.sh

```shell
#!/bin/bash
########################################################################## 
# PROGRAMA:		env-db-<SID>.sh
# Objetivo:		Configurar variaveis de ambientes
##########################################################################
umask 022
EDITOR=vi;                   export EDITOR
TERM=xterm;                  export TERM
TEMP=/tmp;                   export TEMP
TMPDIR=/tmp;                 export TMPDIR

##########################################################################
# CONFIGURAR AMBIENTE ORACLE  
##########################################################################
export ORACLE_SID=sid
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/11.2.0.3/dbh_1
export ORACLE_UNQNAME=sid

export NLS_LANG=AMERICAN_AMERICA.WE8ISO8859P1
#export LANG=us_EN.UTF-8

export ORACLE_OWNER=oracle
export ORACLE_TERM=xterm

#########################################################################
# CONFIGURAR PATH        
#########################################################################
export PATH=$ORACLE_HOME/bin:$ORA_CRS_HOME/bin:$PATH:/usr/local/bin
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$ORA_CRS_HOME/lib:/usr/local/lib:$LD_LIBRARY_PATH
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib

##########################################################################
# Extra
##########################################################################
alias dba='sqlplus "/ as sysdba"'
```
