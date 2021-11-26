##### ENTRANDO NA VPN

```bash
$ sudo openfortivpn

# NOVA MAQUINA PARA ORACLE 11
IP: 172.28.64.64
login: root
senha: PJS3pl4G

$ ssh -X oracle/12345678@172.28.64.64
$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -oracle@172.28.64.64

```

##### AGENDA THIAGO - 172.28.64.68

```bash
$ ssh oracle/"ORAdb@618"@172.28.64.68
$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 oracle@172.28.64.68
$ oracle/ORAdb@618

# AGENDA
- SYS: sysagendadb#618
- SYSTEM: systemagendadb#618
- AGENDA: eiUGv7uYi8utntjG
- Host: 172.28.64.68
- Porta: 1521
- SID: agenda
- Tablespace de dados: TS_agenda
- alert.log: tail -f50 /ora/app/oracle/admin/agenda/bdump/alert_

# ALTERAÇÕES EM 24/09/2021

# AUMENTO DO TAMANHO DOS REDOLOGs
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;

alter database add logfile group 4 ('/ora/app/oracle/oradata/agenda/redo01a.log') size 150M;
alter database add logfile group 5 ('/ora/app/oracle/oradata/agenda/redo02a.log') size 150M;
alter database add logfile group 6 ('/ora/app/oracle/oradata/agenda/redo03a.log') size 150M;

alter system switch logfile;
alter system checkpoint;
alter system checkpoint global;

# APAGAR SOMENTE OS GRUPOS COM STATUS INACITVE
select group#, status from v$log;
alter database drop logfile group 1;
alter database drop logfile group 2;
alter database drop logfile group 3;

# AUMENTO DA MEMORIA (SGA-PGA)
show parameter target;
show parameter memory
show parameter sga
show parameter pga
show parameter spfile
--
create pfile from spfile;
alter system set sga_target = 1536M scope=spfile; 
alter system set sga_max_size = 1536M scope=spfile; 
alter system set pga_aggregate_target = 308M scope=spfile; 

$ shutdown immediate
$ startup

```

##### COLETANDO DADOS - 172.28.64.64

```bash
# Server: 172.28.64.64/root/SP9%68$*
$ ssh root@172.28.64.64
---
$ . oraenv
$ uname -a
$ lsb_release -a
$ find . -name oracle

Versões instaladas atualmente
Oracle Linux Server release 6.4
ORACLE 11g 11.2.0.3.0

# OBS - OL7 Oracle Linux Server 7
O OL7 so é compatível com a versão Oracle 11g a partir da 11.2.0.4.0

```

##### COLETANDO DADOS - 172.20.1.46

```bash
# Server: 172.20.1.46/oracle/dorp#347@soaro
$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 oracle@172.20.1.46
---
$ lsb_release -a
$ find . -name oracle

Red Hat Enterprise Linux Server release 5.4 (Tikanga) 2.6.18-164.6.1.el5 
Oracle Database 10g Enterprise Edition Release 10.2.0.4.0 - 64bi

# BANCO DE DADOS/BACKUP(FRA)
Filesystem              Total   Usado  
/ora/database/oradisk03  591G    363G
/ora/backup              375G    206G

# BANCO HOMOLOGAÇÃO (500G)
sead 250G
patrimon 20G
```

##### RESUMOS

```sql
D26/08/2021
Boa noite Elvis,
Ja estou fazendo a coleta do que precisa para a Migração do nosso Banco.
Não encontrei no nosso novo servidor o instalador do Oracle Database 11g, conforme Antonio me pediu para verificar.

*NOSSA CONFIGURAÇÃO ATUAL*:
*Host*:
IP: 172.20.1.46
CPU: Intel(R) Xeon(R) CPU E5440 @ 2.83GHz (Quad-Core)
Memoria: 64 GB
Disco:
Total para o banco: 600Gb
Usado pelo banco: 363Gb
Total para backup: 400Gb
Usado pelo backup: 200Gb

*Sistema Operacional*:
Red Hat Enterprise Linux Server release 5.4 (Tikanga) 
Kernel: 2.6.18-164.6.1.el5 

*Banco de Dados*:
Oracle Database 10g Enterprise Edition Release 10.2.0.4.0 - 64bi

*PRE REQUISITOS PARA A MIGRAÇẪO EM 172.28.64.64*  
- Oracle Linux Server release 7
- Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bi
- 600Gb para o banco
- 400Gb para o backup do banco

O Sistema Operacional (Oracle Linux Server 7) é de facil acesso, agente pode baixar qualquer versão do site da Oracle.

O Oracle Database 11g também pode ser baixado, mas precisa comprovar que a Seplag tem a licença original da compra do produto. Isso é o que consta no site, não tenho a minima idéia de como fazer isso.

Seria bom ver isso com a empresa que vendeu o Oracle 11g para a Seplag.

Até o momento é isso.

# PRE-REQUISITOS PARA INSTALAÇÃO DO ORACLE 11g 2.0.4
1. Download do Oracle Enterprise Linux 7.9 
   https://edelivery.oracle.com
   Tem que criar uma conta (É gratis)
   Caso tenha dificuldade de baixar o Oracle Linux 7.9, eu tenho a versão 7.7
   que podemos instalar e depois é so executar o yum update para atualizar 
   para a versão 7.9

2. Instalação do Oracle Enterprise Linux 7.9
   Instalar a versão minima somente com usuario root
   Instalar com suporte LVM (Default)
   Acesso VPN
   Acesso SSH
   Nome do Host=ol7sead
   SELINUX=disable
   systemctl stop firewalld
   systemctl disable firewalld

3. Area em disco
   Configuração com dois Arrays (Atende perfeitamente)
   - SGBD e Database 600 Gbytes montar em /u01 
   - FRA/Backup 400 Gbytes      montar em /u02 
   
   Configuração com tres Arrays
   - SGBD 50 Gbytes        montar em /u01
   - Database 600 Gbytes   montar em /u02
   - FRA/Backup 400 Gbytes montar em /u03

   É altamente recomendável que cada área seja em Arrays diferentes 
   
      
```

