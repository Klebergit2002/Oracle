### PENDENCIAS

Novas pendências

- [ ] Criar rotina para apagar arquivos expirados de log do backup

  ```bat
  # LISTAR ARQUIVOS COM MAIS DE 30 DIAS
  forfiles -p "C:\app\Administrator\virtual\scripts\log_backup" -s -d -30 -m *.log -c "cmd /c dir /q @path"
  
  # APAGAR ARQUIVOS COM MAIS DE 30 DIAS
  forfiles -p "C:\app\Administrator\virtual\scripts\log_backup" -s -d -30 -m *.log -c "cmd /c del /f /q @path"
  ```

  

- [ ] Observar pastas vazias na FRA  # delete archivelog from time 'SYSDATE-10'

- [ ] Testar Restore por data e hora 

- [ ] Estudar o Atom

Pendências Antigas

- [ ] Criar tabela (**Organization external**) para o alert.log

- [ ] Script de backup com Politica de Retenção para 7 dias

- [x] Verificar HOMO_AG crescimento dos Backups e Archivelogs

- [ ] Verificar minhas férias na EMGETIS

- [x] FAZER O TRANSPLANTE DO DEBIAN11 PARA MEU  SSD WD (Não deu certo)

- [x] CONFIGURE RMAN OUTPUT TO KEEP FOR 7 DAYS;  (Não achei)

- [x] CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; (Não achei)

- [x] CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default

- [x] CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default

- [ ] CONFIGURAR O ORACLE PARA INICIAR E DESLIGAR AUTOMATICAMENTE

  ```bash
  C:\oracle9i\bin\oradim  -startup -sid ORCL92  ?usrpwd manager
  -starttype SRVC,INST -pfile C:\oracle9i\admin\ORCL92\pfile\init.ora
  
  C:\oracle9i\bin\oradim -shutdown -sid ORCL92  -shutttype SRVC,INST
   'shutmode A
  
  # Registry:
  # Set the registry.  Navigate to the key:
  HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\oracle_home_name and set the key ORA_SID_AUTOSTART=true
  ------
  For automatic startup on Windows, set registry parameter
  ORA_SID_AUTOSTART to true using an Oracle Database tool such as ORADIM.
  Enter the following with parameters at the command prompt:
  C:\> oradim options
  
  To start the listener automatically, set services startup type to automatic.
  
  For automatic shutdown on Windows, set registry parameters
  ORA_SHUTDOWN and
  ORA_SID_SHUTDOWN to stop the relevant OracleServiceSID and shut down.
  Set registry parameter ORA_SID_SHUTDOWNTYPE to control shutdown mode (default is i, or immediate).
  ```
  
  

### APPLE IT 

#### ATIVIDADES OUTUBRO/2021

```ini
# TAREFAS DIÁRIAS
- Manutenção das Instância AG, CLINICAS E ITABS
- Acompanhamento dos Backups DATAPUMP de AG, CLINICAS E ITABS
- Manutenção e Acompanhamento dos Backups RMAN de AG e CLINICAS

# BACKUPS DOS BANCOS PARA UM SERVIDOR DA REDE
- Backup diário de CLINICAS funcionando perfeito
- Backup diário de AG em fase de testes e ajustes
- Backup diário de ITABS em fase de testes 
- Sugestão: fazer backups com mais frequencia dos ARCHIVES

# ATUALIZAÇÃO DE SENHA EM ITABS-PRODUÇÃO
- ALTER USER PORTALBENEF ACCOUNT UNLOCK;
- Alteração do PROFILE DEFAULT - Senha nunca expira
  ALTER PROFILE DEFAULT LIMIT 
  PASSWORD_GRACE_TIME UNLIMITED
  PASSWORD_LIFE_TIME UNLIMITED;
  
# PROBLEMA DE ESPAÇO EM ITABS-PRODUÇÃO
- Tablespace USERS com datafile no limite máximo 32G
- Solução: Criação e adição de mais um datafile ao USERS  

# SENHA EXPIRADA EM AG-HOMOLOGAÇÃO 
- USER SSERVER

# PROBLEMA DE LENTIDAO EM AG-PRODUÇÃO
- Parcialmente resolvido, ainda em observação.
- Procedimentos executados:
	- Geração de novas estatísticas (Participação DBA da Benner) em:
  	  SAUDEPRO.SAM_AUTORIZ; e
  	  SAUDEPRO.WEB_AUTORIZ.
    - Processo SYSMAN demorando muito resolvido com a limpesa da RECYCLEBIN 
      em todos os Schemas de AG;
    - Desenvolvimento de Scripts para refazer os indices de SAM_AUTORIZ e WEB_AUTORIZ
      na tentativa de melhorar a performance dos acessos.

```

#### ATIVIDADES SETEMBRO/2021

```ini
# TAREFAS DIÁRIAS
- Manutenção das Instância AG, CLINICAS E ITABS
- Acompanhamento dos Backups DATAPUMP de AG, CLINICAS E ITABS
- Manutenção e Acompanhamento dos Backups RMAN de AG e CLINICAS

# NOVA POLÍTICA DE BACKUP EM AG-PRODUÇÃO
- Backup FULL: Domingo e Quarta 
- Backup INCREMENTAL: Segunda, Terça, Quinta, Sexta e Sábado

# TESTE DE BANCO STANDBY-MANUAL EM CLINICAS-LAB 
Restauração do backup RMAN em Clinicas-LAB
Copia diária dos archivelogs de CLINICAS-PROD para CLINICAS-LAB
Recuperação diária dos archivelogs em CLINICAS-LAB
Teste bem sucedido

# NOVA INSTANCIA ITABS PARA LABORATORIO E STANDBY-MANUAL
- Instalação do Oracle12c
- Criação de scripts de Importação e Exportação
- Criação dos scripts RMAN
- Habilitar Archivelog
- Testes de impdp DATAPUMP dos schemas de Itabs
- Preparação e testes em ITABS-LAB para Standby manual

# TESTES EM LABORATÓRIO (LAB)
- Testes de Restore RMAN de AG Produção em AG LAB
- Testes de impdp das Instancias de Produção em LAB 

# NOVA PARADA EM ITABS 
- Programação para a segunda parada em ITABS - habilitar o modo archive
- Mudança do backup logico de ITABS para o novo disco

# ATUALIZAÇÃO DE SENHA EM AG-HOMOLOGAÇÃO 
- Redefinição da senha de SYSTEM expirada 
- Redefinição da senha de SYS 

```

#### ATIVIDADES AGOSTO/2021

```ini
# TESTES EM CLINICAS HOMOLOGAÇÃO
Criação de uma FRA
Criação de mais um controlfile na FRA
Criação de mais um grupo de redolog na FRA
Aumento de memória SGA=5G e PGA=2G

# TESTES EM LABORATÓRIO (LAB)
Criação da instância CLINICAS para testes
Backup/Recover fisico e lógico de AG/CLINICAS

Acompanhamento dos backups em AG e CLINICAS
Adequação da politica de backup em AG e CLINICA
Implementação de Compressão e paralelismo no DataPump de CLINICAS

Senha de SYS expirada em ITABS
Solução: Redefinição da senha de SYS, utilizando o orapwd.exe

Programação para a parada em ITABS
Parada no banco ITABS para habilitar o modo archive
Erro no shutdown immediate
Erro na ativação dos archives
Conseguimos subir o banco mesmo com esses problemas
Estamos verificando esses Erros para programar uma nova parada

Mudança do backup logico de ITABS  para o novo disco

```

####  ATIVIDADES JULHO/2021

```ini
# DIA 14/07/2021
- Inclusão de discos de 200G para armazenamento dos backups dos Bancos
- Novo levantamento para adequar os backups aos novos discos
- Reunião para definir a possibilidade de transferir os bancos AG e CLINICAS para o
  Datacenter da EMGETIS
- Configuração do modo ARCHIVELOG para o banco AG 
- Configuração da FRA no disco de 200G
- Habilitando o backup do RMAN
- Primeiro Backup FULL
- Backups INCREMENTAIS
- Acompanhamento diário do crecimento dos backups na FRA 
- Teste de aumento da SGA e PGA no banco AG/HOMOLOGAÇÃO
- Teste de aumento da SGA e PGA no banco CLINICAS/HOMOLOGAÇÃO
```

####  REUNIÃO NO IPES DIA 04/AGOSTO/2021

```ini
- Periodo de retenção em disco dos backups de AG e CLINICAS (2 dias)
- Recuperação de backup RMAN no pior das hipóteses (1h)
  Caso necessário um menor espaço de tempo - Mudar o tamanho dos Redos (200M)
- Backup RMAN para outra área (REDE) - Suporte de redes do IPES
- Datapump dos schemas com compressão - Verificar quais schemas
- Backups Mensais e Anuais usando datapump com compressão de dados
- Testes nos Bancos de Homologação - shutdown (Tarde e noite)
- Precisa de Backups dos bancos de Homologação? Não precisa
- Verificar se são servidores dedicados somente aos bancos (OK-
- Aumentar a memória (SGA/PGA) dos bancos de PRODUÇÃO/HOMOLOGAÇÃO
- Criar banco standby em Homologação, aplicando os archives de Produção
- Fazer testes de restore de Produção com RMAN em LAB

  OBS: Periodo de maior atividade em AG (7:00 as 17:00 hs)
  
- REUNIÃO DIA 10/08/2021 COM IPES E SEAD 
  Migração do Servidor Oracle 12c para Data Center da Emgetis 
  Migração do Sistema Operacional para Oracle Linux  
  
- Configuração no banco CLINICAS - Preparação para backup RMAN
- Teste de compressão com o data pump 
```



#### ATIVIDADES JUNHO/2021

```sql
# DIA 15/06/2021
DEPOIS DA ATUALIZAÇÃO DO SISTEMA DA BENNER
PROBLEMA NO BANCO AG: RESOLVIDO PELO SUPORTE DA BENNER
A BENNER DEIXOU ALGUMAS PROCEDURES/FUNCTIONS PARA ANÁLISE

# TESTE DE NUMBER(10)
drop table teste;
create table teste (num10 NUMBER(10));
select * from teste;
insert into teste num10 values (99999999999);
select user from dual;
---
select   owner,object_type,count(object_name) QTDE
from     dba_objects
where    status = 'INVALID'
and      object_type in ('PACKAGE','PACKAGE BODY','FUNCTION','PROCEDURE')
group by owner,object_type
order by owner;
---
select 'ALTER ' || OBJECT_TYPE || ' ' || OWNER || '.' || OBJECT_NAME || ' COMPILE;'
from    dba_objects
where   status = 'INVALID'
and     object_type in ('PACKAGE','FUNCTION','PROCEDURE');

---------------------------------
PROCEDURES PARA SEREM ANALISADAS
---------------------------------
BS_6DC9EE51
SCRIPTCORRECAO
BS_0B41B826
BS_322397E8
BS_E285C4EC
BS_3BB09BD1
FAPES_BSBEN_ROTINACARTAO
ATUALIZAMONITORAMENTOGUIAS
FORMATA_CPF: Compilado
---

select * from dba_source where upper(text) like upper('%PESQUISA%');
---
# AO COMPILAR OS PROCEDIMENTOS
PROCEDURE SAUDEPRO.ATUALIZAMONITORAMENTOGUIAS
PL/SQL: ORA-00904: "G"."TABENVIADOMONITORAMENTO": identificador inválido
A tabela SAUDEPRO.SAM_GUIA usada nea procedure não tem o campo TABENVIADOMONITORAMENTO
---
PROCEDURE SAUDEPRO.BS_0B41B826
PL/SQL: ORA-00904: "PERMITEPRORROGARDATASUPERIOR": identificador inválido
A tabela SAUDEPRO.SAM_PARAMETROSATENDIMENTO usada nessa procedure não tem o campo PERMITEPRORROGARDATASUPERIOR
----------------------------------------------------------------------
PROCEDURE SAUDEPRO.BS_322397E8
PLS-00306: número incorreto de tipos de argumentos na chamada para 
'BSAUT_GERARPERICIA'
'BSPRECO_CALCULAPRECO'
'BSAUT_GERARPERICIA'
'BSPROPEG_PERCENTUALPF'
'BSAUT_SUPLEMENTACAO_AUTORIZ'
'BSAUT_SUPLEMENTACAO_AUTORIZ'
'BSAUT_INCLIMITE'
---
PROCEDURE SAUDEPRO.BS_3BB09BD1
Erro(19,1): PLS-00905: o objeto SAUDEPRO.BS_E285C4EC é inválido
DEPENDE DA COMPILAÇÃO DE SAUDEPRO.BS_E285C4EC
---
PROCEDURE SAUDEPRO.BS_E285C4EC --DEPENDE DE SAUDEPRO.BS_322397E8
------------------------------------------------------------------------
FUNCTION SAUDEPRO.BS_6DC9EE51;
Erro(1,152): PLS-00201: o identificador 'BS_68B78618_TABLE' deve ser declarado
---
PROCEDURE SAUDEPRO.FAPES_BSBEN_ROTINACARTAO --IDENTIFICADOR INVALIDO K9_GERARCARTAO
---
PROCEDURE SAUDEPRO.SCRIPTCORRECAO
Erro(53,3): PLS-00306: número incorreto de tipos de argumentos na chamada para 'BSTISS_BUSCARAUTORIZPROTOCOLO'
---
FUNCTION SAUDEPRO.FORMATA_CPF --COMPILADO/RESOLVIDO
```

#### ATIVIDADES MAIO/2021

```ini
# ATIVIDADES DE MAIO 2021
- Disponibilização do acesso VPN
- Manutenção e acompanhamento diário dos scripts de Backup com o RMAN
- Testes e ajustes dos scripts do RMAN em LAB
- Verificação diária da utilização de espaço em disco para a politica de retenção dos backup de 7 dias

# PROBLEMAS OCORRIDOS DURANTE O MES
- Alguns problemas com acesso VPN - RESOLVIDO COM AJUDA DO SUPORTE DE REDE EMGETIS/IPES
- Falta de espaço em disco para o backup do dataPump - REPASSADO PARA O SUPORTE DE REDE

# TAREFAS A SEREM FEITAS NOS BANCOS DE PRODUÇÃO (AG, CLINICAS E ITABS)

# TAREFAS COMUNS A TODOS OS BANCOS (AG, CLINICAS E ITABS)
ALERT.LOG:** Rotacionar de tempos em tempos
TASK MANAGER:** Agendar tarefas no Windows
FAST RECOVERY AREA: Manutenção e Gerenciamento do crescimento
RMAN: Implementar scripts e configurações dos Backup/Restore
RMAN: Implementar Política de Retenção
RMAN: Iniciar o ciclo dos Backups FULL (Semanal)
RMAN: Iniciar o ciclo dos Backups INCREMENTAIS (Diários)
RMAN: Fazer acompanhamento dos Backups e Política de Retenção
RMAN: Corrigir possíveis erros
ARCHIVELOGS: Apagar lixos, configurar path
ARCHIVELOGS: Habilitar somente depois da observação dos REDO LOGS
CONTROFILES: Se possivel coloca-los em discos diferentes

# AG
DATAFILES-AUTOEXTEND ON: SAUDE_DAT08.DBF, SAUDE_IDX09.DBF e TEMP02.DBF
REDOLOGS: Multiplexar e aumentar o tamanho
CONTROFILES: Se possivel mover o CONTROL03.CTL para outro disco

# CLINICAS
DATAFILES: AUTOEXTEND ON em IPESAUDEHOM_DAT01.DBF
DATAFILES: AUTOEXTEND ON em PORTAL_DAT01.DBF
DATAFILES: AUTOEXTEND ON em PORTAL_IDX01.DBF
TEMPFILES: AUTOEXTENDED ON em TEMP01.DBF
REDOLOGS: Aumentar o tamanho e coloca-los em grupos distintos

# ITABS
DATAFILES: AUTOEXTEND ON em ITABS_01.DBF
REDOLOGS: Somente Multiplexar (Não geram muitos ARCHIVESLOGS) 
```



### PROCEDURES & FUNCTIONS COM ERROS DE COMPILAÇÃO

SCHEMA: SAUDEPRO

BANCO: AG PRODUÇÂO - 172.24.8.9

| NOME DO OBJETO             | T    | MENSAGEM DE  ERRO                                            | MOTIVO DO ERRO                                               | SOLUÇAO                    |
| :------------------------- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------- |
| BS_3BB09BD1                | P    | Objeto é inválido                                            | Depende da compilação de BS_E285C4EC (1)                     | Depende de (2)             |
| BS_E285C4EC (1)            | P    | Objeto é inválido                                            | Depende da compilação de BS_322397E8 (2)                     | Depende de (2)             |
| BS_322397E8 (2)            | P    | PLS-00306: número incorreto de tipos de argumentos na chamada |                                                              |                            |
| ATUALIZAMONITORAMENTOGUIAS | P    | PL/SQL: ORA-00904: "G"."TABENVIADOMONITORAMENTO": identificador inválido | A tabela SAM_GUIA usada nessa procedure não tem o campo TABENVIADOMONITORAMENTO | Criação do campo na tabela |
| BS_0B41B826                | P    | PL/SQL: ORA-00904: "PERMITEPRORROGARDATASUPERIOR": identificador inválido | A tabela SAM_PARAMETROSATENDIMENTO usada nessa procedure não tem o campo PERMITEPRORROGARDATASUPERIOR | Criação do campo na tabela |
| FAPES_BSBEN_ROTINACARTAO   | P    | Identificador inválido K9_GERARCARTAO                        |                                                              |                            |
| SCRIPTCORRECAO             | P    | PLS-00306: número incorreto de tipos de argumentos na chamada para 'BSTISS_BUSCARAUTORIZPROTOCOLO' |                                                              |                            |
| BS_6DC9EE51                | F    | PLS-00201: o identificador 'BS_68B78618_TABLE' deve ser declarado |                                                              |                            |
| FORMATA_CPF                | F    | Objeto é inválido                                            | A função FORMATA_CNPJ estava declarada  dentro da função FORMATA_CPF | Resolvido                  |

OBS: T=Tipo, P=Procedure e F=Function

Muito provavelmente esses Objetos (Procedures e Funções) não estavam sendo usados porque todos estão com erros de compilação. A possível solução passa por algumas intervenções no âmbito da programação e criação de alguns campos em determinadas tabelas, conforme mostra o quadro acima.

Acho que é muito provável que esses objetos estejam OBSOLETOS. 

Com exceção de FORMATA_CPF que foi solucionado facilmente, devemos observar se o Sistema realmente vai fazer alguma chamada a algum desses objetos, para que, caso verdadeiro possamos juntamente com o pessoal de desenvolvimento da BENNER solucionar o problema.



Boa tarde Bruno

Desculpe a demora para resolver o problema das Procedures e Funções, mas estava providenciando a cirurgia de meu filho, ia ser amanhã dia 30 de junho, mais foi adiada porque nesses ultimos dois dias ele teve febre, talvez alguma infecção ou Covid, estamos investigando. 

Assim que ele melhore, a cirurgia será feita.





SOLUÇÃO ADOTADA PARA O PROBLEMA DE COMPILAÇÃO

Apesar das Procedures e Funções com erro de compilação não estarem sendo utilizadas, adotei a estratégia de renomea-las ao invés de apaga-las, para que caso necessário no futuro possamos reaproveita-las.

Serão renomeadas da seguinte forma:  

Vou colocar um prefixo "OBSOLETO" na frente do nome original para que possa ser identificada.

1.OBSOLETO_BS_6DC9EE51
2.OBSOLETO_SCRIPTCORRECAO
3.OBSOLETO_BS_0B41B826
4.OBSOLETO_BS_322397E8
5.OBSOLETO_BS_E285C4EC
6.OBSOLETO_BS_3BB09BD1
7.OBSOLETO_FAPES_BSBEN_ROTINACARTAO
8.OBSOLETO_ATUALIZAMONITORAMENTOGUIAS   

---

FUNCTION OBSOLETO_BS_6DC9EE51
PROCEDURE OBSOLETO_SCRIPTCORRECAO
PROCEDURE OBSOLETO_BS_0B41B826
PROCEDURE OBSOLETO_BS_322397E8
PROCEDURE OBSOLETO_BS_E285C4EC
PROCEDURE OBSOLETO_BS_3BB09BD1
PROCEDURE OBSOLETO_FAPES_BSBEN_ROTINACARTAO
PROCEDURE OBSOLETO_ATUALIZAMONITORAMENTOGUIAS   



### MINHAS FÉRIAS - 2021


```
Bom dia Elvis,
Tenho férias *vencidas*.
É possível programar para *JUNHO* ou *JULHO*?
Falar com o pessoal da Emgetis

*ÚLTIMAS FÉRIAS:*
PER.AQUISITIVO: *26/12/18 à 25/12/19*
PER.GOZO: *02/01/20 à 31/01/20*

NUM_CPF: 155.982.095-00
NOME: KLEBER BARBOSA DO NASCIMENTO
COD_VINCULO: 133555	  
```

#### SETUP SSD

```ini
KINGDION: SATA 3 120GB SSD
KINGSPEC: P4-120
WD-GREEN: WDC-WDS120G2G0A
```

#### ACESSO VIA RDP - REMMINA

```ini
# Abrir o prompt de comandos
# cd PSTools
HOMO-AG:	psexec \\172.24.8.17 -s cmd		explorer \\172.24.8.17\c$
HOMO-CLI:	psexec \\172.24.8.39 -s cmd		explorer \\172.24.8.39\c$
HOMO-LAB:	psexec \\172.24.8.171 -s cmd	explorer \\172.24.8.171\c$
PROD-AG:	psexec \\172.24.8.9 -s cmd		explorer \\172.24.8.9\c$
PROD-CLI:	psexec \\172.24.8.19 -s cmd		explorer \\172.24.8.19\c$
PROD-ITABS:	psexec \\172.24.8.41 -s cmd		explorer \\172.24.8.41\c$
---
# Copiar e colar
cd \app\Administrator\virtual\fast_recovery_area\producao\PRODUCAO
cd \app\Administrator\virtual
tree /f /a fast_recovery_area > FRA_LAB_AG.txt
rman target / msglog=\app\Administrator\virtual\scripts\log_backup\saida_rman_teste.log

set ORACLE_SID=PRODUCAO
set ORACLE_SID=CLINICAS
echo %ORACLE_SID%
---
find "ERROR" /n *.log
findstr /i "ERROR" *.log
---
tree /a /f
start app.exe
```

#### CRIAR A ESTRUTURA DE BACKUP-RMAN

```sql
# c:\app\adminstrator\virtual\scripts
backup_full_homo_ag.bat
backup_full_homo_ag.sql
backup_diario_homo_ag.bat
backup_diario_homo_ag.sql
# c:\app\adminstrator\virtual\scripts\log_backup
logs do rman
```

#### PROBLEMAS DO DIA A DIA

```sql
sqlpplus / as sysdba
shutdown immediate;
shutdown abort; --Depende do caso
startup;
---
# p1 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_IDX09.DBF'
# p2 'E:\ORADATA\PRODUCAO\DADOS\PRODUCAO\DATAFILE\SAUDEPRO_DAT08.DBF'
alter database datafile p1 autoextend on;
alter database datafile p2 autoextend on;
```

#### TABLESPACE IPESAUDE_DAT 100%

```sql
# p1 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._01.DBF'
# p2 'C:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\CLINICAS\DATAFILE\IPESAUDE_DAT._02.DBF'
# p3 'C:\APP\ADMINISTRADOR\VIRTUAL\ORADATA\CLINICAS\PORTAL_DAT01.DBF'
alter database datafile p1 autoextend on;
alter database datafile p2 autoextend on;
alter database datafile p3 autoextend on;
```

#### SENHA EXPIRADA

```sql
# Desabilita expiração de senhas
ALTER PROFILE DEFAULT LIMIT COMPOSITE_LIMIT UNLIMITED
  PASSWORD_LIFE_TIME UNLIMITED
  PASSWORD_REUSE_TIME UNLIMITED
  PASSWORD_REUSE_MAX UNLIMITED
  PASSWORD_VERIFY_FUNCTION NULL
  PASSWORD_LOCK_TIME UNLIMITED
  PASSWORD_GRACE_TIME UNLIMITED
  FAILED_LOGIN_ATTEMPTS UNLIMITED;

ALTER PROFILE DEFAULT LIMIT 
  PASSWORD_GRACE_TIME UNLIMITED
  PASSWORD_LIFE_TIME UNLIMITED;
  
select resource_name,limit from dba_profiles where profile='<profile_name>';

select username,expiry_date,account_status from dba_users; 
alter user teste identified by "teste123" account unlock;

---

select * from dba_users;
USER SSERVER DE HOMO-AG
ALTER USER SSERVER IDENTIFIED BY "****";
ALTER USER SSERVER ACCOUNT UNLOCK;
ALTER USER SSERVER PASSWORD EXPIRE;
---
ALTER USER CAMPANHAMENSAGENS IDENTIFIED BY "*****";
```

### EMGETIS - ORACLE

```bash
$ ls -lt
$ ls -lr 
$ ls -lSrh (l=saida detalhada, S=ordem de tamanho, r=ordem reversa, h=melhor para humanos)
#$ du -h --max-depth=1 | sort 
#$ ls | sort 
$ ll 2>/dev/null
$ du -hsc * 2>/dev/null
$ find . -name '*'.trc -mtime +360 -daystart -exec ls {} \;
$ find . -name '*'.log -mtime +6 -daystart -exec rm {} \;
```

