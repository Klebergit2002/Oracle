##### ENVIO DE RALATÓRIOS

```ini
# PROCEDIMENTOS PARA PAGAMENTO DO SALÁRIO APPLE IT
1. Enviar Relatório Mensal para Ana pelo Whatsapp
2. Ana envia para Paulão para aprovação
3. Depois de aprovado enviar para o FINANCEIRO
 email: financeiro@appleit.com.br
```

##### ANYDESK - INSTALAÇÃO E CONEXÃO

```sh
wget https://download.anydesk.com/linux/anydesk_6.1.0-1_amd64.deb -O anydesk.deb
sudo dpkg -i anydesk.deb
sudo apt install -f
```

##### ACESSO VIA RDP & PSTOOLS

```sql
# COMPARTILHAMENTO ADM
\\172.24.8.17\c$	HOMO-AG
\\172.24.8.39\c$	HOMO-CLI 
\\172.24.8.171\c$	LAB
\\172.24.8.9\c$		PROD-AG
\\172.24.8.19\c$	PROD-CLI
c:
cd \app\administrator\virtual\scripts

# DOS REMOTO (PSTOOLS)
psexec \\172.24.8.171 -s cmd --LAB 
psexec \\172.24.8.171 "c:\MC\mc.exe" 
psexec \\172.24.8.171 "c:\Windows\explorer.exe" 
psexec \\172.24.8.17  "c:\Windows\explorer.exe"
---
psexec \\172.24.8.17 -s cmd  --HOMO-AG
psexec \\172.24.8.39 -s cmd  --HOMO-CLINICAS
psexec \\172.24.8.9  -s cmd  --PROD-AG
psexec \\172.24.8.19 -s cmd  --PROD-CLINICAS

# POWER SHELL REMOTO
Enter-PSSession Ipesaude-bdh01
Exit-PSSession
 172
```

##### LOCAL DOS SCRIPTS DE BACKUP

```sql
\\172.24.8.9\c$\app\administrator\virtual\scripts
```

##### AGENDANDO TAREFAS NO WINDOWS

```sql
# DOS SHELL
sysdm.cpl # Ajustar desempenho do windows
taskschd  # Task Manager
services.msc
taskmgr #

# POWER SHELL
Get-Content  -Wait -Tail 30
Get-Content .\backup_full_homo_ag_2021-04-30_22-55-29.log -Wait -Tail 30

SQL> host strings -a ARQUIVO.DBF ??
```

##### OMF (Oracle-Managed Files )

```ini
# PARA UTILIZAR OMF DEVEMOS CONFIGURAR OS SEGUINTES PARAMETROS
DB_CREATE_FILE_DEST :Local dos datafiles e tempfiles
show parameters db_create_file_dest;
alter system set db_create_file_dest='/u01/app/oracle/oradata/' scope=both;
---
DB_CREATE_ONLINE_LOG_DEST_n :Local dos redo log controlfiles (n para multiplexar)
---
DB_RECOVERY_FILE_DEST_SIZE: Define o tamnho da FRA
DB_RECOVERY_FILE_DEST :Local da FRA para backup do RMAN
```

##### DBID/VARIAVEIS DE AMBIENTE BANCOS DE DADOS DO IPES

```shell
set ORACLE_SID=ORCL
set ORACLE_HOME=c:\app\Administrador\virtual\product\12.2.0\dbhome_1
set ORACLE_BASE=c:\app\Administrador\virtual
---
set DBID=1569221745; ??
---
set DBID=3426120398; (LAB-AG)
set DBID=3426120398; (HOMO-AG)
set DBID=4135542948; (HOMO-CLINICAS)
---
set DBID=3331875998; (PROD-AG)
set DBID=4119043554; (PROD-CLINICAS)
```

##### ENDEREÇOS DB

```ini
LAB       : 172.24.8.171 ipesaude-bdh01 producao     
HOMO-AG   : 172.24.8.17  ipesaude-bdh01 producao     
HOMO-CLI  : 172.24.8.39  Ipesaude-bdh02 clinicas    
PROD-CLI  : 172.24.8.19  Ipesaude-bd02  clinicas    
PROD-AG   : 172.24.8.9   Ipesaude-bd03  producao    
PROD-ITABS: 172.24.8.41  ipesaude-abs01 itabs    
PROD-BI   : 172.24.8.3   IPESAUDE-BI01    
```

##### LEVANTAMENTO DE PARAM/TAMANHO DOS BANCOS 

```sql
select * from v$session;
select * from dict where table_name like '%DBA%';
select * from v$database;
select * from dba_data_files;
select * from dba_tablespaces;
select * from dba_control_files;
show parameters control_files;

select * from  dba_data_files;
select sum(bytes)/Power(1024,3) Gbytes from dba_data_files;
select tablespace_name, bytes/Power(1024,3) Gbytes, file_name,autoextensible from dba_data_files order by 2 desc;
```

##### ALTERAÇÃO DO AUTOEXTEND ON/OFF

```sql
alter database datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF' autoextend on;
alter database datafile 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF' autoextend on;
```

##### SQLNET.ORA

```ini
SQLNET.ORA: ALTERAR O PARAMETRO EM \ORACLE_HOME\network\admin
PARAMETRO: SQLNET.AUTHENTICATION_SERVICES=(NTS)
ORACLE_HOME: C:\app\Administrator\virtual\product\12.2.0\dbhome_1
PATH: %ORACLE_HOME%\network\admin\sqlnet.ora
```

##### SQLPLUS

```sql
alter database open read only;
select * from v$diag_info order by name;
```

##### PROMPT DE COMANDOS

```shell
$ tnsping producao
$ crontab -e
# MODELO CRONTAB PBACKUP RMAN
20 22 * * 5 /bin/bash /u01/teste/bkp_nivel0_teste.rcv
30 22 * * 1,2,3,4,7 /bin/bash /u01/teste/bkp_nivel1_teste.rcv
```

##### ENTERPRISE MANAGER

```sql
# Consultado a porta
select dbms_xdb_config.getHttpsPort() from dual;

# Definindo a porta
exec dbms_xdb_config.sethttpsport(port-number);

# Acessando o EM
https://172.24.8.9:5500ort/em
```

##### PARAMETROS DE SESSION/PROCESS/TRANSACTIONS

```sql
# Verifique o valor dos parametros;
show parameter sessions; 
show parameter processes;
show parameter transactions;

Se você tem a intenção de aumentar o numero de processos, você tambem terá que modificar o numero de sessões e transações.

Para não colocar qualquer numero a Oracle indica uma conta simples para definir esses parâmetros;

processes=x
sessions=x*1.1+5
transactions=sessions*1.1

Definido o valor será necessário apenas modificar o parametro se você utiliza o SPfiles;

alter system set processes=xxx scope=spfile;
alter system set sessions=xxx scope=spfile;
alter system set transactions=xxx scope=spfile;

shutdown immediate 
startup


```

