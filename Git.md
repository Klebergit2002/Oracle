## GIT & GITHUB

### CONFIGURAÇÕES INICIAIS

```bash
# NOVA CONTA NOV/2021
email : kleber2002@hotmail.com
passwd: xadrez@2002

username: klebergit2002 (k minusculo)
token : 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

echo "# Oracle" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Klebergit2002/Oracle.git
git push -u origin main

# VERIFICANDO A VERSAO
git --version

# DEFININDO O USUÁRIO
git config --global user.name "KleberBarbosa"
git config --global user.email kleber2002@hotmail.com

# Não solicita senha da segunda vez 
git config credential.helper store  

# DEFININDO O EDITOR PADRÃO
git config --global core.editor nano

# CONFERINDO AS CONFIGURAÇÕES
git config --list
git config user.name

# PEDINDO AJUDA
git help <verb>
git <verb> --help
man git-<verb>
git help config

# APAGANDO O REPOSITORIO GIT
$ REM cd pasta do git
$ rm -rf .git
```

### OBTENDO UM REPOSITÓRIO GIT

```bash
# EXISTEM DUAS FORMAS DE SE OBTER UM REPOSITÓRIO GIT
# 1. Fazendo uso de um projeto ou diretório existente e o importa para o Git
# 2. Fazendo um clone de um repositório Git existente a partir de outro servidor

# 1.INICIANDO UM REPOSITÓRIO EM UM DIRETÓRIO EXISTENTE
$ cd diretorio
$ git init			# Inicia a linha do tempo
$ git add *.c		# Adiciona ou atualiza mudanças
--$ git add LICENSE
$ git commit -m		# Adiciona um ponto na linha do tempo 

# 2.CLONANDO UM REPOSITORIO EXISTENTE
$ git clone https://github.com/libgit2/libgit2 -- Clona e cria a pasta 'libgit2'
$ git clone https://github.com/libgit2/libgit2 mylibgit -- Clona na pasta 'mylibgit'

# 2.1 ATUALIZANDO REPOSITORIO LOCAL COM O REMOTO
$ git pull origin master

# ESTADOS DOS ARQUIVOS NO SEU DIRETÓRIO DE TRABALHO
1. Rastreado     'Arquivos que o Git conhece'
2. Não Rastreado 'São todos os outros'

# ESTÁGIO DOS ARQUIVOS
1. Modified # Modificado quando alterado
2. Staged   # Selecionado para o proximo commit
3. Commited # Gravado com segurança
```

### TRABALHANDO DE FORMA LOCAL

```bash
# VERIFICANDO OS STATUS DE SEUS ARQUIVOS
$ git status

# RASTREANDO ARQUIVOS NOVOS
$ git add arq1.txt	# Coloca arq1.txt no STAGE'
$ git status		# Informa o estado das alterações
$ git status -s 	# Status curto --short'
$ git show			# Mostra determinado ponto da historia

# IGNORANDO ARQUIVOS (.gitignore)
$ cat .gitignore
*.[oa] -- Ignora arquivos terminados com '.o' ou '.a' 
*~     -- Ignora arquivso terminados com '~' 

# DIFERENÇAS ENTRE ARQUIVOS DENTRO E FORA DO STAGE
$ git diff			#Ver o que alterou e ainda não mandou para o STAGE
$ git diff --staged	# Ver o que alterou no STAGE
$ git diff --cached	# Ver o que já foi para o STAGE até agora

# FAZENDO COMMITS DAS SUAS ALTERAÇÕES
$ git commit
$ git commit -m "Mensagem"
$ git commit -a -m "Pula o STAGE"

# REMOVENDO ARQUIVOS
$ rm arq1.txt		# Não fazer dessa forma'
$ git rm arq1.txt	# Talvez precise da opção (-f)

# MOVENDO/RENOMEANDO ARQUIVOS
$ git mv origem.txt destion.txt

# VENDO HISTÓRICOS DE COMMITS
$ git log	 	# Lista em ordem cronológica inversa'
$ git log -p 	# Lista diferenças introduzidas em cada commit'
$ git log -2 	# Lista apenas os dois últimos itens'
$ git log -p -2
$ git log --stat
$ git log --pretty=online
$ git log --pretty=format:"%h - %an, %ar : %s"
$ git log --pretty=format:"%h %s" --graph
$ git log --since=2.weeks
$ git log -Sfunction_name

# DESFAZENDO AS COISAS
$ git commit -m "initial commit" # Não era pra fazer, esqueceu do git add
$ git add forgotten_file
$ git commit --amend

# RETIRANDO ARQUIVOS DO STAGE
$ git add *
$ git status
$ git reset HEAD CONTRIBUTING.md

# DESFAZENDO AS MODIFICAÇÕES DE UM ARQUIVO
$ git checkout -- CONTRIBUTING.md
$ git status

# DESFAZENDO COMMIT
git checkout -- arquivo.txt # Volta para o estado anterior ao ultimo commit
git reset --soft
git reset --hard
git revert 11a5....	# commit especifico
git revert HEAD		# commit mais recente

# MOVIMENTANDO-SE ENTRE COMMITS
$ git log
$ git checkout <rash do commit>
$ git checkout master # Volta para a situação atual 

# INICIAR UMA NOVA RAMIFICAÇÃO DO PROJETO
$ git branch "<nome>"		# Ramificação
$ git checkout "<branch>"	# Muda de branch 
$ git status
$ git merge master 			# Junta com o master
$ git branch -D "<branch>"  # Deleta a branch
$ git branch 				# Lista as branchs
```

### TRABALHANDO DE FORMA REMOTA

```bash
# EXIBINDO SEUS REPOSITÓRIOS REMOTOS
$ git remote
$ git remote -v

# REGISTRANDO REPOSITÓRIOS REMOTOS NA SUA MAQUINA LOCAL
$ cd projeto
$ git clone <url> 

# REGISTRANDO REPOSITÓRIOS LOCAIS NO GITHUB
# Criar um novo repositório no GitHub
# Oracle12c <Privado>

# Token - nova forma de acesso ao github - usar no lugar da senha
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
git config credential.helper store # Não solicita senha

$ git remote set-url origin https://KleberBarbosa@github.com/KleberBarbosa/Oracle12c.git

---

$ git remote add origin https://bitbucket.org/kleber2002/oracle12c.git
$ git branch -M main
$ git push -u origin main
---
$ git remote add origin https://github.com/KleberBarbosa/Oracle12c-IPES.git
$ git branch -M main
$ git push -u origin main
---
# Oracle12c-IPES <Publico>
$ echo "# Oracle12c-IPES" >> README.md
$ git init
$ git add README.md
$ git commit -m "Primeiro' commit"
$ git branch -M main
$ git remote add origin https://github.com/KleberBarbosa/Oracle12c-IPES.git
$ git push -u origin main

# CONFERINDO LOCAL-REMOTO
$ git show-ref
$ git ls-remote
```