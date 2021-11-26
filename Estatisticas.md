##### ESTATISTICAS DE TRANSAÇÕES

```sql
select sum(value) total_transacoes from v$sysstat
 where name in ('user commits','user rollbacks');
  
select * from v$sysstat;

SELECT * FROM DBA_TAB_MODIFICATIONS 
WHERE table_owner = 'IPESAUDE' 
  and  to_date(timestamp) = to_date('04/05/2021','DD/MM/YYYY');

select max(table_owner), sum(inserts),sum(deletes),sum(updates) 
from dba_tab_modifications 
where table_owner='IPESAUDE' 
and to_date(timestamp) = to_date('04/05/2021','DD/MM/YYYY');

# TAMANHO DO BANCO DE DADOS
select to_char(sum(bytes)/power(1024,3),'999G999D999') TAMANHO_GB from dba_segments;
---
select tablespace_name,to_char(sum(bytes)/power(1024,3),'999G999D999') TAMANHO_GB 
  from dba_segments group by rollup(tablespace_name);
```

##### ARCHIVES POR DIA/HORA

```sql
# ARCHIVES POR DIA
select trunc(COMPLETION_TIME,'DD') Day, thread#, 
round(sum(BLOCKS*BLOCK_SIZE)/1048576) MB,count(*) Archives_Generated 
from v$archived_log 
where COMPLETION_TIME > sysdate-10 
group by trunc(COMPLETION_TIME,'DD'),
thread# order by 1,2;

# ARCHIVES POR HORA
alter session set nls_date_format='dd/mm/yyyy hh24:mi:ss';
select trunc(COMPLETION_TIME,'HH') Hour,thread# , round(sum(BLOCKS*BLOCK_SIZE)/1048576) MB,count(*) Archives 
from v$archived_log   
group by trunc(COMPLETION_TIME,'HH'),thread#  
order by 1;
```

##### ESTATÍSTICAS DO REDO LOG

```sql
select to_char(first_time,'DD/MM/YYYY') day,
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'00',1,0)),'999') "00",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'01',1,0)),'999') "01",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'02',1,0)),'999') "02",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'03',1,0)),'999') "03",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'04',1,0)),'999') "04",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'05',1,0)),'999') "05",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'06',1,0)),'999') "06",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'07',1,0)),'999') "07",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'08',1,0)),'999') "08",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'09',1,0)),'999') "09",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'10',1,0)),'999') "10",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'11',1,0)),'999') "11",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'12',1,0)),'999') "12",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'13',1,0)),'999') "13",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'14',1,0)),'999') "14",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'15',1,0)),'999') "15",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'16',1,0)),'999') "16",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'17',1,0)),'999') "17",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'18',1,0)),'999') "18",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'19',1,0)),'999') "19",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'20',1,0)),'999') "20",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'21',1,0)),'999') "21",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'22',1,0)),'999') "22",
  to_char(sum(decode(substr(to_char(first_time,'HH24'),1,2),'23',1,0)),'999') "23",
   sum(1) "TOTAL_IN_DAY"
  from v$log_history group by to_char(first_time,'DD/MM/YYYY')
  order by to_date(day) desc;
```

#### TAMANHO DO BANCO/SCHEMAS

```sql
# TAMANHO DO BANCO
select tablespace_name,to_char(sum(bytes)/power(1024,3),'999G999D999') TAMANHO_GB 
  from dba_segments group by rollup(tablespace_name);

# TAMANHO DA TABLE AGENDAMENTOS NA USERS
select segment_name,sum(bytes)/1024/1024/1024 GB from user_segments 
where segment_type='TABLE' 
and segment_name=upper('AGENDAMENTOS')
group by segment_name

# TAMANHO DOS SCHEMAS NA TABLESPACE USERS
select owner SCHEMA, to_char(sum(bytes)/1024/1024/1024,'999,999,999.999') GIGA_BYTES from dba_segments where tablespace_name = 'USERS' group by owner ;

# SCHEMAS POR TABLESPACE
select distinct tablespace_name,owner SCHEMA from dba_segments where owner in ('CAMPANHAMENSAGENS','PORTALBENEF','GESTAO','AGENDAIPES','GDPORTAL') order by 1 desc;

```

