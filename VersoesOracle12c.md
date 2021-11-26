#### VERIFICAÇÃO DAS VERSÕES DO ORACLE 

```sql
# Entrar como DBA
select name from v$database;
select * from v$version;
```

###### OBS: Todos os servidores de banco com exceção de PRODUÇÃO-AG estão com as versões Enterprise como mostra os resultados obtidos dos SELECTS acima.

###### Não pude verificar banco PROD-BI (172.24.8.3) porque ainda não tenho acesso.



#### PRODUÇÃO AG - 172.24.8.9

```sql
Oracle Database 12c Standard Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production
```

#### PRODUÇÃO CLINICAS - 172.24.8.19

```sql
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production
```

#### PRODUÇÃO ITABS - 172.24.8.41

```sql
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production
```



#### HOMOLOGAÇÃO AG - 172.24.8.17

```sql
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production
```

#### HOMOLOGAÇÃO CLINICAS - 172.24.8.39

```sql
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
PL/SQL Release 12.2.0.1.0 - Production
"CORE	12.2.0.1.0	Production"
TNS for 64-bit Windows: Version 12.2.0.1.0 - Production
NLSRTL Version 12.2.0.1.0 - Production
```

