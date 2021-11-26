##### MOVENDO CONTROLFILES PARA OUTRO DISCO

```sql
# TAMANHO DO CONTROLFILE
#Para diminuir o tamnho temos que recria-lo
show parameter control_file_record_keep_time

SELECT name FROM v$controlfile;
c1 'E:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL01.CTL'
c2 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\CONTROLFILES\CONTROL02.CTL'
c3 'E:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL03.CTL'

# TRANSFERIR C3 PARA DISCO D:
# novo_c3 'D:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL03.CTL'

ALTER SYSTEM SET CONTROL_FILES=c1,c2,novo_c3 scope=spfile;

SQL> shutdown immediate;
SQL> exit
$ mv c3 novo_c3
$ sqlplus / as sysdba
SQL> startup
---
```

##### MOVENDO CONTROLFILES PARA OUTRO DISCO UTILIZANDO O RMAN

```sql
# ADICIONANDO MAIS UMA COPIA DE CONTROLFILE
SELECT name FROM v$controlfile;
c1 'E:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL01.CTL'
c2 'C:\APP\ADMINISTRATOR\VIRTUAL\PRODUCT\ORADATA\PRODUCAO\CONTROLFILES\CONTROL02.CTL'
c3 'D:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL03.CTL'

# ADICIONANDO C4
c4 'F:\ORADATA\PRODUCAO\CONTROLFILES\CONTROL04.CTL'
ALTER SYSTEM SET CONTROL_FILES=c1,c2,c3,c4 scope=spfile;

SQL> shutdown immediate;
SQL> startup nomount
SQL> exit
$ rman target /
RMAN> restore controlfile from c1;
RMAN> alter database mount;
RMAN> alter database open;
exit;
```

##### CRIANDO UM NOVO CONTROLFILES EM OUTRO DISCO 

```sql 
SELECT name FROM v$controlfile;
adrci

create controlfile
set database orcl
logfile controlnew1   
('\oracle\oradata\orcl\controlnew1.log’,
'\oracle\oradata\orcl\controlnew1.log’)
resetlogs;
---
RMAN> ALTER DATABASE BACKUP CONTROLFILE TO TRACE;
RMAN> ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS 'C:\B\control.trc';
RMAN> ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS 'C:\DBA-Kleber\control-clinicas.tx';
```
