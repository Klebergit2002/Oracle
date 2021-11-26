##### AUMENTANDO O TAMANHO DOS REDOLOGS (BANCO CONFIGURADO SEM OMF)

##### ESTRATÉGIA: Criar novos grupos e apagar os antigos

```sql
# VERIFICAR OMF
show parameters db_create_file_dest
     C:\app\Administrator\virtual\oradata
show parameters db_recovery_file_dest;
     C:\app\Administrator\virtual\fast_recovery_area\producao

# REALIZANDO A TROCA DE SWITCHES
alter system switch logfile;
alter system checkpoint;
alter system checkpoint global;

# VERIFICANDO OS GRUPOS DE REDOLOG FILES
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;

C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\
O1_MF_4_J9G6H2CT_.LOG
O1_MF_5_J9G6J7FC_.LOG
O1_MF_6_J9G6JSD1_.LOG
C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\
O1_MF_4_J9G6HBH1_.LOG
O1_MF_5_J9G6JDDB_.LOG
O1_MF_6_J9G6JVW8_.LOG

# ADICIONANDO NOVOS GRUPOS DE REDOLOGS
alter database add logfile group 4 ('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO01a.LOG',
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO01b.LOG')   size 300M; 

alter database add logfile group 5 ('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO02a.LOG',
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO02b.LOG')   size 300M; 

alter database add logfile group 6 ('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO03a.LOG',
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO03b.LOG')   size 300M; 

# Observe que poderíamos criar esses arquivos em discos diferentes,
# garantindo que não haja perda em caso de danos em um dos discos.

# DROPANDO DATAFILES ANTIGOS
select group#, bytes/1024/1024 as MB, members, status from v$log ;

# Sempre verificar os grupos em qual status estão. 
# Se estiverem CURRENT (recebendo dados) ou 
# ACTIVE (gravando os dados nos archivelogs) não poderão ser deletados.

# APAGAR SOMENTE OS GRUPOS COM STATUS INACITVE
ALTER DATABASE DROP LOGFILE GROUP 1; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 2; --Inactive
ALTER DATABASE DROP LOGFILE GROUP 3; --Inactive

# LIMPEZA DE REDOLOGS
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP x
```

##### AUMENTANDO O TAMANHO DOS REDOLOGS (BANCO CONFIGURADO COM OMF)

##### ESTRATÉGIA: Criar novos grupos e apagar os antigos

```sql
# VERIFICAR OMF
show parameters db_create_file_dest
     C:\app\Administrator\virtual\oradata
show parameters db_recovery_file_dest;
     C:\app\Administrator\virtual\fast_recovery_area\producao

# VER O NOME E O TAMANHO 
select group#, bytes/1024/1024 as MB, members, status, archived from v$log;
select group#, type, member from v$logfile;

# CRIAR 3 NOVOS GRUPOS COM O TANNHO DESEJADO
# POR UTILIZAR O OMF, O BANCO CRIA OS REDOLOGS MULTIPLEXADOS NOS CAMINHOS
# INFORMADOS NOS PARAMETROS "DB_CREATE_FILE_DEST" E "DB_RECOVERY_FILE_DEST"
alter database add logfile group 4 size 300M;
alter database add logfile group 5 size 300M;
alter database add logfile group 6 size 300M;

# VERIFICAR QUAL REDO ESTÁ EM USO, SÓ É POSSIVEL EXCLUIR OS GRUPOS QUE ESTÃO
# COM STATUS DE INACTIVE OU UNUSED
select group#, status from v$log;
alter system switch logfile;
alter system checkpoint;
alter system checkpoint global;

# APAGAR SOMENTE OS GRUPOS COM STATUS INACITVE
alter database drop logfile group 1;
alter database drop logfile group 2;
alter database drop logfile group 3;
```

##### TRABALHANDO COM REDOLOGS 

```sql
# CAMINHOS DOS REDOLOGS
# E:\oradata\producao\dados\PRODUCAO\ONLINELOG\REDO01b.LOG
# C:\app\administrator\virtual\product\ORADATA\PRODUCAO
select * from v$log;
select * from v$logfile;
select group#, bytes/1024/1024 as MB, members, status from v$log ;
select * from v$history;

# ADICIONAR NOVO GRUPO
# TEM QUE SER SEMPRE O PROXIMO GRUPO, NÃO PODE REAPROVEITAR NUMEROS ANTERIORES
# OBS: NẪO APAGOU OS ARQUIVOS FISICAMENTE DO DISCO - APAGUEI MANUALMENTE

alter database add logfile group 7 
('C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO01a.LOG',
 'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO01b.LOG') 
 size 300M #REUSE; 

# ADICIONAR MEMBROS A UM GRUPO EXISTENTE
ALTER DATABASE ADD LOGFILE MEMBER 
'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\PRODUCAO\ONLINELOG\REDO01c.LOG' TO GROUP 7;  
ALTER DATABASE ADD LOGFILE MEMBER
'C:\APP\ADMINISTRATOR\VIRTUAL\FAST_RECOVERY_AREA\PRODUCAO\PRODUCAO\ONLINELOG\REDO01d.LOG' 
 TO GROUP 7;

# APAGAR GRUPO
# O membro do grupo que estiver CURRENT ou ACTIVE não pode ser apagado
alter database drop logfile group 1;
alter database drop logfile group 2;

# REALOCAR
shutdown immediate;
# Copiar os redos para a nova localização
startup mount;
alter database rename file 
'c:\app\Administrador\virtual\redolog\REDO01.LOG',
'c:\app\Administrador\virtual\redolog\REDO02.LOG', 
'c:\app\Administrador\virtual\redolog\REDO03.LOG' 
TO  
'c:\app\Administrador\virtual\oradataX\redologs\REDO01.LOG' ,
'c:\app\Administrador\virtual\oradataX\redologs\REDO02.LOG' ,
'c:\app\Administrador\virtual\oradataX\redologs\REDO03.LOG' ;
alter database open;
---
# Para utilizar os novos redo logs
alter system switch logfile;   # Algumas vezes para ativar os novos redo logs
```

