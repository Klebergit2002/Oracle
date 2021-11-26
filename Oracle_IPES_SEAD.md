##### VERIFICANDO A VERSÃO E FEATURES DO ORACLE

```bash
select * from v$version;
select * from dba_feature_usage_statistics where currently_used = 'TRUE';
```

##### ORACLE SEPLAG

```ini
# VERSÃO PRODUÇÃO
Oracle Database 10g Enterprise Edition Release 10.2.0.4.0 - 64bi
PL/SQL Release 10.2.0.4.0 - Production
"CORE	10.2.0.4.0	Production"
TNS for Linux: Version 10.2.0.4.0 - Production
NLSRTL Version 10.2.0.4.0 - Production

# VERSÃO HOMOLOGAÇÃO
Oracle Database 10g Release 10.2.0.4.0 - 64bit Production
PL/SQL Release 10.2.0.4.0 - Production
"CORE	10.2.0.4.0	Production"
TNS for Linux: Version 10.2.0.4.0 - Production
NLSRTL Version 10.2.0.4.0 - Production

# FEATURES - PRODUÇÃO E HOMOLOGAÇÃO (IGUAIS)
Automatic Segment Space Management (system)
Automatic Segment Space Management (user)
Automatic SQL Execution Memory
Automatic Undo Management
Character Set
Extensibility
LOB
Locally Managed Tablespaces (system)
Locally Managed Tablespaces (user)
Object
Partitioning (system)
Protection Mode - Maximum Performance
Recovery Manager (RMAN)
RMAN - Disk Backup
Segment Advisor
Server Parameter File
Shared Server
Streams (system)
Streams (user)
Virtual Private Database (VPD)
XDB

# TEM NO 10g ENTERPRISE e NÃO TEM NO 12c STANDARD
Protection Mode - Maximum Performance
RMAN - Disk Backup
Segment Advisor
Shared Server
Streams (system)
Streams (user)
Virtual Private Database (VPD)
XDB 

```

##### ORACLE IPES

```ini
# VERSÃO
Oracle Database 12c Standard Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production

# FEATURES
Adaptive Plans
Automatic Maintenance - Optimizer Statistics Gathering
Automatic Maintenance - Space Advisor
Automatic Maintenance - SQL Tuning Advisor
Automatic Reoptimization
Automatic Segment Space Management (system)
Automatic Segment Space Management (user)
Automatic SGA Tuning
Automatic SQL Execution Memory
Automatic Undo Management
Character Set
DBMS_STATS Incremental Maintenance
Deferred Segment Creation
Extensibility
Instance Caging
LOB
Locally Managed Tablespaces (system)
Locally Managed Tablespaces (user)
Object
Oracle Java Virtual Machine (system)
Oracle Utility Datapump (Export)
Oracle Utility Metadata API
Partitioning (system)
Real Application Security
Recovery Area
Resource Manager
SecureFiles (system)
SecureFiles (user)
Server Parameter File
Services
SQL Plan Directive
SQL Profile
Traditional Audit
Unified Audit
```

##### EMAIL

```ini
A versão Standard do banco de dados Oracle possui um limite de hardware de até 2 processadores físicos e um limite de software de 8 threads simultâneas por instância. 
Reuniao EMGETIS DIA 20/07/2021
Reunião SEAD DIA 21/07/202
Conforme entendimentos acordados em reuniões na EMGETIS e SEAD, teremos os seguintes cenários:
SGBD's envolvidos 
Oracle12c Standard Edition (IPES) - Limitações 2 processadores fisicos e 8 threads simultaneas
Oracle11g Enterprise Edition RAC (SEAD)
Oracle10g Enterprise Edition (SEAD)

1. Instalar o Oracle12c Standard, não RAC, cedido pelo IPES, em cada maquina disponibilizada
   pela SEAD, onde teremos uma maquina com as intâncias de Produção da SEAD (Sipes, Patrimonio, Seplan e BI) 
   e a outra com as instâncias do IPES (AG, Clinicas).
   
   
   Essa opção necessita verificar se o Oracle12 Standard contempla todos os requisitos 
   necessário para o Sistema da Folha de Pagamento do Estado (SIPES)

2. Atualizar o Oracle12c Standard para a versão Oracle12c RAC


```



##### STARTUP E SHUTDOWN EM UMA INSTANCIA ASM ORACLE

```bash
/etc/oratab
+ASM1:/u01/app/grid:N
pbanco:/u01/app/oracle/product/11.2.0.3/db_1:N

export ORACLE_SID=+ASM1
export ORACLE_HOME=/u01/app/grid/
sqlplus / as sysasm

. oraenv
export ORACLE_SID=+ASM
export ORACLE_SID=pbanco
export ORACLE_HOME=/u01/app/oracle/product/11.2.0.3/db_1/
sqlplus /nolog
conn / as sysdba

```



```
1.ORACLE RAC
Infraestrutura Oracle
LINUX
  ASM Sistema de arquivos (LVM)
  GRID
RAC (Varias maquinas que apontam para um STORAGE com todos os bancos)
  Gerenciamento do RAC

Pre requisitos:
Suporte tecnico, curso ...

2.ORACLE single-instance
  LINUX
    ext4
```

