##### OBJETIVOS

```ini
- Minimizar tempo de resposta e recuperação de dados;
- Minimizar concorrência de acesso aos dados;
- Otimizar a taxa de transferência de dados;
- Otimizar a capacidade de carga do Banco de Dados
```

##### 1. APLICAÇÕES - 60% (SQL ruim)

```ini
SQL ruim: mal escrito;
Recursos serializados: Uma so CPU;
Recursos Multitarefas: Melhor usar o paralelismo;
```

##### 2. PROJETO - 20% (Modelagem ruim do banco)

```ini
Modelagem dos objetos: tabelas, visões, indices, sequences etc.;
Armazenamento dos objetos.
```

##### 3. BANCO DE DADOS - 17,5% (Configurações ruins) Tuning de Instancia

```ini
Alocação de dados em Memória;
Estrutura do Banco de Dados;
Parâmetros inapropriados.
```

##### 4. SISTEMA OPERACIONAL -2,5% (Configurações do SO)

```ini
I/O de dados;
Largura de banda;
Alocação de espaço em disco;
Swap;
Parâmetros do SO.
```

##### FERRAMENTAS PARA TUNING

```sql
# ARQUIVOS DE  LOGs
select * from v$parameter order by name;
background_dump_dest (caminho do Alert_SEAD.log)

# ARQUIVOS DE TRACES
user_dump_dest(caminho dos traces)

# Podemos habilitar os traces dos usuarios
ALTER SESSION SET SQLTRACE=true

# TABELAS & VIEWS DE SISTEMAS
select * from dba_tables;
select * from dba_tab_columns;
select * from dba_indexes;
select * from dba_indexstats; ? nome
---
select * from v$logfile;
select * from v$instance;
select * from v$database;
select * from v$session;
select * from v$parameter;

# INSTANCE/DATABASE
select * from v$database;
select * from v$instance;
select * from v$option;
select * from v$parameter;
select * from v$backup;
select * from v$px_process_sysstat;
select * from v$process;
select * from v$waitstat;
select * from v$system_event;

# MEMORY
select * from v$buffer_pool_statistics;
select * from v$db_object_cache;
select * from v$librarycache;
select * from v$rowcache;
select * from v$sysstat;
select * from v$sgastat;

# DISK
select * from v$datafile;
select * from v$filestat;
select * from v$log;
select * from v$log_history;
select * from v$dbfile;
select * from v$tempfile;
select * from v$tempstat;

# CONTENTION
select * from v$lock;
select * from v$rollname;
select * from v$rollstat;
select * from v$waitstat;
select * from v$latch;

# USER/SESSION
select * from v$lock;
select * from v$open_cursor;
select * from v$process;
select * from v$sort_usage;
select * from v$session;
select * from v$sesstat;
select * from v$transaction;
select * from v$session_event;
select * from v$session_wait;
select * from v$px_sesstat;
select * from v$px_session;
select * from v$session_object_cache;

T: Troubleshooting (Rastreando um problema)
T/P: Troubleshootin (Problema e Performace)

# CONSULTAS MAIS DEMORADAS
select sql_text, elapsed_time/1000000 elapsed_sec, executions, disk_reads, buffer_gets
from v$sqlarea
order by elapsed_time desc;

# 10 CONSULTAS MAIS DEMORADAS
SELECT *
  FROM (  SELECT ROUND ( ( (cpu_time / 1000000) / 60), 2) AS "Tempo total de CPU",
                 executions AS "Quant. exec.",
                 rows_processed AS "Quant. linhas proc.",
                 disk_reads AS "Leituras no disco",
                 first_load_time AS "Primeira utilização",
                 last_load_time AS "Última utilização",
                 parsing_schema_name AS "Usuário analisado",
                 sql_text AS "SQL exec."
            FROM v$sqlarea
           WHERE parsing_schema_name NOT IN ('SYS', 'SYSTEM', 'SYSMAN', 'DBSNMP')
        ORDER BY 1 DESC)
 WHERE ROWNUM <= 10;
```



##### ANALISANDO A SGA E PGA

```sql
OBS: Deve-se colocar 85% ou 60% da memoria do servidor dedicado para o Oracle dependendo do 
     Dessa memoria vai 80% para SGA e 20% para a PGA
     Ex: Servidor dedicado com 60G
     85% de 60G = 51G 
     80% de 51G = 41G SGA
     20% de 51G = 10G PGA
     
Análise de utilização da memória do banco de dados (SGA e PGA )

Para verificarmos a utilização de memória de nosso banco de dados e se sua utilização está ocorrendo de forma adequada, podemos seguir os passos abaixo, onde teremos uma visão inicial do comportamento e da utilização da memória interna do banco de dados.

1) Utilização da SGA
Uma das formas de verificação é analisarmos se está ocorrendo muito redimensionamento automático de memória.
Se isto estiver ocorrendo, pode ser necessário aumentarmos o tamanho da SGA.

Uma forma de avaliarmos esses redimensionamentos é com o SQL abaixo:

select component, oper_type, oper_mode, parameter, initial_size/1024/1024/1024 as initial_size_MB, final_size/1024/1024/1024 as final_size_MB, to_char(start_time,'dd/mm/yyyy  hh24:mi:ss') start_time, to_char(end_time,'dd/mm/yyyy  hh24:mi:ss') end_time 
from v$memory_resize_ops;

Analisando os resultados do select acima, percebemos que alguns componentes da SGA estão sofrendo redimensionamento e isto indica a possibilidade de aumento da SGA do banco de dados que estamos analisando.


2) utilização de PGA
A verificação na PGA, pode ser feita com o SQL abaixo, onde devemos verificar se o valor do campo "ESTD_OVERALLOC_COUNT" é maior que 0 (zero).
Se o valor for maior que zero, devemos aumentar o tamanho da PGA.

SELECT  round(PGA_TARGET_FOR_ESTIMATE/1024/1024)  target_mb, ESTD_PGA_CACHE_HIT_PERCENTAGE  cache_hit_perc,ESTD_OVERALLOC_COUNT 
FROM  V$PGA_TARGET_ADVICE;


 TARGET_MB CACHE_HIT_PERC ESTD_OVERALLOC_COUNT
-------------------- ------------------------------ --------------------------------------------
      1125                         92                          0
      2250                         92                          0
      4500                         92                          0
      6750                         92                          0
      9000                         92                          0
     10800                      100                          0
     12600                      100                          0
     14400                      100                          0
     16200                      100                          0
     18000                      100                          0
     27000                      100                          0
     36000                      100                          0
     54000                      100                          0
     72000                      100                          0



- O Campo TARGET_MB informa o valor da PGA;
- O Campo CACHE_HIT_PERC informa o percentual de utilização do cache da PGA;
- O Campo ESTD_OVERALLOC_COUNT informa se ocorreu "estouro" da PGA configurada atualmente;

Para validarmos se precisamos ou não aumentar a PGA, devemos fazer a seguinte comparação:

1) Da lista TRAGET_MB informada, qual o valor de PGA configurada para nosso ambiente?
2) Na coluna ESTD_OVERALLOC_COUNT correspondente ao nosso TRAGET_MB o valor é maior que zero?

Se a resposta para o item 2 acima for sim, devemos avaliar a possibilidade de aumentar a PGA.

```

