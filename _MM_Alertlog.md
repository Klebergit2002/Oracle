##### ALERT.LOG EM UMA TABELA DO BANCO

```sql
# Criar um diretório no banco de dados para acesso ao arquivo “alert.log”
drop directory ALERT_LOG;
create directory ALERT_LOG as 'C:\app\Administrador\virtual\diag\rdbms\producao\producao\trace';

# Cria uma tabela externa apontando diretamente para o arquivo “alert.log”
create table alert_log (msg varchar2(150))
organization external (
   type oracle_loader
   default directory ALERT_LOG
   access parameters (records delimited by newline)
   location('alert_producao.log')
)
reject limit unlimited;
# reject limit 1000;
# alter table alert_log reject limit unlimited;

# EXEMPLOS DE SELECTS
select msg from alert_log where msg like 'ORA-00600%';

SELECT msg FROM alert_log FETCH FIRST 25 ROWS ONLY;
SELECT msg FROM alert_log OFFSET 1000 ROWS FETCH FIRST 25 ROWS ONLY;
SELECT msg FROM alert_log ORDER BY 1 DESC OFFSET 1000 ROWS FETCH FIRST 25 ROWS ONLY;

# ORA-12170 TNS:Connect timeout
Action: If the error occurred because of a slow network or system, reconfigure one or all of the parameters 
%ORACLE_HOME%\network\admin\sqlnet.ora
SQLNET.INBOUND_CONNECT_TIMEOUT
  Recomendações:
  INBOUND_CONNECT_TIMEOUT_LISTENERname=2 in listner.ora
  SQLNET.INBOUND_CONNECT_TIMEOUT=3 in sqlnet.ora
SQLNET.SEND_TIMEOUT=3
SQLNET.RECV_TIMEOUT=3
in sqlnet.ora to larger values. 

SQLNET.ORA: ALTERAR O PARAMETRO EM \ORACLE_HOME\network\admin
PARAMETRO: SQLNET.AUTHENTICATION_SERVICES=(NTS)
ORACLE_HOME: C:\app\Administrator\virtual\product\12.2.0\dbhome_1
PATH: %ORACLE_HOME%\network\admin\sqlnet.ora

If a malicious client is suspected, use the address in sqlnet.log to identify the source and restrict access. Note that logged addresses may not be reliable as they can be forged (e.g. in TCP/IP).
See Also:
"Configuring the Listener and the Oracle Database To Limit Resource Consumption By Unauthorized Users" for further information about setting the SQLNET.INBOUND_CONNECT_TIMEOUT parameter

# ERRO: Thread 1 cannot allocate new log, sequence ...
alter system set archive_lag_target=0 scope=both;
```

##### CAMINHO DO ALERT.LOG

```sql
# C:\app\administrator\virtual\diag\rdbms\producao\producao\trace
# C:\app\Administrador\virtual\diag\rdbms\orcl\orcl\trace\alert_orcl.log

# VER O CAMINHO DO ALERT.LOG
show parameters pfile;
show parameters dump;
select * from v$parameter order by name;
---
# ABRIR NO LINUX
cd /path-do-alert.log
tail -f

# ABRIR NO POWER SHELL
c:
cd C\app\Administrator\virtual\diag\rdbms\producao\producao\trace
Get-Content alert_producao.log -Wait -Tail 30
```

