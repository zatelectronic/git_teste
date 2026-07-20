# Manual de Git & GitHub — Do Básico ao Avançado

> Guia de referência prático, com foco em fluxo de trabalho real (firmware, PCB, projetos pessoais no GitHub).

---

## Índice.

1. [Conceitos fundamentais](#1-conceitos-fundamentais)
2. [Instalação e configuração inicial](#2-instalação-e-configuração-inicial)
3. [Comandos básicos do dia a dia](#3-comandos-básicos-do-dia-a-dia)
4. [Branches (ramificações)](#4-branches-ramificações)
5. [Trabalhando com remotos e GitHub](#5-trabalhando-com-remotos-e-github)
6. [Resolvendo conflitos](#6-resolvendo-conflitos)
7. [.gitignore e arquivos ignorados](#7-gitignore-e-arquivos-ignorados)
8. [Desfazendo coisas (reset, revert, checkout, restore)](#8-desfazendo-coisas)
9. [Stash, tags e cherry-pick](#9-stash-tags-e-cherry-pick)
10. [Rebase interativo (avançado)](#10-rebase-interativo-avançado)
11. [Reflog — o &#34;desfazer tudo&#34; de emergência](#11-reflog)
12. [Submódulos e Git LFS](#12-submódulos-e-git-lfs)
13. [Hooks do Git](#13-hooks-do-git)
14. [GitHub: SSH, PAT, Actions, Pages, Issues, PR](#14-github-recursos-avançados)
15. [Fluxos de trabalho (Git Flow x Trunk-Based)](#15-fluxos-de-trabalho)
16. [Boas práticas de commits](#16-boas-práticas-de-commits)
17. [Comandos de diagnóstico e produtividade](#17-diagnóstico-e-produtividade)

---

## 1. Conceitos fundamentais

- **Repositório (repo):** pasta com histórico versionado (`.git/` escondido dentro dela).
- **Commit:** uma "foto" do estado dos arquivos em um momento, com uma mensagem.
- **Working directory:** seus arquivos como estão agora, no disco.
- **Staging area (index):** área intermediária onde você escolhe o que vai entrar no próximo commit.
- **HEAD:** ponteiro para o commit/branch atual em que você está.
- **Branch:** linha de desenvolvimento independente (um ponteiro móvel para um commit).
- **Remote:** cópia do repositório hospedada em outro lugar (ex: GitHub), chamada normalmente de `origin`.

Fluxo típico:

```
Working Directory → (git add) → Staging Area → (git commit) → Repositório local → (git push) → Remoto (GitHub)
```

---

## 2. Instalação e configuração inicial

### Instalação

- **Windows:** [git-scm.com](https://git-scm.com/download/win) ou `winget install --id Git.Git -e`
- **Linux (Debian/Ubuntu):** `sudo apt install git`
- **Mac:** `brew install git`

### Configuração obrigatória (uma vez por máquina)

```bash
git config --global user.name "Kenneth Zat"
git config --global user.email "seu-email@exemplo.com"
```

### Configurações úteis

```bash
git config --global init.defaultBranch main       # branch padrão "main" em vez de "master"
git config --global core.editor "code --wait"     # usar VS Code como editor do Git
git config --global color.ui auto                 # saída colorida
git config --global pull.rebase false              # estratégia padrão do pull (merge)
```

Ver tudo que está configurado:

```bash
git config --list
```

---

## 3. Comandos básicos do dia a dia

### Criar ou clonar um repositório

```bash
git init                          # cria um repo novo na pasta atual
git clone https://github.com/zatelectronic/repo.git   # clona um repo existente
```

### Ver o estado atual

```bash
git status        # o que mudou, o que está staged, branch atual
git log           # histórico de commits
git log --oneline --graph --all   # histórico compacto e visual (ótimo pra entender branches)
```

### Adicionar e commitar

```bash
git add arquivo.cpp        # adiciona um arquivo específico ao staging
git add .                  # adiciona tudo que mudou na pasta atual
git commit -m "Mensagem clara do que foi feito"
git commit -am "Add + commit em um passo (só para arquivos já rastreados)"
```

### Ver diferenças

```bash
git diff                # mudanças não staged
git diff --staged       # mudanças já no staging, ainda não commitadas
git diff HEAD~1 HEAD    # diferença entre o penúltimo e o último commit
```

---

## 4. Branches (ramificações)

```bash
git branch                     # lista branches locais
git branch -a                  # lista branches locais + remotas
git branch nome-da-branch      # cria uma branch nova (sem mudar pra ela)
git switch nome-da-branch      # muda para a branch (forma moderna)
git checkout nome-da-branch    # muda para a branch (forma clássica, ainda muito usada)
git switch -c nova-feature     # cria E já muda para a branch
```

### Mesclando (merge)

```bash
git switch main
git merge nova-feature      # traz as mudanças de "nova-feature" para "main"
```

- **Fast-forward:** quando não houve divergência, o Git só "empurra" o ponteiro. Simples e limpo.
- **Merge commit:** quando houve divergência, o Git cria um commit novo que une as duas histórias.

### Apagar branch

```bash
git branch -d nome-da-branch    # apaga só se já foi mesclada (seguro)
git branch -D nome-da-branch    # força apagar mesmo sem merge
```

---

## 5. Trabalhando com remotos e GitHub

```bash
git remote -v                                   # lista remotos configurados
git remote add origin https://github.com/zatelectronic/repo.git
git push -u origin main                         # primeiro push, já linkando a branch
git push                                        # pushes seguintes
git pull                                        # baixa e mescla mudanças do remoto
git fetch                                       # só baixa, sem mesclar (mais seguro pra inspecionar antes)
```

### Diferença fetch x pull

- `git fetch` → atualiza as referências remotas localmente (`origin/main`), mas **não mexe** no seu working directory.
- `git pull` → é basicamente `git fetch` + `git merge` (ou rebase, dependendo da config) automaticamente.

### Criando repositório no GitHub e conectando

```bash
gh repo create meu-projeto --public --source=. --remote=origin --push
```

*(requer o [GitHub CLI](https://cli.github.com/) instalado e autenticado com `gh auth login`)*

Ou manualmente: cria o repo vazio no site do GitHub, depois:

```bash
git remote add origin https://github.com/zatelectronic/meu-projeto.git
git branch -M main
git push -u origin main
```

---

## 6. Resolvendo conflitos

Conflito acontece quando o Git não consegue decidir sozinho como unir duas mudanças na mesma linha/arquivo.

```bash
git merge feature-x
# CONFLICT (content): Merge conflict in main.cpp
```

O arquivo vai ficar assim:

```cpp
<<<<<<< HEAD
codigo_da_branch_atual();
=======
codigo_da_outra_branch();
>>>>>>> feature-x
```

**Passos para resolver:**

1. Abra o arquivo, decida o que fica (pode ser um dos dois, os dois, ou algo novo).
2. Apague as marcações `<<<<<<<`, `=======`, `>>>>>>>`.
3. `git add arquivo-resolvido.cpp`
4. `git commit` (sem `-m`, ele já sugere a mensagem de merge) ou `git merge --continue`

Para abortar o merge e voltar ao estado anterior:

```bash
git merge --abort
```

---

## 7. .gitignore e arquivos ignorados

Crie um arquivo `.gitignore` na raiz do projeto:

```gitignore
# Local History (VS Code)
.history/

# Build / PlatformIO
.pio/
build/
*.o
*.hex
*.bin

# Ambiente Python
__pycache__/
*.pyc
venv/
.env

# Sistema
.DS_Store
Thumbs.db
```

Se um arquivo já estava sendo rastreado **antes** de entrar no `.gitignore`, ele continua sendo rastreado. Para parar de rastrear sem apagar do disco:

```bash
git rm --cached .history -r
git commit -m "Para de rastrear .history/"
```

---

## 8. Desfazendo coisas

| Situação                                                                              | Comando                                   |
| --------------------------------------------------------------------------------------- | ----------------------------------------- |
| Desfazer mudanças não staged em um arquivo                                            | `git restore arquivo.cpp`               |
| Tirar um arquivo do staging (mantendo a edição)                                       | `git restore --staged arquivo.cpp`      |
| Editar a mensagem do último commit                                                     | `git commit --amend -m "Nova mensagem"` |
| Desfazer o último commit mas manter as mudanças no working directory                  | `git reset --soft HEAD~1`               |
| Desfazer o último commit e o staging, mantendo as edições nos arquivos               | `git reset --mixed HEAD~1` (padrão)    |
| Apagar completamente o último commit e as mudanças (⚠️ destrutivo)                  | `git reset --hard HEAD~1`               |
| Criar um commit novo que desfaz um commit antigo (seguro para histórico compartilhado) | `git revert <hash-do-commit>`           |

**Regra prática:** use `reset` em branches locais que ninguém mais usa. Use `revert` em branches já compartilhadas/publicadas (como `main`), porque ele não reescreve histórico.

---

## 9. Stash, tags e cherry-pick

### Stash — guardar mudanças temporariamente

```bash
git stash                      # guarda as mudanças e limpa o working directory
git stash list                 # lista os stashes salvos
git stash pop                  # aplica o último stash e remove ele da lista
git stash apply stash@{0}      # aplica sem remover
git stash drop stash@{0}       # descarta um stash específico
```

Útil quando você está no meio de uma mudança e precisa trocar de branch rapidamente sem commitar algo incompleto.

### Tags — marcar versões

```bash
git tag v1.0.0                             # tag simples no commit atual
git tag -a v1.0.0 -m "Primeira release"    # tag anotada (recomendada, guarda autor/data/mensagem)
git push origin v1.0.0                     # envia uma tag específica
git push origin --tags                     # envia todas as tags
```

### Cherry-pick — trazer um commit específico de outra branch

```bash
git cherry-pick <hash-do-commit>
```

Útil quando você quer só **um** commit de uma branch, sem trazer o resto do histórico dela.

---

## 10. Rebase interativo (avançado)

Rebase reescreve o histórico "reaplicando" seus commits em cima de outro ponto.

```bash
git rebase main            # reaplica os commits da branch atual em cima do main
```

### Rebase interativo — limpar histórico antes de um PR

```bash
git rebase -i HEAD~4     # edita os últimos 4 commits
```

Abre um editor com opções por commit:

```
pick   → mantém o commit como está
reword → mantém o commit, mas edita a mensagem
edit   → pausa ali para você alterar o conteúdo
squash → funde este commit com o anterior (mantém as duas mensagens)
fixup  → funde com o anterior e descarta a mensagem deste
drop   → remove o commit completamente
```

**Regra de ouro:** nunca dê rebase em commits que já foram enviados (`push`) e que outras pessoas já usam — isso reescreve hashes e bagunça quem já baixou aquele histórico. Rebase é seguro em branches locais/pessoais antes do primeiro push, ou em branches de feature só suas.

Se der conflito no meio do rebase:

```bash
# resolva o conflito no arquivo, depois:
git add arquivo-resolvido.cpp
git rebase --continue
# ou, para desistir de tudo:
git rebase --abort
```

---

## 11. Reflog

O `reflog` guarda um histórico de **tudo** que o HEAD já apontou — inclusive commits "perdidos" depois de um `reset --hard` ou rebase malfeito. É o botão de pânico.

```bash
git reflog
```

Saída tipo:

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~1
e4f5g6h HEAD@{1}: commit: Adiciona leitura do sensor
```

Para recuperar um commit "perdido":

```bash
git reset --hard e4f5g6h
```

O reflog expira por padrão em ~90 dias, mas na prática cobre praticamente qualquer acidente recente.

---

## 12. Submódulos e Git LFS

### Submódulos — repositório dentro de outro repositório

```bash
git submodule add https://github.com/zatelectronic/lib-sensor.git libs/sensor
git submodule update --init --recursive     # ao clonar um repo que tem submódulos
```

Usado quando um projeto de firmware depende de uma biblioteca que também é um repositório separado.

### Git LFS (Large File Storage) — arquivos grandes (imagens de alta resolução, binários, datasets)

```bash
git lfs install
git lfs track "*.bin"
git add .gitattributes
```

Evita que arquivos grandes inchem o histórico do `.git`.

---

## 13. Hooks do Git

Scripts que rodam automaticamente em certos eventos. Ficam em `.git/hooks/` (não versionados por padrão).

Exemplo — `pre-commit` que impede commit se houver `TODO` no código:

```bash
#!/bin/sh
# .git/hooks/pre-commit
if git diff --cached | grep -q "TODO"; then
  echo "Commit bloqueado: existe um TODO pendente."
  exit 1
fi
```

Depois: `chmod +x .git/hooks/pre-commit`

Para compartilhar hooks com o time, ferramentas como [Husky](https://typicode.github.io/husky/) (Node) versionam os hooks dentro do projeto.

---

## 14. GitHub: recursos avançados

### Autenticação: SSH vs Token (PAT)

**SSH (recomendado para uso diário):**

```bash
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"
cat ~/.ssh/id_ed25519.pub    # copiar e colar em GitHub → Settings → SSH and GPG keys
```

Depois, use URLs no formato `git@github.com:zatelectronic/repo.git` em vez de `https://`.

**Personal Access Token (PAT):** usado quando o Git pede usuário/senha via HTTPS (senha normal não funciona mais desde 2021). Gerado em GitHub → Settings → Developer settings → Personal access tokens. Pode ser "fine-grained" (escopo restrito a repos específicos) ou "classic".

> ⚠️ Trate um PAT como uma senha: nunca commite ele no código, e prefira sempre dar um prazo de expiração, exceto em casos específicos (ex: integração automatizada como o Vercel, onde token sem expiração é aceitável porque fica guardado como variável de ambiente segura).

### Pull Requests (PR)

1. Cria uma branch (`git switch -c feature/nome`)
2. Commita e dá push (`git push -u origin feature/nome`)
3. No GitHub, abre um Pull Request da branch para `main`
4. Depois de revisado/aprovado, faz o merge (pode ser "Merge commit", "Squash and merge" ou "Rebase and merge" — GitHub oferece os três direto na interface)

### GitHub Actions (CI/CD)

Arquivo em `.github/workflows/nome.yml`, exemplo simples (o que você já usa pro snake animation, por exemplo):

```yaml
name: Exemplo de Workflow
on:
  push:
    branches: [main]
  schedule:
    - cron: "0 0 * * *"   # todo dia à meia-noite

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Rodando alguma tarefa automatizada"
```

### GitHub Pages

Publica um site estático direto de um repositório (Settings → Pages → escolher branch/pasta). Ótimo para portfólio, documentação ou changelog de projetos de firmware.

### Issues e Projects

- **Issues:** rastreamento de bugs/tarefas, com labels, assignees e milestones.
- **Projects (kanban):** organiza issues/PRs em quadros (To do / In progress / Done).

Fechar uma issue automaticamente por um commit:

```bash
git commit -m "Corrige leitura do sensor de temperatura, closes #12"
```

---

## 15. Fluxos de trabalho

### Git Flow (projetos com releases bem definidas)

- `main` → sempre estável, o que está em produção
- `develop` → integração das próximas features
- `feature/*` → uma branch por funcionalidade, nasce de `develop`
- `release/*` → prepara uma versão antes de ir pra `main`
- `hotfix/*` → correção urgente direto em `main`

### Trunk-Based Development (mais comum hoje, principalmente em projetos pessoais/pequenos times)

- Uma branch principal (`main`), branches de feature **curtas** (dias, não semanas)
- Merge frequente, PRs pequenos
- Mais simples de manter, menos "branch hell"

Para projetos individuais como o seu (firmware + PCB), trunk-based com branches curtas de feature costuma ser suficiente e mais leve que Git Flow completo.

---

## 16. Boas práticas de commits

**Formato recomendado (Conventional Commits):**

```
tipo(escopo): descrição curta no imperativo

Corpo explicando o "porquê", se necessário.
```

Tipos comuns: `feat` (funcionalidade nova), `fix` (correção de bug), `docs` (documentação), `refactor`, `test`, `chore` (manutenção, configs).

Exemplos:

```
feat(firmware): adiciona leitura de temperatura via I2C
fix(pcb-gerber): corrige footprint invertido do conector J3
docs(readme): atualiza seção de tecnologias
```

**Regras gerais:**

- Commits pequenos e focados (um commit = uma mudança lógica)
- Mensagem no imperativo ("adiciona", não "adicionado" ou "adicionando")
- Nunca commitar credenciais, tokens ou arquivos de build gerados

---

## 17. Diagnóstico e produtividade

```bash
git blame arquivo.cpp              # quem mudou cada linha e em qual commit
git log -p arquivo.cpp             # histórico completo de mudanças de um arquivo
git log --author="Kenneth"         # commits de uma pessoa específica
git log --since="2 weeks ago"      # commits recentes
git bisect start                   # busca binária pra achar qual commit introduziu um bug
  git bisect bad                   # marca o commit atual como com bug
  git bisect good <hash-antigo>    # marca um commit antigo como sem bug
  # o Git vai pulando pelo histórico até você achar o commit exato
git worktree add ../pasta-extra branch-x   # trabalha em duas branches ao mesmo tempo, em pastas separadas
```

---

## Referência rápida — os 15 comandos que resolvem 90% do dia a dia

```bash
git status
git add .
git commit -m "mensagem"
git push
git pull
git switch -c nova-branch
git switch main
git merge nova-branch
git log --oneline --graph --all
git diff
git stash / git stash pop
git restore arquivo
git reset --soft HEAD~1
git revert <hash>
git remote -v
```

---

*Manual de referência — Git & GitHub, PT-BR.*
