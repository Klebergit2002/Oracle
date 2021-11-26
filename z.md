### RASCUNHOS

#### SOLICITAÇÕES SEAD

```sql
#ORACLE VKTM (schedule)
[root@orarac1 ~]# strace -p 6723

Process 6723 attached - interrupt to quit
gettimeofday({1323711237, 10495}, NULL) = 0
gettimeofday({1323711237, 10555}, NULL) = 0
nanosleep({0, 10000000}, {1323711237, 10495}) = 0
gettimeofday({1323711237, 21947}, NULL) = 0
gettimeofday({1323711237, 21999}, NULL) = 0
nanosleep({0, 10000000}, {1323711237, 21947}) = 0
# SOLICITAÇÕES DE ELVIS EM 21/10/2021
Se possível, gera um relatório com todos os usuários ativos do banco e quais as visões que eles tem acesso.
Não precisa listar os daqui do setor, apenas esses usuários externos
bisefaz, seed, transparencia...

# ATUALIZAR ESTATISTICAS DO ORACLE
sqlplus / as sysdba
@$ORACLE_HOME/rdbms/admim/utlbstat.sql

atualiza_estatisticas.sh
export ORACLE_SID=sead

sqlplus / as sysdba << SQL_SESSION
set timing on
begin
  for i in ( select username from dba_users where user_id > 49'RECAD' ) 
  loop
    dbms_stats.gather_schema_stats(ownname=> i.username);
  end loop; 
end;
/
exit;
/
set timing off
SQL_SESSION

# USUÁRIOS EXTERNOS
select * from dba_users where user_id >= 49 and username in ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP')
   and account_status = 'OPEN' order by username;

----------

# 1. VER QUEM TEM VIEWS EM DBA_VIEWS (PRINCIPAL): MOSTRA QUEM CRIOU AS VIEWS
SELECT owner,view_name FROM DBA_VIEWS WHERE OWNER IN ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP') ORDER BY 1,2;

# 2. COMPLEMENTAR COM DBA_TAB_PRIVS: MOSTRA TODAS AS PERMISSOES PARA O SCHEMA
SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE IN ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP')  
 AND SUBSTR(UPPER(TABLE_NAME),1,3) = 'VW_' ORDER BY 1,2;

# CONSULTAR EM DBA_OBJECTS OS DONOS DAS VIEWS (QUEM CRIOU = 1. ITEM)
SELECT * FROM DBA_OBJECTS WHERE OBJECT_TYPE = 'VIEW' 
   AND OWNER IN ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP'); 

# RELATORIO FINAL DAS VIEWS
SELECT owner DONO,view_name NOME_DA_VIEW FROM DBA_VIEWS 
WHERE OWNER IN ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP') UNION
SELECT GRANTEE DONO, TABLE_NAME NOME_DA_VIEW FROM DBA_TAB_PRIVS 
 WHERE GRANTEE IN ('BISEFAZ','EDOC','FHS','SEED','SERGIPEPREV','TCE','TRANSPARENCIA','WEBSFP','WEBSSP')  
 AND SUBSTR(UPPER(TABLE_NAME),1,3) = 'VW_' ORDER BY 1,2;

# VIEWS AUXILIARES
# UNION, UNION ALL, MINUS, INTERSECT
SELECT * FROM DICTIONARY;
SELECT * FROM DBA_CATALOG;
select * from dba_objects;
select * from dba_segments;
-
SELECT * FROM BISEFAZ.VW_CADASTRO;
SELECT * FROM BISEFAZ.VW_CADASTRO_BI_SEFAZ;
SELECT * FROM BISEFAZ.VW_CONTRA_CHEQUE;
SELECT * FROM BISEFAZ.VW_FINANCEIRO;
SELECT * FROM BISEFAZ.VW_FINANCEIRO_123;
SELECT * FROM BISEFAZ.VW_GRUPO_CARGO;
SELECT * FROM BISEFAZ.VW_IPES_SAUDE;
SELECT * FROM BISEFAZ.VW_OCORRENCIA;
SELECT * FROM BISEFAZ.VW_OCORRENCIA_BI_SEFAZ;
SELECT * FROM BISEFAZ.VW_ORGAO;
SELECT * FROM BISEFAZ.VW_RPPS;
SELECT * FROM BISEFAZ.VW_RUBRICA;
SELECT * FROM BISEFAZ.VW_SERVIDORES_REC_FGTS;
SELECT * FROM BISEFAZ.VW_SSP_INTRANET_POLICIAL;
--
SELECT * FROM FHS.VW_ACIDENTES_SEM_VITIMA;
SELECT * FROM FHS.VW_CIAT;
SELECT * FROM FHS.VW_SERVIDOR_VINCULO;
--
SELECT * FROM TRANSPARENCIA.VW_ATIVIDADE_FUNCAO;
SELECT * FROM TRANSPARENCIA.VW_CARGO;
SELECT * FROM TRANSPARENCIA.VW_CONTRA_CHEQUE_TEMP;
SELECT * FROM TRANSPARENCIA.VW_ESCOLARIDADE;
SELECT * FROM TRANSPARENCIA.VW_GRUPO_CARGO;
SELECT * FROM TRANSPARENCIA.VW_LOCAL;
SELECT * FROM TRANSPARENCIA.VW_MUNICIPIO;
SELECT * FROM TRANSPARENCIA.VW_NIVEL_SALARIAL;
SELECT * FROM TRANSPARENCIA.VW_ORGAO;
SELECT * FROM TRANSPARENCIA.VW_RUBRICA;
SELECT * FROM TRANSPARENCIA.VW_RUBRICA_CONTRA_CHEQUE;
SELECT * FROM TRANSPARENCIA.VW_SERVIDORES;
SELECT * FROM TRANSPARENCIA.VW_TRANSPARENCIA;
SELECT * FROM TRANSPARENCIA.VW_TRANSPARENCIA_RUBRICA;
SELECT * FROM TRANSPARENCIA.VW_TRANSP_ESTRUTURA_CARGO;
SELECT * FROM TRANSPARENCIA.VW_TRANSP_QUANTITATIVO_CARGO;
SELECT * FROM TRANSPARENCIA.VW_VINCULO;
SELECT * FROM TRANSPARENCIA.VW_VINCULO_VAGA;
--
SELECT * FROM WEBSSP.VW_SSP_INTRANET_CARGO;
SELECT * FROM WEBSSP.VW_SSP_INTRANET_POLICIAL;

```

### RSYNC

```bash
# BACKUP INCREMENTAL
$ rsync --help
$ rsync -hvrP /home/kleber/Arquivos /media/kleber/backup # Copia diretorio
$ rsync -hvrP /home/kleber/Arquivos/ /media/kleber/backup # Copia arquivos
$ rsync -hvP  /home/kleber/Arquivos/* /media/kleber/backup # Copia arquivos, não precisa do -r
$ rsync -hvaP /home/kleber/Arquivos/ /media/kleber/backup # Copia arquivos mantendo tudo da origem
----
# --delete deixa os 2 sempre iguais
$ rsync -hva --delete /home/kleber/Arquivos/ /media/kleber/backup 

# Comprime transfere e descomprime (Economia de banda) 
$ rsync -hvaz /home/kleber/Arquivos/ /media/kleber/backup 

# Não transfere os jpeg 
$ rsync -hva --exclude="*.jpeg" /home/kleber/Arquivos/ /media/kleber/backup 

# Transfere somente os *.txt (Observar a ordem do include, nesse caso include na frente) 
$ rsync -hva --include="*.txt" --exclude="*" /home/kleber/Arquivos/ /media/kleber/backup 

# Opção -u, se atualizar o destino ele não mexe, deixa como esta
$ rsync -hvau /home/kleber/Arquivos/ /media/kleber/backup 
-
# Transferencia remota via SSH
$ rsync -hvPa /home/kleber/Arquivos/ kleber@192.168.1.220:~kleber/backup 

```



```bash
# WHATS SEM CADASTRAR TELEFONE
55 (079) 99148-2001
https://api.whatsapp.com/send?phone=55079991482001
http://wa.me/55079991482001

# PROBLEMA CRA
MARCIO-Atendimento
https://crase.org.br/
https://sistemacrase.com.br
15598209500/xadrez2002
CRA: 1-1282
Rua Senador Rollembeg, 513 São José Aracaju 
@cra.crase 
Tel.(79) 3214-2229 
Funcionamento: Seg a qui: 8hàs17h Sex: 8hàs14h 

# LOGIN INFORMÁTICA
Vocês tem cabo de força trifásico tipo PC, 90 graus de 20 Amperes?

# IPES
Boa tarde Bruno,
Não sei se é possivel algum PEIXÃO ai do IPES me ajudar.
Meu filho como você sabe é paciente Oncológico, ele acabou de fazer um tratamento
de Quimioterapia, mas alguns tumores ainda resistiram a Quimio e êle vai precisar
de mais uma cirurgia Urgente. O médico já fez todas as soliçitações, liguei para
o IPES e me falaram que tem que dar entrada no Hospital e o próprio Hospital
providencia tudo.
O IPES é muito bom, mas tem um problema na demora da aquisição do material
para a cirurgia.
Como é uma cirurguia Urgente, se alguem ai puder me ajudar ficaria muito grato.

# IVETE SEGUROS 20/06/2021(Domingo)
Boa tarde Ivete, com relação ao Corolla, o Seguro mais o BONUS da Toyolex, fica praticamente irrecusável, pelo menos durante esse ano, infelizmente vou cancelar o nosso e fazer com eles.
No próximo ano eu entro em contato com você para continuarmos.
Agradeço pela compreensão.

# JOAO TOYOLEX - 20/06/2021(Domingo)
Boa tarde amigo,
Acho que vou colocar esse carro no nome de meu filho ou de minha esposa.
O que é preciso para fazer isso?
```

[Clinica Santa Helena](https://clinicasantahelena.com.br/) OK

```bash
# EXAMES FEITOS EM 29-Junho-2021
# RESULTADOS EM 30-Junho-2021
3361063 J9N700BZ Saiu incompleto
3361076 DWN430AK Saiu incompleto
3361082 5w8146IV Não saiu
```

### ESTRUTURA DE DADOS DO ORACLE

```ini
ORADATA
  CONTROLFILES
  DATAFILES
    SYSTEM
    SYSAUX
    USERS
    UNDOTBS
    TEMP
    DADOS_SISTEMAS
  ONLINELOG
```

```INI
FAST_RECOVERY_AREA - FRA
  ARCHIVELOG
  BACKUPSET
  AUTOBACKUP
  CONTROLFILE (MULTIPLEXADO)
  ONLINELOG (MULTIPLEXADO)
```

####  MOSTRA TODOS OS ARQUIVOS DO BANCO E PATH DA FRA E ARCHIVES

```sql
select name from v$controlfile union all
select name from v$tempfile union all
select name from v$dbfile union all
select member from v$logfile union all
select 'ARCHIVE_DEST = '||destination as name from V$ARCHIVE_DEST where dest_id = 1
union all
select 'RECOVERY_FILE_DEST = '||upper(name) as name from V$RECOVERY_FILE_DEST 
order by 1;

# SPFILE & PFILE
create pfile from spfile (mais usado)
create spfile from pfile (cuidado p/não desatualizar o spfile)
```

#### LOTERIAS/FAMILIA

```ini
*LFACIL-INDEP.2021*
DATA: *11/09/2021*
SITUAÇÃO: *Fechado*
PREMIO: *R$ 150.000.000*     
P/COTA: *R$ 2.631.579*

*FAM. BUSCAPÉ (19 Cotas)*
  (5) Kleber *(Isento)*
  (5) Renaura *(pg)*
  (2) Eduardo *(pg)*
  (2) Ericson *(pg)*
  (2) Rodrigo *(pg)*
  (1) Ana Célica *(pg)*
  (1) Angela *(pg)*
  (1) Dilma *(pg)*
  (1) Genica *(pg)*
  (1) Henrique *(pg)*
  (1) Paula *(pg)*
  (1) Rafael *(pg)*
  (1) Zezinho *(pg)*

*FAM. BARBOSA (10 Cotas)*
  (5) Marcos *(pg)*
  (5) Leticia *(pg)*

*FAM. MAIA (11 Cotas*)
  (3) Gracinha *(pg)*
  (3) Paulinho *(pg)*
  (1) Isac *(pg)*
  (1) Jorge *(pg)*
  (1) Ledice *(pg)*
  (1) Ricardo *(pg)*
  (1) Viviane *(pg)*

*TRABALHO (12 Cotas)*
  (5) Evancy-EMG *(pg)*
  (2) Jamison-SEAD *(pg)*
  (1) Anselmo-CC *(pg)*
  (1) Antonio-FHS *(pg)*
  (1) Joangele-SEAD *(pg)*
  (1) Jonatas-SEF *(pg)*
  (1) Mário-FHS *(pg)*

*ESTATÍSTICAS:*
  *100%* pagos de *R$ 520,00*
  *52* Cotas
  *29* Participantes
  *208* Cartões de 15 dezenas
  *R$ 520,00*

Pagamentos ate dia:
*06/09/2021-Segunda*
```

#### LOTERIAS/TRABALHO

```ini
*LFACIL 2327*
DATA: *20/09/2021*
SITUAÇÃO: *Fechado*
ESTIMATIVA: *R$ 1.500.000*     
POR COTA: *R$ 26.316*

*FAM. BUSCAPÉ (19 Cotas)*
  (5) Kleber *(Isento)*
  (5) Renaura *(pg)*
  (2) Eduardo *(pg)*
  (2) Ericson *(pg)*
  (2) Rodrigo *(pg)*
  (1) Ana Célica *(pg)*
  (1) Angela *(pg)*
  (1) Dilma *(pg)*
  (1) Genica *(pg)*
  (1) Henrique *(pg)*
  (1) Paula *(pg)*
  (1) Rafael *(pg)*
  (1) Zezinho *(pg)*

*FAM. BARBOSA (10 Cotas)*
  (5) Marcos *(pg)*
  (5) Leticia *(pg)*

*FAM. MAIA (11 Cotas*)
  (3) Gracinha *(pg)*
  (3) Paulinho *(pg)*
  (1) Isac *(pg)*
  (1) Jorge *(pg)*
  (1) Ledice *(pg)*
  (1) Ricardo *(pg)*
  (1) Viviane *(pg)*

*TRABALHO (12 Cotas)*
  (5) Evancy-EMG *(pg)*
  (2) Jamison-SEAD *(pg)*
  (1) Anselmo-CC *(pg)*
  (1) Antonio-FHS *(pg)*
  (1) Joangele-SEAD *(pg)*
  (1) Jonatas-SEF *(pg)*
  (1) Mário-FHS *(pg)*

*LFACIL INDEPENDENCIA*
CONCURSO: 2320 
DATA: 11/09/2021
RESULTADO:
01 02 03 05 06 09 12 13 15 17 21 22 23 24 25
ACERTOS:
15 acerto(s) de 11 ponto(s)
15 x 5,00 = R$ 75,00
Se todos concordarem essoal vou fazer um joguinho na Lotofacil com esse valor.

*LOTOFACIL-2327*
*RESULTADO*:
01 02 03 04 06 07 08 11 13 15 17 19 20 21 25
*ACERTOS*:
6 acerto(s) de 11 ponto(s)
1 acerto(s) de 12 ponto(s)
- 6 x 5,00 = R$ 30,00
-1 x 10,00 = R$ 10,00
TOTAL: R$ 40,00

Pessoal vamos fazer mais um joguinho de R$ 40,00

Boa tarde apostadores,
Fiz esse joguinho de 16 dezenas na Lotofacil que custa R$ 40,00
Boa sorte pra nós

*RESULTADO LFACIL-2339*
01-03-06-07-08-10-11-14-18-19-20-22-23-24-25
** **    ** **    **    ** ** ** **       **
Nada pra ninguem
```

#### LOTERIAS RESULTADOS

```
Bom dia Rafa,
Mudei o Netflix para testar uma nova modalidade de assinatura por cotas, que fica muito mais barato.
Tem um site que o pessoal vende o perfil por R$ 17,00
Eu cancelei a minha conta do Netflix e assinei essa para agente testar.
So tem um incoveniente, temos que usar somente um perfil, mas aqui agente não usa quase nada. 
Se ficar ruim agente volta para o plano anterior.

Teste ai pra ver se ta tudo certo.

Nosso Perfil: Kleber
No lugar do email coloque o telefone
Tel: 32988860585 
Senha: 4568gmc72

```

```plsql
# CONSULTA HENRIQUE IPES

SET SERVEROUTPUT ON
DECLARE 
   stmt VARCHAR2(4000);
   cnt  NUMBER(10);
BEGIN
  FOR tab IN (
          SELECT * FROM DBA_TAB_COLUMNS
           WHERE OWNER = 'PORTAL'
             AND DATA_TYPE IN ('NUMBER')
             AND TABLE_NAME NOT LIKE 'BIN%'
             AND COLUMN_NAME LIKE '%HANDLE%'
           )
  LOOP
    stmt := 'SELECT COUNT(*) FROM ' || tab.OWNER || '.' ||tab.TABLE_NAME ||' WHERE ' || tab.COLUMN_NAME || ' LIKE ''%181927%''';
    EXECUTE IMMEDIATE stmt INTO cnt;
    IF (cnt > 0) THEN
      DBMS_OUTPUT.PUT_LINE('tabela: ' || tab.TABLE_NAME || ', coluna: ' || tab.COLUMN_NAME);
    END IF;
  END LOOP;
END;

select * from SAUDEPRO.SAM_MATRICULA where cpf in ('15598209500','02546756540','19246452500');
select * from SAUDEPRO.SFN_DOCUMENTO where cnpjcpf = '02546756540';
select * from SAUDEPRO.TBL_BENEFICIARIOS  where cpf = '02546756540';
```

##### PROBLEMAS COM A MAQUINA DE CAFÉ

```ini
A maquina estava moendo e fazendo café normalmente quando em um certo momento na hora que apertei o botão de fazer o café apareceu no visor uma mensagem na cor vermelha, que mostrava uma chave e o numero 1.
Consultei o significado da mensagem no manual e para tentar corrigir esse erro ele aconselhou desligar e ligar por 2 ou 3 vezes, fiz isso no primeiro dia do erro e realmente a maquina voltou a funcionar corretamente durante todo o dia, mas no segundo dia apareceu o mesmo erro, fiz o procedimento indicado no manual, mas desta vez não resolveu.


```

##### COMPARAÇÃO DE SCHEMAS ORACLE

```sql
REM http://www.dbspecialists.com/files/scripts/compare_schemas.sql
REM
REM compare_schemas.sql
REM ===================
REM
REM This script is provided by Database Specialists, Inc. 
REM (http://www.dbspecialists.com) for individual use and not for sale. 
REM Database Specialists, Inc. does not warrant the script in any way 
REM and will not be responsible for any loss arising out of its use.
REM
REM Your feedback is welcome! Please send your comments about this script
REM to scriptfeedback@dbspecialists.com
REM
REM This script will compare two Oracle schemas and generate a report of
REM discrepencies. This script has been used against Oracle 7.3.4, 8.0.5,
REM and 8.1.7 databases, but it should also work with other versions. 
REM
REM Please note that the following schema object types and attributes are 
REM not compared by this script at this time:
REM
REM         cluster definitions
REM         comments on tables and columns
REM         nesting, partition, IOT, and temporary attributes of tables
REM         snapshots/materialized views, logs, and refresh groups
REM         foreign function libraries
REM         object types
REM         operators
REM         indextypes
REM         dimensions
REM         auditing information
REM         new schema attributes added for Oracle 9i
REM
REM Version 02-04-2002
REM

PROMPT
PROMPT Schema Comparison
PROMPT =================
PROMPT
PROMPT Run this script while connected to one Oracle schema. Enter the Oracle
PROMPT username, password, and SQL*Net / Net8 service name of a second schema.
PROMPT This script will compare the two schemas and generate a report of
PROMPT differences.
PROMPT
PROMPT A temporary database link and table will be created and dropped by 
PROMPT this script.
PROMPT

ACCEPT schema CHAR PROMPT "Enter username for remote schema: "
ACCEPT passwd CHAR PROMPT "Enter password for remote schema: " HIDE
ACCEPT tnssvc CHAR PROMPT "Enter SQL*Net / Net8 service for remote schema: "

PROMPT

ACCEPT report CHAR PROMPT "Enter filename for report output: "

SET FEEDBACK OFF
SET VERIFY   OFF

CREATE DATABASE LINK rem_schema CONNECT TO &schema IDENTIFIED BY &passwd
USING '&tnssvc';

SET TRIMSPOOL ON

SPOOL &report

SELECT SUBSTR (RPAD (TO_CHAR (SYSDATE, 'mm/dd/yyyy hh24:mi:ss'), 25), 1, 25) 
       "REPORT DATE AND TIME"
FROM   SYS.dual;

COL local_schema  FORMAT a35 TRUNC HEADING "LOCAL SCHEMA"
COL remote_schema FORMAT a35 TRUNC HEADING "REMOTE SCHEMA"

SELECT USER || '@' || C.global_name local_schema,
       A.username || '@' || B.global_name remote_schema
FROM   user_users@rem_schema A, global_name@rem_schema B, global_name C
WHERE  ROWNUM = 1;

SET PAGESIZE  9999
SET LINESIZE  250
SET FEEDBACK  1

SET TERMOUT OFF

PROMPT

REM Object differences
REM ==================

COL object_name FORMAT a30

PROMPT SUMMARY OF OBJECTS MISSING FROM LOCAL SCHEMA

SELECT   object_type, COUNT (*)
FROM
(
SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name)
FROM     user_objects@rem_schema
MINUS
SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name)
FROM     user_objects
)
GROUP BY object_type
ORDER BY object_type;

PROMPT SUMMARY OF EXTRANEOUS OBJECTS IN LOCAL SCHEMA

SELECT   object_type, COUNT (*)
FROM
(
SELECT   object_type, 
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name)
FROM     user_objects
WHERE    object_type != 'DATABASE LINK'
OR       object_name NOT LIKE 'REM_SCHEMA.%'
MINUS
SELECT   object_type, 
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name)
FROM     user_objects@rem_schema
)
GROUP BY object_type
ORDER BY object_type;

PROMPT OBJECTS MISSING FROM LOCAL SCHEMA

SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name) object_name
FROM     user_objects@rem_schema
MINUS
SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name) object_name
FROM     user_objects
ORDER BY object_type, object_name;

PROMPT EXTRANEOUS OBJECTS IN LOCAL SCHEMA

SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name) object_name
FROM     user_objects
WHERE    object_type != 'DATABASE LINK'
OR       object_name NOT LIKE 'REM_SCHEMA.%'
MINUS
SELECT   object_type,
         DECODE (object_type, 
                 'INDEX', DECODE (SUBSTR (object_name, 1, 5), 
                                  'SYS_C', 'SYS_C', object_name), 
                 'LOB',   DECODE (SUBSTR (object_name, 1, 7),
                                  'SYS_LOB', 'SYS_LOB', object_name),
                 object_name) object_name
FROM     user_objects@rem_schema
ORDER BY object_type, object_name;

PROMPT OBJECTS IN LOCAL SCHEMA THAT ARE NOT VALID

SELECT   object_name, object_type, status
FROM     user_objects
WHERE    status != 'VALID'
ORDER BY object_name, object_type;

REM Table differences
REM =================

PROMPT TABLE COLUMNS MISSING FROM ONE SCHEMA
PROMPT (NOTE THAT THIS REPORT DOES NOT LIST DISCREPENCIES IN COLUMN ORDER)

(
SELECT   table_name, column_name, 'Local' "MISSING IN SCHEMA"
FROM     user_tab_columns@rem_schema
WHERE    table_name IN
         (
         SELECT table_name
         FROM   user_tables
         )
MINUS
SELECT   table_name, column_name, 'Local' "MISSING IN SCHEMA"
FROM     user_tab_columns
)
UNION ALL
(
SELECT   table_name, column_name, 'Remote' "MISSING IN SCHEMA"
FROM     user_tab_columns
WHERE    table_name IN
         (
         SELECT table_name
         FROM   user_tables@rem_schema
         )
MINUS
SELECT   table_name, column_name, 'Remote' "MISSING IN SCHEMA"
FROM     user_tab_columns@rem_schema
)
ORDER BY 1, 2;

COL schema         FORMAT a15
COL nullable       FORMAT a8
COL data_type      FORMAT a9
COL data_length    FORMAT 9999 HEADING LENGTH
COL data_precision FORMAT 9999 HEADING PRECISION
COL data_scale     FORMAT 9999 HEADING SCALE
COL default_length FORMAT 9999 HEADING LENGTH_OF_DEFAULT_VALUE

PROMPT DATATYPE DISCREPENCIES FOR TABLE COLUMNS THAT EXIST IN BOTH SCHEMAS

(
SELECT   table_name, column_name, 'Remote' schema, 
         nullable, data_type, data_length, data_precision, data_scale,
         default_length
FROM     user_tab_columns@rem_schema
WHERE    (table_name, column_name) IN
         (
         SELECT table_name, column_name
         FROM   user_tab_columns
         )
MINUS
SELECT   table_name, column_name, 'Remote' schema, 
         nullable, data_type, data_length, data_precision, data_scale,
         default_length
FROM     user_tab_columns
)
UNION ALL
(
SELECT   table_name, column_name, 'Local' schema,
         nullable, data_type, data_length, data_precision, data_scale,
         default_length
FROM     user_tab_columns
WHERE    (table_name, column_name) IN
         (
         SELECT table_name, column_name
         FROM   user_tab_columns@rem_schema
         )
MINUS
SELECT   table_name, column_name, 'Local' schema,
         nullable, data_type, data_length, data_precision, data_scale,
         default_length
FROM     user_tab_columns@rem_schema
)
ORDER BY 1, 2, 3;

REM Index differences
REM =================

COL column_position FORMAT 999  HEADING ORDER

PROMPT INDEX DISCREPENCIES FOR INDEXES THAT EXIST IN BOTH SCHEMAS

(
SELECT   A.index_name, 'Remote' schema, A.uniqueness, A.table_name, 
         B.column_name, B.column_position
FROM     user_indexes@rem_schema A, user_ind_columns@rem_schema B
WHERE    A.index_name IN
         (
         SELECT index_name
         FROM   user_indexes
         )
AND      B.index_name = A.index_name
AND      B.table_name = A.table_name
MINUS
SELECT   A.index_name, 'Remote' schema, A.uniqueness, A.table_name, 
         B.column_name, B.column_position
FROM     user_indexes A, user_ind_columns B
WHERE    B.index_name = A.index_name
AND      B.table_name = A.table_name
)
UNION ALL
(
SELECT   A.index_name, 'Local' schema, A.uniqueness, A.table_name, 
         B.column_name, B.column_position
FROM     user_indexes A, user_ind_columns B
WHERE    A.index_name IN
         (
         SELECT index_name
         FROM   user_indexes@rem_schema
         )
AND      B.index_name = A.index_name
AND      B.table_name = A.table_name
MINUS
SELECT   A.index_name, 'Local' schema, A.uniqueness, A.table_name, 
         B.column_name, B.column_position
FROM     user_indexes@rem_schema A, user_ind_columns@rem_schema B
WHERE    B.index_name = A.index_name
AND      B.table_name = A.table_name
)
ORDER BY 1, 2, 6;

REM Constraint differences
REM ======================

PROMPT CONSTRAINT DISCREPENCIES FOR TABLES THAT EXIST IN BOTH SCHEMAS

SET FEEDBACK OFF

CREATE TABLE temp_schema_compare
(
database     NUMBER(1),
object_name  VARCHAR2(30),
object_text  VARCHAR2(2000),
hash_value   NUMBER
);

DECLARE
  CURSOR c1 IS
    SELECT constraint_name, search_condition
    FROM   user_constraints
    WHERE  search_condition IS NOT NULL;
  CURSOR c2 IS
    SELECT constraint_name, search_condition
    FROM   user_constraints@rem_schema
    WHERE  search_condition IS NOT NULL;
  v_constraint_name  VARCHAR2(30);
  v_search_condition VARCHAR2(32767);
BEGIN
  OPEN c1;
  LOOP
    FETCH c1 INTO v_constraint_name, v_search_condition;
    EXIT WHEN c1%NOTFOUND;
    v_search_condition := SUBSTR (v_search_condition, 1, 2000);
    INSERT INTO temp_schema_compare
    (
    database, object_name, object_text
    )
    VALUES
    (
    1, v_constraint_name, v_search_condition
    );
  END LOOP;
  CLOSE c1;
  OPEN c2;
  LOOP
    FETCH c2 INTO v_constraint_name, v_search_condition;
    EXIT WHEN c2%NOTFOUND;
    v_search_condition := SUBSTR (v_search_condition, 1, 2000);
    INSERT INTO temp_schema_compare
    (
    database, object_name, object_text
    )
    VALUES
    (
    2, v_constraint_name, v_search_condition
    );
  END LOOP;
  CLOSE c2;
  COMMIT;
END;
/

SET FEEDBACK 1

(
SELECT   REPLACE (TRANSLATE (A.constraint_name,'012345678','999999999'), 
                  '9', NULL) constraint_name,
         'Remote' schema, A.constraint_type, A.table_name,
         A.r_constraint_name, A.delete_rule, A.status, B.object_text
FROM     user_constraints@rem_schema A, temp_schema_compare B
WHERE    A.table_name IN
         (
         SELECT table_name
         FROM   user_tables
         )
AND      B.database (+) = 2
AND      B.object_name (+) = A.constraint_name
MINUS
SELECT   REPLACE (TRANSLATE (A.constraint_name,'012345678','999999999'),
                  '9', NULL) constraint_name,
         'Remote' schema, A.constraint_type, A.table_name,
         A.r_constraint_name, A.delete_rule, A.status, B.object_text
FROM     user_constraints A, temp_schema_compare B
WHERE    B.database (+) = 1
AND      B.object_name (+) = A.constraint_name
)
UNION ALL
(
SELECT   REPLACE (TRANSLATE (A.constraint_name,'012345678','999999999'),
                  '9', NULL) constraint_name,
         'Local' schema, A.constraint_type, A.table_name,
         A.r_constraint_name, A.delete_rule, A.status, B.object_text
FROM     user_constraints A, temp_schema_compare B
WHERE    A.table_name IN
         (
         SELECT table_name
         FROM   user_tables@rem_schema
         )
AND      B.database (+) = 1
AND      B.object_name (+) = A.constraint_name
MINUS
SELECT   REPLACE (TRANSLATE (A.constraint_name,'012345678','999999999'),
                  '9', NULL) constraint_name,
         'Local' schema, A.constraint_type, A.table_name,
         A.r_constraint_name, A.delete_rule, A.status, B.object_text
FROM     user_constraints@rem_schema A, temp_schema_compare B
WHERE    B.database (+) = 2
AND      B.object_name (+) = A.constraint_name
)
ORDER BY 1, 4, 2;

REM Database link differences
REM =========================

PROMPT DATABASE LINK DISCREPENCIES

COL db_link FORMAT a40

(
SELECT   db_link, 'Remote' schema, username, host
FROM     user_db_links@rem_schema
MINUS
SELECT   db_link, 'Remote' schema, username, host
FROM     user_db_links
)
UNION ALL
(
SELECT   db_link, 'Local' schema, username, host
FROM     user_db_links
WHERE    db_link NOT LIKE 'REM_SCHEMA.%'
MINUS
SELECT   db_link, 'Local' schema, username, host
FROM     user_db_links@rem_schema
)
ORDER BY 1, 2;

REM Sequence differences
REM ====================

PROMPT SEQUENCE DISCREPENCIES

(
SELECT   sequence_name, 'Remote' schema, min_value, max_value, 
         increment_by, cycle_flag, order_flag, cache_size
FROM     user_sequences@rem_schema
MINUS
SELECT   sequence_name, 'Remote' schema, min_value, max_value, 
         increment_by, cycle_flag, order_flag, cache_size
FROM     user_sequences
)
UNION ALL
(
SELECT   sequence_name, 'Local' schema, min_value, max_value, 
         increment_by, cycle_flag, order_flag, cache_size
FROM     user_sequences
MINUS
SELECT   sequence_name, 'Local' schema, min_value, max_value, 
         increment_by, cycle_flag, order_flag, cache_size
FROM     user_sequences@rem_schema
)
ORDER BY 1, 2;

REM Private synonym differences
REM ===========================

PROMPT PRIVATE SYNONYM DISCREPENCIES

(
SELECT   synonym_name, 'Remote' schema, table_owner, table_name, db_link
FROM     user_synonyms@rem_schema
MINUS
SELECT   synonym_name, 'Remote' schema, table_owner, table_name, db_link
FROM     user_synonyms
)
UNION ALL
(
SELECT   synonym_name, 'Local' schema, table_owner, table_name, db_link
FROM     user_synonyms
MINUS
SELECT   synonym_name, 'Local' schema, table_owner, table_name, db_link
FROM     user_synonyms@rem_schema
)
ORDER BY 1, 2;

REM PL/SQL differences
REM ==================

PROMPT SOURCE CODE DISCREPENCIES FOR PACKAGES, PROCEDURES, AND FUNCTIONS
PROMPT THAT EXIST IN BOTH SCHEMAS

SELECT   name, type, COUNT (*) discrepencies
FROM
(
(
SELECT   name, type, line, text
FROM     user_source@rem_schema
WHERE    (name, type) IN
         (
         SELECT object_name, object_type
         FROM   user_objects
         )
MINUS
SELECT   name, type, line, text
FROM     user_source
)
UNION ALL
(
SELECT   name, type, line, text
FROM     user_source
WHERE    (name, type) IN
         (
         SELECT object_name, object_type
         FROM   user_objects@rem_schema
         )
MINUS
SELECT   name, type, line, text
FROM     user_source@rem_schema
)
)
GROUP BY name, type
ORDER BY name, type;

PROMPT SOURCE CODE DISCREPENCIES FOR PACKAGES, PROCEDURES, AND FUNCTIONS
PROMPT THAT EXIST IN BOTH SCHEMAS (CASE INSENSITIVE COMPARISON)

SELECT   name, type, COUNT (*) discrepencies
FROM
(
(
SELECT   name, type, line, UPPER (text)
FROM     user_source@rem_schema
WHERE    (name, type) IN
         (
         SELECT object_name, object_type
         FROM   user_objects
         )
MINUS
SELECT   name, type, line, UPPER (text)
FROM     user_source
)
UNION ALL
(
SELECT   name, type, line, UPPER (text)
FROM     user_source
WHERE    (name, type) IN
         (
         SELECT object_name, object_type
         FROM   user_objects@rem_schema
         )
MINUS
SELECT   name, type, line, UPPER (text)
FROM     user_source@rem_schema
)
)
GROUP BY name, type
ORDER BY name, type;

REM Trigger differences
REM ===================

PROMPT TRIGGER DISCREPENCIES

SET FEEDBACK OFF

TRUNCATE TABLE temp_schema_compare;

DECLARE
  CURSOR c1 IS
    SELECT trigger_name, trigger_body
    FROM   user_triggers;
  CURSOR c2 IS
    SELECT trigger_name, trigger_body
    FROM   user_triggers@rem_schema;
  v_trigger_name VARCHAR2(30);
  v_trigger_body VARCHAR2(32767);
  v_hash_value   NUMBER;
BEGIN
  OPEN c1;
  LOOP
    FETCH c1 INTO v_trigger_name, v_trigger_body;
    EXIT WHEN c1%NOTFOUND;
    v_trigger_body := REPLACE (v_trigger_body, ' ', NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(9), NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(10), NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(13), NULL);
    v_trigger_body := UPPER (v_trigger_body);
    v_hash_value := dbms_utility.get_hash_value (v_trigger_body, 1, 65536);
    INSERT INTO temp_schema_compare (database, object_name, hash_value)
    VALUES (1, v_trigger_name, v_hash_value);
  END LOOP;
  CLOSE c1;
  OPEN c2;
  LOOP
    FETCH c2 INTO v_trigger_name, v_trigger_body;
    EXIT WHEN c2%NOTFOUND;
    v_trigger_body := REPLACE (v_trigger_body, ' ', NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(9), NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(10), NULL);
    v_trigger_body := REPLACE (v_trigger_body, CHR(13), NULL);
    v_trigger_body := UPPER (v_trigger_body);
    v_hash_value := dbms_utility.get_hash_value (v_trigger_body, 1, 65536);
    INSERT INTO temp_schema_compare (database, object_name, hash_value)
    VALUES (2, v_trigger_name, v_hash_value);
  END LOOP;
  CLOSE c2;
END;
/

SET FEEDBACK 1

(
SELECT   A.trigger_name, 'Local' schema, A.trigger_type,
         A.triggering_event, A.table_name, SUBSTR (A.referencing_names, 1, 30)
         referencing_names, SUBSTR (A.when_clause, 1, 30) when_clause,
         A.status, B.hash_value
FROM     user_triggers A, temp_schema_compare B
WHERE    B.object_name (+) = A.trigger_name
AND      B.database (+) = 1
AND      A.table_name IN
         (
         SELECT table_name
         FROM   user_tables@rem_schema
         )
MINUS
SELECT   A.trigger_name, 'Local' schema, A.trigger_type,
         A.triggering_event, A.table_name, SUBSTR (A.referencing_names, 1, 30)
         referencing_names, SUBSTR (A.when_clause, 1, 30) when_clause,
         A.status, B.hash_value
FROM     user_triggers@rem_schema A, temp_schema_compare B
WHERE    B.object_name (+) = A.trigger_name
AND      B.database (+) = 2
)
UNION ALL
(
SELECT   A.trigger_name, 'Remote' schema, A.trigger_type,
         A.triggering_event, A.table_name, SUBSTR (A.referencing_names, 1, 30)
         referencing_names, SUBSTR (A.when_clause, 1, 30) when_clause,
         A.status, B.hash_value
FROM     user_triggers@rem_schema A, temp_schema_compare B
WHERE    B.object_name (+) = A.trigger_name
AND      B.database (+) = 2
AND      A.table_name IN
         (
         SELECT table_name
         FROM   user_tables
         )
MINUS
SELECT   A.trigger_name, 'Remote' schema, A.trigger_type,
         A.triggering_event, A.table_name, SUBSTR (A.referencing_names, 1, 30)
         referencing_names, SUBSTR (A.when_clause, 1, 30) when_clause,
         A.status, B.hash_value
FROM     user_triggers A, temp_schema_compare B
WHERE    B.object_name (+) = A.trigger_name
AND      B.database (+) = 1
)
ORDER BY 1, 2, 5, 3;

REM View differences
REM ================

PROMPT VIEW DISCREPENCIES

SET FEEDBACK OFF

TRUNCATE TABLE temp_schema_compare;

DECLARE
  CURSOR c1 IS
    SELECT view_name, text
    FROM   user_views;
  CURSOR c2 IS
    SELECT view_name, text
    FROM   user_views@rem_schema;
  v_view_name    VARCHAR2(30);
  v_text         VARCHAR2(32767);
  v_hash_value   NUMBER;
BEGIN
  OPEN c1;
  LOOP
    FETCH c1 INTO v_view_name, v_text;
    EXIT WHEN c1%NOTFOUND;
    v_text := REPLACE (v_text, ' ', NULL);
    v_text := REPLACE (v_text, CHR(9), NULL);
    v_text := REPLACE (v_text, CHR(10), NULL);
    v_text := REPLACE (v_text, CHR(13), NULL);
    v_text := UPPER (v_text);
    v_hash_value := dbms_utility.get_hash_value (v_text, 1, 65536);
    INSERT INTO temp_schema_compare (database, object_name, hash_value)
    VALUES (1, v_view_name, v_hash_value);
  END LOOP;
  CLOSE c1;
  OPEN c2;
  LOOP
    FETCH c2 INTO v_view_name, v_text;
    EXIT WHEN c2%NOTFOUND;
    v_text := REPLACE (v_text, ' ', NULL);
    v_text := REPLACE (v_text, CHR(9), NULL);
    v_text := REPLACE (v_text, CHR(10), NULL);
    v_text := REPLACE (v_text, CHR(13), NULL);
    v_text := UPPER (v_text);
    v_hash_value := dbms_utility.get_hash_value (v_text, 1, 65536);
    INSERT INTO temp_schema_compare (database, object_name, hash_value)
    VALUES (2, v_view_name, v_hash_value);
  END LOOP;
  CLOSE c2;
END;
/

SET FEEDBACK 1

(
SELECT   A.view_name, 'Local' schema, B.hash_value
FROM     user_views A, temp_schema_compare B
WHERE    B.object_name (+) = A.view_name
AND      B.database (+) = 1
AND      A.view_name IN
         (
         SELECT view_name
         FROM   user_views@rem_schema
         )
MINUS
SELECT   A.view_name, 'Local' schema, B.hash_value
FROM     user_views@rem_schema A, temp_schema_compare B
WHERE    B.object_name (+) = A.view_name
AND      B.database (+) = 2
)
UNION ALL
(
SELECT   A.view_name, 'Remote' schema, B.hash_value
FROM     user_views@rem_schema A, temp_schema_compare B
WHERE    B.object_name (+) = A.view_name
AND      B.database (+) = 2
AND      A.view_name IN
         (
         SELECT view_name
         FROM   user_views
         )
MINUS
SELECT   A.view_name, 'Remote' schema, B.hash_value
FROM     user_views A, temp_schema_compare B
WHERE    B.object_name (+) = A.view_name
AND      B.database (+) = 1
)
ORDER BY 1, 2;

REM Job queue differences
REM =====================

COL what     FORMAT a30
COL interval FORMAT a30

PROMPT JOB QUEUE DISCREPENCIES

(
SELECT   what, interval, 'Remote' schema
FROM     user_jobs@rem_schema
MINUS
SELECT   what, interval, 'Remote' schema
FROM     user_jobs
)
UNION ALL
(
SELECT   what, interval, 'Local' schema
FROM     user_jobs
MINUS
SELECT   what, interval, 'Local' schema
FROM     user_jobs@rem_schema
)
ORDER BY 1, 2, 3;

REM Privilege differences
REM =====================

PROMPT OBJECT-LEVEL GRANT DISCREPENCIES

(
SELECT   owner, table_name, 'Remote' schema, grantee, privilege, grantable
FROM     user_tab_privs@rem_schema
WHERE    (owner, table_name) IN
         (
         SELECT owner, object_name
         FROM   all_objects
         )
MINUS
SELECT   owner, table_name, 'Remote' schema, grantee, privilege, grantable
FROM     user_tab_privs
)
UNION ALL
(
SELECT   owner, table_name, 'Local' schema, grantee, privilege, grantable
FROM     user_tab_privs
WHERE    (owner, table_name) IN
         (
         SELECT owner, object_name
         FROM   all_objects@rem_schema
         )
MINUS
SELECT   owner, table_name, 'Local' schema, grantee, privilege, grantable
FROM     user_tab_privs@rem_schema
)
ORDER BY 1, 2, 3;

PROMPT SYSTEM PRIVILEGE DISCREPENCIES

(
SELECT   privilege, 'Remote' schema, admin_option
FROM     user_sys_privs@rem_schema
MINUS
SELECT   privilege, 'Remote' schema, admin_option
FROM     user_sys_privs
)
UNION ALL
(
SELECT   privilege, 'Local' schema, admin_option
FROM     user_sys_privs
MINUS
SELECT   privilege, 'Local' schema, admin_option
FROM     user_sys_privs@rem_schema
)
ORDER BY 1, 2;

PROMPT ROLE PRIVILEGE DISCREPENCIES

(
SELECT   granted_role, 'Remote' schema, admin_option, default_role, os_granted
FROM     user_role_privs@rem_schema
MINUS
SELECT   granted_role, 'Remote' schema, admin_option, default_role, os_granted
FROM     user_role_privs
)
UNION ALL
(
SELECT   granted_role, 'Local' schema, admin_option, default_role, os_granted
FROM     user_role_privs
MINUS
SELECT   granted_role, 'Local' schema, admin_option, default_role, os_granted
FROM     user_role_privs@rem_schema
)
ORDER BY 1, 2;

SPOOL OFF

SET TERMOUT ON

PROMPT
PROMPT Report output written to &report

SET FEEDBACK OFF

DROP TABLE temp_schema_compare;
DROP DATABASE LINK rem_schema;

SET FEEDBACK 6
SET PAGESIZE 20

```

