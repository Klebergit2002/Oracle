## GIT - BOSON TREINAMENTOS

Sistema de Controle de Versão Distribuído, criado por Linus Torvalds em 2005.

#### **Vantagens do Git**

- Permite rastrear alterações com muita facilidade
- Excelente para trabalho em equipe
- Permite fazer revisão em alterações feitas por outros membros
- Sistema de branching aprimorado
- Merging extremamente simples
- Stashing para testes de features em interferir em modificações atuais
- Trabalha com snapshots
- Histórido com autenticação criptográfica

#### **Exemplos de Projetos que usam Git**

- Android
- Debian
- Eclipse
- Gimp
- jQuery
- Kernel do Linux
- PHP
- Reddit
- Ruby Rails
- VLC

#### Sistema de três Estados

- **Estado Modificado:** Snapshot atual, arquivos do diretório de trabalho

- **Estado Preparado:** Arquivos modificados são marcados para irem ao Staging Area

- **Estado Consolidado:** Os dados são salvos no Banco de Dados local do Git

#### Fluxo de Operações do Git

- Os arquivos são criados, modificados ou excluídos
- Arquivos que serão incluídos no snapshot são adicionados na área de staging
- Snapshot é criado
- Adicionar modificações ao banco de dados
- Assim, um arquivo vai do estado modificado para staged e então para commited

#### Github

- É possível tem um local central para armazenar todo o trabalho realizado
- Github é um exemplo desse tipo de repositório centralizado
- Não é obrigatório, porém permite a colaboração de outros desenvolvedores
- Bitbucket é outro exemplo de serviço de hospedagem de projetos

#### Instalação do Git

Site oficial para download: https://git-scm.com/

```bash
# Debian e derivados
$ sudo apt install git
$ git --version

# Windows
# Baixar o executavel e instalar
# Usar o GitBash (prompt de comando)
```

#### Comandos iniciais do Git

```bash
# Configurando o usuário
$ git config --global user.name = "KleberBarbosa"
$ git config --global user.email = "kleber2002@gmail.com"

# Configurando a pasta do projeto
$ mkdir /pasta-git
$ cd /pasta-git
$ git init .	# Cria a pasta .git
$ touch teste.txt
$ git status
$ git add teste.txt
$ git commit -m "Primeiro commit!"
$ git log

# Git Help
$ git help
$ git help -a 
$ git help -g
$ git help git-config
```

#### Repositórios

Para cada projeto novo, seguimos os seguintes passos:

1. Criar um diretório para armazenar o projeto
2. Entrar no diretório (opcional)
3. Inicializar o repositório Git

#### Diretórios

É criado um subdiretório oculto **.git**, que guardará os dados de alteraçõee e snapshots.

A área for do diretório **.git** é o **Diretório de Trabalho**, onde ficam armazenados os arquivos que você ira manipular, ou seja, é a área onde interagimos diretamento com os arquivos do projeto.

#### Staging Area

Local para onde os arquivos vão antes de um snapshot ser criado.

Somente os arquivos da **staging area** vão para um snapshot.

Selecionamos os arquivos desejados no diretório de trabalho.

#### Commits

- Snapshot do projeto em um determinado momento, com informações sobre o autor do conteúdo e quem realizou.
- O estado anterior do projeto é denominado "pai". Commits são ligados entre si por conexões pai-filho.
- O conjunto de commits relacionados entre si por paternidade é chamado de branch (ramo).
- Um commit pode ter dois pais, se for criado pe.... (mescla) de dois branches.

#### Identificação do Commit

Identificamos um commit pelo seu nome, que é a string de 40 caracteres obtida pelo hash (SHA1) do commit.

#### Exercício Passo-a-passo

Criar um novo repositório, de nome exercicio01. (Iniciar o repositorio!).

```bash
$ mkdir /home/kleber/Extras/exercicio01
$ cd /home/kleber/Extras/exercicio01
$ git init .
```

Criar um arquivo de nome projeto.txt no diretório, e digitar algum texto nele.

```bash
$ cd /home/kleber/Extras/exercicio01
$ touch projeto.txt
$ nano projeto.txt << "Testando repositorios" 
```

Colocar o arquivo na staging area.

```bash
$ git add projeto.txt
$ git status
```

Efetuar commit no projeto, acrescentando uma mensagem curta.

```bash
$ git commit -m "Projeto iniciado"
$ git status
```

Criar dois novos arquivos, de nomes tarefa.txt e pendencias.txt.

```bash
$ touch tarefa.txt pendencias.txt
$ ls -l
```

Colocá-los na staging area e comitar o projeto.

```bash
$ git add *
$ git status
$ git commit -m "Arquivos novos adicionados"
```

Renomear o arquivo pendencias.txt para restante.txt.

```bash
$ mv pendencias.txt restante.txt
$ ls -l
```

Adicionar texto ao arquivo tarefa.txt e verificar o status do diretório

```bash
$ nano tarefa.txt << "Testando mais um commit"
$ git status
```

Colocar na área de stage os arquivos restante.txt e tarefa.txt.

```bash
$ git add restante.txt tarefa.txt
$ git status
```

Tirar o arquivo tarefa.txt da staging area, efetuar commit e verificar status do pojeto

```bash
$ git rm --cached tarefa.txt
$ git status
$ git commit -m "Mais um commit" 
$ git status
```

Finalizando o exercicio01

```bash
$ cd ..
$ rm -rf /home/kleber/Extras/exercicio01
```

#### Clonagem

É possível copiar um diretório inteiro, com todo o seu histórico e snapshots, em um processo chamado de "clonagem".

#### Log do Commit

O log do commit lista as seguintes informações:

- Nome do commit (via hash)
- Autor
- Email
- Data
- Descrição

```bash
# Mostra o log dos commits
$ git log
$ git log teste.txt # So os logs do arquivo teste.txt

# Mostra o que foi alterado em cada commit
$ git show # Ultimo commit
$ git show <Hash>
```

#### Revisar alterações atuais

Podemos verificar as alterações realizadas nos arquivos no diretório de trabalho (**cópia local**),  com o comando  **git diff**

- **git diff** é o mesmo que **git diff HEAD**

```bash
$ git diff
$ git diff arq1
$ git diff --staged # Mostra a diferença dos arquivos não commitados
```

#### Remoção de arquivos

```bash
# Deletando o arq3
$ git add .
$ git commit -m "Commit do arquivo 3"
$ git rm arq3
$ git status
$ git commit -m "Arq3 excluido"
$ ls -l

# Recuperando o arq3
$ git log --diff-filter=D --summary
$ git checkout 4ec6~1 arq3
$ git add .
$ git commit -m "Arq3 recuperado"
$ git log
```



#### Reverter Commits

Eventualmente você terá a necessidade de reverter um commit, por exemplo por conta de erros cometidos nos arquivos.

Desfazer um commit é basicamente comitar exatamente o oposto do commit em si. O git, na prática, cria um novo commit com as mudanças opostas.

Para tal usamos comando **git revert**.

```bash
$ git revert <hash do commit>
$ git revert HEAD # Ultimo commit

# Exemplo
$ git log --oneline # Pega o hash do commit
$ git revert <hash>
$ git revert --abort
```

##### Regras importantes

1. Deve-se trabalhar em um diretório de trabalho limpo. 

   Executar sempre **git add** e **git commit** antes de tentar reverter um commit.

2. Novamente: não mude o passado - ao menos não por enquanto!



#### Desfazer mudanças

Às vezes precisamos desfazer alterações realizadas em nossos arquivos locais, após os commits terem sido executados.

Existe mais de uma maneira de fazer isso. Uma delas é o emprego do comando **git reset.**

Sintaxe: **git reset [modo] [arquivo | commit]**

#### Comando git reset

O comando git reset permite desfazer alterações.

O git reset move os ponteiros HEAD e do branch atual para um estado específicado.

Além de atualizar os ponteiros de referência do commit, também pode modificar o estado das três árvores.

#### Principais aplicações

As principais aplicações do git reset são:

- Descartar commits em um branch privado
- Desfazer alterações não comitadas, também em uma branch privado
- Tirar arquivos da área stage

Para descartar commits em um branch público deve-se usar o comando git revert. Já para descartar alterações no diretório de trabalho, o indicado é o git checkout.

#### Modos do git reset

O git reset possui três modos principais de invocação, correspondendo aos três estados internos do git (Ávores):

- --soft
- --mixed (padrão)
- --hard

O git ainda possui outros adicionais:

- --merge
- --keep

#### Modos do git reset

- **--soft:** Não toca no arquivo de índice ou no diretório de trabalho, mas reseta a HEAD para o commit especificado. Muda todos os arquivos para o estado "*Changes to be commited*" (é modo seguro).
- **--mixed:** Reseta o índice mas não a árvore de trabalho e relata o que não foi atualizado. É a ação padrão. Tira o que está no stage, não toca no diretório de trabalho (é modo seguro).
- **--hard:**  Reseta o índice e a árvore de trabalho. Quaisquer mudanças em arquivos rastreados na árvore de trabalho desde o commit são descartadadas. Descarta o que está na stage area, o que não está e atualiza o diretório de trabalho (não é modo seguro).

