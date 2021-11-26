#### Crashmanager (Linux)

```ini
# Utilização do shell crashmanager.sh (Linux)
# Simula perdas aleatórias

# CENÁRIOS DO CRASHMANAGER
1.Loss of a control file
2.Loss of all control files
3.Loss of a redo log file group member
4.Loss of a redo log file group
5.Loss of a non-system datafile
6.Loss of a temporary datafile
7.Loss of a SYSTEM datafile
8.Loss of an UNDO datafile
9.Loss of a Read-Only tablespace
10.Loss of an Index tablespace
11.Loss of all indexes in USERS tablespace
12.Loss of a non-system tablespace
13.Loss of a temporary tablespace
14.Loss of a SYSTEM tablespace
15.Loss of an UNDO tablespace
16.Loss of the password file
17.Loss of all datafiles
18.Loss of redo log member of a multiplexed group
19.Loss of all redo log member of an INACTIVE group
20.Loss of all redo log member of an ACTIVE group
21.Loss of all redo log member of CURRENT group
```

##### 1.PERDA DO SYSTEM01.DBF

```sql
# Abrir o alert log no modo tail -f (LINUX)
# Power shell (WIN)
C:\app\Administrator\virtual\diag\rdbms\producao\producao\trace
Get-Content C:\app\Administrator\virtual\diag\rdbms\producao\producao\trace\alert_producao.log -Wait -Tail 30

sqlplus / as sysdba
select name from v$database;
select status from v$instance;
show parameters dump; # PEGAR O CAMINHO DO ALERT.LOG
exit;
rman target /
validate database;
exit;
sqlplus / as sysdba
alter system switch logfile; # PROVOCANDO O ERRO
/
/
# Nao apareceu o ERRO ainda
create table t1 as select * from all_objects; # Agora apareceu o ERRO
exit;
rman target /

# Como nao podemos colocar a system em off line
# teremos que desligar o banco
shutdown immediate; # NÃO VAI FUNCIONAR
shutdown abort;     # SEMPRE FUNCIONA

# INICIO DA RESTAURAÇÃO
rman target /
startup mount; # PQ TENHO OS CONTROLFILES
report schema; # PARA VER O NUMERO DO DATAFILE
restore datafile 1;
recover datafile 1;
alter database open;
```

##### 2.PERDA DO UNDOTBS01

```sql
sqlplus / as sysdba
create table t1 as select * from all_objects; # PROVOCANDO O ERRO
# Aparaceu logo o ERRO

# INICIO DA RESTAURAÇÃO
rman target /
validate database;
report schema;
restore datafile 4;
# Deu erro porque o tablespace esta em uso
# OBS: A cada erro sair do sqlplus ou rman e entrar novamente

sqlplus / as sysdba
alter database datafile 4 offline; # Colocando em offline
# Nao consegue coloca-lo em offline, para isso temos que desligar o banco;

shutdown immediate; # NAO PERMITE 
# Os redo estão perfeitos então ABORT não terá problemas
shutdown abort;
startup mount;
exit;
rman target /
restore datafile 4;
recover datafile 4;
alter database open;
validate database;
exit;
```

##### 3.PERDA DE TODOS OS CONTROLFILES

```sql
# O que está acontecendo no banco
sqlplus / as sysdba
select count(*) from all_objects;
# Reportou imediatamente o ERRO
shutdown immediate; # NÃO FUNCIONA
shutdown abort; # ESSE SEMPRE FUNCIONA
startup nomount; 
exit;

# INICIO DA RESTAURAÇÃO
rman target /
restore controlfile; # NÃO FUNCIONA
restore controlfile from autobackup; # NÃO ENCONTROU
exit;
# Procurar na FRA-NÃO TEM AUTOBACKUP, NEM TEM BACKUP DOS CONTROLS
# OBS: No primeira peça de cada backup o Oracle silenciosamente 
# faz um backup dos controlfiles.
# Então agente tem que procurar manualmente e tentar achar usando 
# cada um em BACKUPSET da FRA
# Para cada piece de backup fazer o seguinte:
rman target /
restore controlfile from 'caminho e nome backup no BACKUPSET';
# Fazer ate achar um backup que tenha os controlfiles
alter database mount;
recover database; # PORQUE OS CONTROLFILES ESTÃO VELHOS
alter database open; # NÃO FUNCIONOU
alter database open resetslog; # GERANDO OUTRA ENCARNAÇÃO DO BANCO
# Recria tambem todos os redologs
# A primeira coisa a fazer depois de RESETSLOG é um novo backup
validate database;
```

##### 4.PERDA DO UNDOTBS01.DBF (RECUPERAÇÃO SEM BACKUP) 

```sql
# VERIFICANDO O BANCO DEPOIS DA PERDA DO UNDO
sqlplus / as sysdba
select count(*) from all_objects;
create table t1 as select * from all_objects;
alter system switch logfile;
/
/
insert into t1 select * from t1;
# Ate aqui tudo funcionando bem
delete from t1; # MOSTRA O ERRO-delete sem UNDO não é possivel
exit;
sqlplus / as sysdba
show parameter undo;

# ESTRATÉGIA: passar o undo_management para MANUAL
alter system set undo_management='MANUAL' scope=spfile;
shutdown abort; # IMMEDIATE NÃO FUNCIONA SEM UNDO
startup mount;
show parameter undo; # VERIFICAR SE ESTA NO MODO 'MANUAL'
alter database datafile 4 offline drop;
alter database open; # DEU ERRO MAS ABRIU A INSTANCIA
select status from v$instance;

# ESTRATEGIA: Apagar a tablespace de undo, e criar outra
drop tablespace undotbs1;

# Não deixou dropar porque achou rollback segment active, quando 
# a tablespace de undo foi perdida, tinha alguma coisa sendo feita no banco
# ORA-01548: active rollback segment 'SYSSMU1_414232537$' found,
desc dba_rollback_segs;
select segment_name from dba_rollback_segs where status = 'NEEDS RECOVERY';
create pfile form spfile;
shutdown abort;

# Copiar todos segment_name e colocar no pfile
# Editar o pfile e colocar no final o parametro:
# _offline_rollback_segments=('SYSSMU1_414232537$' ,'SYSSMU1_414232537$',
# 'SYSSMU1_414232537$', 'SYSSMU1_414232537$' ) 
# Remover o spfile para o Oracle iniciar pelo pfile
sqlplus / as sysdba
startup
# Da o erro para abrir a instancia (NORMAL)
select status from v$instance; # OPEN
drop tablespace undotbs1;
create undo tablespace undotbs1 datafile 'caminho/undotbs01.dbf' size 100M;
shutdown abort;

# Editar o pfile e alterar
# *.undo_management='AUTO' --DEVE ESTAR COMO 'MANUAL'
# _offline_rollback_segments=(.....) --APAGAR DEPOIS QUE TUDO OCORRER BEM
sqlplus / as sysdba
startup
show parameter undo # CONFERIR SE ESTA EM AUTOMATICO
exit;
rman target /
validate database 
```

##### 5.PERDA DOS SYSTEM01.DBF (RECUPERAÇÃO SEM BACKUP-SO LINUX) 

```sql
# OBS: SO FUNCIONA NO UNIX,LINUX E SEUS DERIVADOS
sqlplus / as sysdba
show parameters dump; --PATH DO ALERT.LOG
rman target /
list backup of datafile 1; # NÃO ACHOU, SEM BACKUP
# OBS: Esse tipo de recuperação é somente no LINUX ou qualquer S.O. UNIX
# Na verdade esse arquivo ainda existe em memória 

# INICIO DA RESTAURAÇÃO
ps aux | grep dbwr
ora_dbw0_ORCL com PID 25554
cd /proc/25554/fd # (fd=File descriptor)
# Ver arquivos em uso com lock exclusivo pelo dbwr
# O system foi movido, então aparece dessa forma
# 258 -> /u01/oradata/ORCL/system01.dbf.2014417_090235.bck
cat 258 >  /u01/oradata/ORCL/system01.dbf # (NOME ANTIGO DA SYSTEM)
chown oracle:oinstall /u01/oradata/ORCL/system01.dbf 
# Achou uma falha
# Entrar com o usuario oracle do SO
rman target /
validate datafile 1;
exit;
# OBS: Se o arquivo SYSTEM fosse apagado ele apareceria 
# 258 -> /u01/oradata/ORCL/system01.dbf (deleted) tudo em vermelho
# Preceder da mesma forma anterior
# Fazer todos os testes na instancia
sqlplus / as sysdba
create table t1 tablespace system as select * from all_objects; 
select count(*) from t1;
exit;
```

##### 6.DEMONSTRANDO O FILE DESCRIPTOR

```ini
Apagando o datafile e desligando o computador
Perde tudo não pode ser restaurado
Os diretorios so existem em tempo de execução
Única saida restaurar o datafile
```

##### 7.COSSCHECK E DELETE EXPIRED

```sql
# Não existe nenhum problema em executa-los
# Ele faz a validação do logico com o fisico
rman target /
list archive log all;
# A = avaliable
# X = expired não tem haver com o tempo, tem haver com estar lá ou não 
crosscheck archive log all;
delete expired archive log all;
---
# Voltar um archive log para a localização 
catalog start with '/caminho/arquivo' # Não pode usar * ele é recursivo
```

##### 8. SNAPSHOT CONTROLFILE

```sql
rman target /
show all;
CONFIGURE SHAPSHOT CONTROLFILE NAME TO '/caminho/snapcf_ORCL.f';
# Toda vez que o RMAN faz um backup ele copia antes o controlfile para esse caminho
# Para recupera-lo é so copiar para os locais dos controlfiles originais 
# com o nome original

# SITUAÇÃO
# Perdeu todos os controlfiles
shutdown abort;
exit;
# Para saber o nome e caminho dos controlfiles, olhar no spfile ou pfile;
# Copiar snapcf_ORCL.f /oradata/control01.ctl
# Copiar snapcf_ORCL.f /FRA/control02.ctl
startup mount;
# Atualizar os controlfiles
recover database;
alter database open;
ou
alter database open resetslog;
list incarnation;
# Fazer o backup imediatamente
backup database;
```

##### 9. PARA QUE SERVE A OPÇÃO OPTIMIZATION ON

```sql
# Não faz backups do que não precisa (Os que ja foram feitos)
# Obedece a politica de retenção backup do RMAN
CONFIGURE RETENTION POLICY REDUNDANCY 1;
CONFIGURE BACKUP OPTIMIZATION ON;
```

