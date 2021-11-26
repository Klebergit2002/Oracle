#### GERENCIAMENTO DE MEMORIA 

O Oracle administra automaticamente a memória, devemos configurar poucos parâmetros para isso, como apenas o total que o Oracle poderá utilizar.

Estruturas principais PGA e SGA

```ini
# CONFIGURAR A PGA 
PGA_AGGREGATED_TARGET: É um soft limit. Se precisar de mais memória, o Oracle irá usar.
PGA_AGGREGATED_LIMIT: É um hard limit. É o máximo de memória que o Oracle pode utilizar para a PGA.
WORKAREA_SIZE_POLICY: Quando configurado em AUTO o Oracle irá atribuir memória as sessões enquanto tenta manter o total de memória da instância de acordo com o valor configurado no PGA_AGGREGATED_TARGET.

# CONFIGURAR A SGA
SGA_TARGET: Esse é o tamanho da memória alocada para a SGA. É um parâmetro dinâmico. Isso significa que podemos aumentar ou diminuir mesmo com o banco em execução. Mas nunca podemos aumentar mais do que o valor que esta em SGA_MAX_SIZE, e esse não é um parâmetro dinâmico.

:Ao configurar o tamanho da SGA e PGA com os parâmetros mencionados acima, estamos configurando o ASMM (Automatic Shared Memory Management).

-----

:Existe também uma outra forma do Oracle administrar a memória, que seria o AMM(Automatic Memory Management). Nesse modo, configuramos apenas um parâmetro, que seria o valor total disponível tanto para a SGA como para a PGA e o Oracle se encarrega de administrar corretamente o tamanho para ambas.

MEMORY_TARGET: Esse parâmetro é o tamanho de memória disponível para o Oracle. Esse parâmetro é dinâmico, podendo alterar com o banco funcionando. Mas nunca podemos aumentar mais do que o valor que esta configurado em MEMORY_MAX_SIZE, e esse não é dinâmico.

:Então, você pode deixar todos os parâmetros com o valor zero. E configurar apenas os dois últimos mencionados.

:Qual a melhor opção? A resposta é depende. Pela minha experiência entendo que o ASMM em ambientes com grande utilização de memória é melhor. Isso por que para utilizar o huge pages do linux devemos estar com ASSM, com AMM o huge pages não funciona. Em um post futuro irei falar um poucos mais sobre isso.

:Mas, qual o tamanho correto para o meu ambiente? Bem, o Oracle possui vários advisors que nos auxilia em escolher o tamanho ideal para a nossa situação. Essas informações são baseadas na atividade e performance do banco.

:Para verificar o que o Oracle recomenda, pode-se verificar essas informações no Enterprise Manager ou então através de algumas views.

# VERIFICANDO A SGA (ASMM) 
SELECT sga_size, sga_size_factor, estd_db_time, estd_physical_reads FROM V$SGA_TARGET_ADVICE;
 
SGA_SIZE: Tamanho imaginario da SGA
SGA_SIZE_FACTOR: Fator para o aumento da SGA
ESTD_DB_TIME: Tempo total para executar a carga do banco
ESTD_PHISICAL_READS: Leituras fisicas no disco

# VERIFICANDO A PGA (ASMM) 
SELECT pga_target_for_estimate, pga_target_factor, estd_time, estd_extra_bytes_rw FROM V$PGA_TARGET_ADVICE;

# CASO DO AMM INDICA O TOTAL DE MEMÓRIA NECESSÁRIA
SELECT * FROM V$MEMORY_TARGET_ADVICE;
```

#### CONFIGURANDO MEMORIA NO ORACLE 12c 

```sql
# AMM (Automatic Memory Management) 
# Deu problema com "sga_max_size"

show parameter target;
SQL> show parameter memory
SQL> show parameter sga
SQL> show parameter pga
SQL> show parameter spfile

SQL> create pfile from spfile;

# Vale lembrar que o memory_target = sga_target + pga_aggregate_target
alter system set memory_target = 5G scope=spfile; 
alter system set memory_max_target = 5G scope=spfile; 
---
alter system set sga_max_size = 0 scope=spfile; 
alter system set sga_target = 0 scope=spfile;   
alter system set pga_aggregate_target = 0 scope=spfile;
--alter system set pga_aggregate_limit = 0 scope=spfile; 
---
SQL> shutdown immediate;
SQL> startup;
SQL> show parameter target;

# Monitorando do gerenciamento automático da memória
SQL> select * from v$memory_target_advice order by memory_size;

# Verificar memoria/redece do Oracle
$ lpcs -m
$ netstat
```

```sql
# ASMM (Automatic Shared Memory Management) - AG esta usando esse

$ sqlplus /nolog
SQL> CONNECT / as sysdba
SQL> create pfile from spfile;
SQL> create spfile from pfile; # Resolve tambem
---
alter system set sga_target = 4G scope=spfile; 
alter system set sga_max_size = 4G scope=spfile; 
---
alter system set pga_aggregate_target = 2G scope=spfile; 
--Setar com o dobro de pga_aggregate_target
alter system set pga_aggregate_limit = 4G scope=spfile; 

# Cuidado com esse parametro
# alter system set pga_aggregate_limit = 4G scope=spfile;
---   
alter system set memory_max_target = 0 scope=spfile;
alter system set memory_target = 0 scope=spfile;
---
SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP
SQL> startup PFILE=     'C:\app\Administrador\virtual\product\12.2.0\dbhome_1\database\INITCLINICAS.ORA';
SQL> show parameter memory
SQL> show parameter sga

```

```sql
# MMM (Manual memory management) É o gerenciamento estático.
Após eu definir o valor da sga_target como 0 ele iniciou a minha instancia com apenas 256M, por que será que isso aconteceu? Eu continuo usando meu SPFILE normalmente.

SQL> show parameter pfile
SQL> show parameter sga
SQL> show parameter memory

SQL> alter system set sga_max_size=1056m scope=spfile;
SQL> alter system set db_cache_size=200m scope=spfile;
SQL> alter system set log_buffer=100m scope=spfile;
SQL> alter system set shared_pool_size=200m scope=spfile;
SQL> alter system set pga_aggregate_target=500m scope=spfile;
SQL> shut immediate;
SQL> startup;

SQL> show parameter sga;
SQL> show parameter pga;
SQL> show parameter cache;
SQL> show parameter shared;

```

##### GERENCIAMENTO AMM (TESTADO)

```sql
# GERENCIAMENTO DE MEMORIA (AMM)
# OBS: A mem compartilhad do S.O.nao deve 
# ser menor que memory_target e memory_max_target

# umount tmpfs
# mount -t tmpfs shmfs -o size=1500m /dev/shm
---
show parameter spfile
create pfile from spfile;

SELECT * FROM V$MEMORY_TARGET_ADVICE;
select * from v$memory_target_advice order by memory_size;
select * from v$parameter where name like '%sga%' or name like '%target%';
---
alter system set memory_target = 60G scope=spfile;
alter system set memory_max_target = 60G scope=spfile;
alter system set sga_max_size = 60G scope=spfile; 
---
alter system set sga_target = 0 scope=spfile; 
alter system set pga_aggregate_target = 0 scope=spfile; -- 20% da SGA
alter system set pga_aggregate_limit = 0 scope=spfile; --Setar com o dobro de pga_aggregate_target

```

