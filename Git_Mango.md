## GIT - MANGO

Sistema de Controle de Versão Distribuído, criado por Linus Torvalds em 2005.

### **#1 Git - Config, Init e Clone**

- Workspace
- Local Repository
- Remote Repository

![Git_workspace](/home/kleber/Dropbox/Apple_IT_IPES_2021/Oracle-Windows/Git_workspace.png)

### Config

```bash
# Existem três níveis de configurações - arquivos
-System	 # Vale para qualquer projeto e qualquer usuário (global) 
-User	 # Seu usuário apenas
-Project # Seu projeto apenas

$ git config --list
$ git config --system --edit # Acessa arquivo do system
$ git config --global --edit # Acessa arquivo do user
$ git config --local --edit	 # Arquivo local do projeto

$ git config --global core.editor nano	# Alerando o editor

# Prioridades das configurações
"local" tem prioridade sobre o "global" 
"global" tem prioridade sobre o "system"
```

### Init - Formas de iniciar um projeto com o Git

```bash
# Primeira forma
$ mkdir projeto
$ cd projeto
$ git init
$ git status

# Segunda forma, quando ja existe um repositório remoto
$ mkdir projeto
$ cd projeto
$ git clone https://github.com/KleberBarbosa/Oracle12c.git
$ git status

# Lista repositórios remotos
$ git remote -v

# Registrando um repositório remoto no Github
$ git remote add origin https://github.com/KleberBarbosa/Oracle12c.git
$ git remote -v
```

### #2 Git - Alias, Status, Add, Commit, Amend e Stash

#### Alias

```bash
$ git config --global --edit # Configurações somente para meu user
# Inserir no arquivo
# [alias]
# 	s = !git status -s
# 	c = !git add --all && git commit -m
# 	p = !git push
```

#### Status, Add

```bash
# Três niveis de status
"Untracked	:" Arquivos novos (Não reconhecido pelo git)
"not staged	:" Arquivos modificados (Reconhecido pelo git)
"staged		:" Arquivos prontos para serem comitados

$ git status -s # Arquivos que o git ainda não conhece
$ git add .		# Aquivos vão para a Staged Area
$ git add --all
$ git status -s 
```

#### Commit, Amend

```bash
# Juntando commits (Junta com o último commit)
$ git commit --amend --noedit
```

#### Stash

```bash
# Esconder e guardar o codigo 
$ git add .
$ git stash
$ git stash list
$ git stash apply # Voltar o codigo sem limpar o stash 
$ git stash pop   # Volta limpando o stash  
```

### #3 Git - Conventional Commits, Logs e Tags

#### Conventional Commits

```bash
<type>[optional scope]: <description>
[optional body]
[optional footer(s)]

# Exemplos de tipos:
fix:, feat:
build:, chore:, ci:, docs:, style:, refactor:, perf:, test
```

#### Logs

```bash
$ git log --oneline
# Personalizando o git log
# %H  = hash completo
# %h  = hash reduzido
# %cn = quem fez o commit
# %cr = data
# %d  = branch
# %C  = cor
$ git log --pretty=format:'%C(cyan)%h %C(red)%d %C(white)%s - %C(cyan)%cn, %C(green)%cr'

# Adicionando um alias
$ git config --global --edit 
# [alias]
# 	l = !git log --pretty=format:'%C(cyan)%h %C(red)%d %C(white)%s - %C(cyan)%cn, %C(green)%cr'
```

#### Tags

```bash
# Marcar uma versão (tipos: Light e Anotada)
# Tags Light - Repositório Local
# Tags Anotadas - Repositorio Remoto
$ git tag 1.0
$ git tag
$ git show
# Colocando tag em commits anteriores
$ git tag -a "0.1.beta" -m "release 0.1.beta" <hash do commit>

# Apagando tag
$ git tag -d <nome da tag>

$ git tag 1.0 -m "realease 1.0"

# Enviando tags para o Servidor Remoto
$ git push origin master --tags 	   # Envia todas as tags: Não recomendado
$ git push origin master --follow-tags # Envia so as tags anotadas: Recomendado

# Colocando o follow-tags como padrão
$ git config --global --edit 
# [push]
#	followtags = true

# Removendo tags do Servidor Remoto
# Remove primeiro do Servidor Local
$ git tag -d <nome da tag> 				 # Remove do Servidor Local
$ git push --delete origin <nome da tag> # Remove do Servidor Remoto
```

### #4 Git - Reset e Revert

#### Reset

```bash
# Reverter o git add
$ git reset

# Reverter um commit
$ git reset <id_commit anterior>

```

#### Revert

```bash

```

### #5 Git - Rm, Clean e Checkout
