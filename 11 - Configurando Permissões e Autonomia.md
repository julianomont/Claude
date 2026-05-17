# Configurando Permissões e Autonomia

> Como configurar o Claude para parar de pedir permissão para tudo — com segurança.

← [[00 - Principal]] | Ver modos globais: [[08 - Modos de Permissão]]

---

## Por que o Claude pede permissão

O Claude pede confirmação antes de executar qualquer ação que possa mutar estado: editar arquivos, rodar comandos de shell, acessar a rede. Isso é o comportamento `default` — seguro, mas lento para desenvolvimento.

Há três camadas de controle:

```
1. Modo de permissão global   → define o comportamento geral da sessão
2. Allowlist no settings.json → permite comandos específicos sem prompt
3. Flags de inicialização     → sobrescreve tudo para aquela execução
```

---

## Camada 1 — Modo de Permissão Global

Configure no `settings.json` ou passe como flag:

```json
// ~/.claude/settings.json
{
  "permissionMode": "acceptEdits"
}
```

| Modo | Claude edita arquivos | Claude roda shell | Quando usar |
|---|---|---|---|
| `default` | pede | pede | Exploração, projetos novos |
| `acceptEdits` | **sem prompt** | pede | Desenvolvimento diário |
| `bypassPermissions` | **sem prompt** | **sem prompt** | Scripts, CI, sessões confiáveis |

**Recomendação para desenvolvimento diário:** `acceptEdits` — você mantém controle sobre execuções de shell mas não fica aprovando cada edição de arquivo.

---

## Camada 2 — Allowlist no settings.json

A allowlist define quais comandos específicos o Claude pode executar **sem pedir**, independente do modo global.

### Onde fica o arquivo

| Arquivo | Escopo |
|---|---|
| `~/.claude/settings.json` | Global — vale para todos os projetos |
| `.claude/settings.json` | Projeto — só neste repositório (recomendado) |

### Estrutura do arquivo

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(docker compose up *)",
      "Bash(kubectl get *)"
    ],
    "deny": [
      "Bash(git push *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

---

## Sintaxe dos Padrões

### Bash

```
Bash(comando)           → exato — só esse comando
Bash(comando *)         → prefixo — comando + qualquer argumento
Bash(npm run test)      → exato com subcomando
Bash(npm run test *)    → subcomando com qualquer argumento
```

**Exemplos:**

```json
"Bash(npm test)"              // só "npm test" sem argumentos
"Bash(npm test *)"            // npm test + quaisquer flags
"Bash(docker compose *)"      // qualquer subcomando do docker compose
"Bash(kubectl get *)"         // kubectl get pods, nodes, services...
"Bash(bun run typecheck)"     // exato — só esse script
```

**Atenção: o espaço antes do `*` é obrigatório para prefixo funcionar:**
```json
"Bash(git log *)"   ← correto — cobre "git log --oneline", "git log main..HEAD"
"Bash(git log*)"    ← errado  — não funciona como esperado
```

### Ferramentas MCP

Use o nome completo da ferramenta, sem wildcard:

```json
"mcp__slack__slack_read_thread"
"mcp__github__get_pull_request"
"mcp__filesystem__read_file"
```

### Ferramentas do Claude Code

```json
"Read"        // leitura de arquivos — raramente precisa, já é permissiva
"Edit"        // edição de arquivos
"Write"       // criação de arquivos
"WebSearch"   // busca na web
"WebFetch"    // fetch de URL
```

---

## O que o Claude JÁ permite automaticamente (sem precisar de regra)

Estes comandos **nunca geram prompt** — adicionar uma regra para eles no `allow` é desnecessário:

### Comandos sempre auto-permitidos
`ls`, `cat`, `head`, `tail`, `find`, `echo`, `printf`, `cd`, `pwd`, `which`, `date`, `df`, `du`, `wc`, `grep`, `rg`, `jq`, `sort`, `uniq`, `diff`, `sed` (somente leitura), `tree`, `file`, `stat`, `ps`, `lsof`, `env`, `hostname`

### Git — todos os subcomandos de leitura
`git status`, `git log`, `git diff`, `git show`, `git blame`, `git branch`, `git tag`, `git remote`, `git ls-files`, `git stash list`, `git reflog`, `git describe`, e todos os demais que não mutam o repositório

### GitHub CLI — todos os subcomandos de leitura
`gh pr view`, `gh pr list`, `gh pr diff`, `gh issue view`, `gh issue list`, `gh run view`, `gh run list`, `gh repo view`, `gh auth status`, `gh release list`

### Docker — leitura
`docker ps`, `docker images`, `docker logs`, `docker inspect`

> **Regra prática:** Se o comando só lê e não muda nada, o Claude provavelmente já permite. Só adicione ao allowlist se ainda estiver recebendo prompt.

---

## Camada 3 — Flags de Inicialização

Para sessões pontuais sem alterar configuração permanente:

```bash
# Aceita edições sem prompt, mas ainda pergunta para shell
claude --permission-mode acceptEdits

# Aceita tudo — útil para scripts
claude --permission-mode bypassPermissions "aplique todo o lint"

# Libera ferramentas específicas para aquela execução
claude -p "rode os testes e corrija as falhas" \
  --allowedTools Bash,Read,Edit
```

---

## Método Automático: `/fewer-permission-prompts`

A forma mais prática de configurar o allowlist sem precisar fazer manualmente:

```
/fewer-permission-prompts
```

**O que essa skill faz:**
1. Varre os últimos 50 arquivos de transcrição de sessões (`~/.claude/projects/`)
2. Extrai todos os comandos Bash e chamadas MCP que você realmente usa
3. Filtra apenas os de leitura/seguro (descarta writes, deletes, pushes)
4. Remove os que o Claude já auto-permite (seriam entradas inúteis)
5. Gera uma tabela ranqueada por frequência de uso
6. Adiciona automaticamente ao `.claude/settings.json` do projeto atual

**Output típico:**

| # | Padrão | Contagem | Notas |
|---|---|---|---|
| 1 | `Bash(npm run dev)` | 38 | start do servidor de desenvolvimento |
| 2 | `Bash(docker compose up *)` | 24 | ambiente local |
| 3 | `Bash(kubectl get *)` | 19 | inspeção do cluster |
| 4 | `mcp__github__get_pull_request` | 11 | consulta de PRs |

> Execute `/fewer-permission-prompts` no início de um projeto para gerar uma allowlist baseada em uso real, não em suposição.

---

## Configurando via Skill `/update-config`

Para editar o `settings.json` através de linguagem natural:

```
/update-config

"adicione permissão para npm test e npm run lint sem prompt"
"mova a permissão de git push para o settings global"
"bloqueie qualquer comando rm -rf"
"defina acceptEdits como modo padrão"
```

Mais rápido do que editar o JSON manualmente.

---

## O que NUNCA colocar no allowlist

### Nunca use wildcard em interpretadores de linguagem

```json
// PERIGOSO — equivale a execução de código arbitrário
"Bash(python3 *)"
"Bash(node *)"
"Bash(bash *)"
"Bash(sh *)"
"Bash(eval *)"

// PERIGOSO — package runners aceitam qualquer script
"Bash(npx *)"
"Bash(bunx *)"
"Bash(uvx *)"

// PERIGOSO — task runners genéricos
"Bash(npm run *)"
"Bash(make *)"
```

**Por quê:** `Bash(python3 *)` permite `python3 malicious_script.py` com o mesmo nível de confiança que `python3 --version`. O wildcard anula qualquer garantia de segurança.

**Alternativa segura — use o nome exato do script:**
```json
// SEGURO — só esse script específico
"Bash(python3 scripts/analyze.py)"
"Bash(npm run typecheck)"
"Bash(npm run test:unit)"
```

### Nunca coloque no allowlist global

```json
// Não faça isso em ~/.claude/settings.json
"Bash(git push *)"      // push vai para remoto — ação irreversível
"Bash(rm *)"            // deleção — sem volta
"Bash(docker rm *)"     // remove containers
"Bash(kubectl delete *)" // destrói recursos de cluster
```

Ações irreversíveis ou que afetam sistemas externos devem sempre requerer confirmação.

---

## Configuração Recomendada por Tipo de Projeto

### Projeto Node.js / TypeScript

```json
// .claude/settings.json
{
  "permissionMode": "acceptEdits",
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm test *)",
      "Bash(npm run typecheck)",
      "Bash(npm run lint)",
      "Bash(npm run build)",
      "Bash(npx tsc --noEmit)"
    ],
    "deny": [
      "Bash(npm publish *)",
      "Bash(npm run deploy *)"
    ]
  }
}
```

### Projeto com Docker / Kubernetes

```json
{
  "permissionMode": "acceptEdits",
  "permissions": {
    "allow": [
      "Bash(docker compose up *)",
      "Bash(docker compose down)",
      "Bash(docker compose logs *)",
      "Bash(docker build *)",
      "Bash(kubectl apply *)",
      "Bash(kubectl rollout *)"
    ],
    "deny": [
      "Bash(kubectl delete *)",
      "Bash(docker rm *)",
      "Bash(docker rmi *)"
    ]
  }
}
```

### Script / CI (máxima autonomia)

```json
{
  "permissionMode": "bypassPermissions",
  "permissions": {
    "deny": [
      "Bash(git push *)",
      "Bash(npm publish *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

> No CI, use `bypassPermissions` com um `deny` cirúrgico para bloquear apenas as ações verdadeiramente destrutivas.

---

## Configuração Global vs Projeto

### Global (`~/.claude/settings.json`)
Para comportamentos que você quer em **todo projeto**:

```json
{
  "permissionMode": "acceptEdits",
  "permissions": {
    "allow": [
      "Bash(npm run typecheck)",
      "Bash(npm run lint)"
    ]
  }
}
```

### Projeto (`.claude/settings.json`)
Para regras específicas do repositório. Esse arquivo deve ser commitado:

```json
{
  "permissions": {
    "allow": [
      "Bash(./scripts/seed-db.sh)",
      "Bash(make test)"
    ],
    "deny": [
      "Bash(make deploy)"
    ]
  }
}
```

As configurações do projeto **somam** às globais — elas não se sobrescrevem.

---

## Fluxo de Setup Recomendado

```
1. Na primeira sessão de um projeto novo:
   → /fewer-permission-prompts
   → (revise e ajuste a allowlist gerada)

2. Adicione ao settings.json global o modo padrão:
   → /update-config "defina acceptEdits como modo padrão global"

3. Para projetos com comandos específicos:
   → edite .claude/settings.json manualmente ou via /update-config
   → commit o arquivo junto com o repositório

4. Para sessões de automação pontuais:
   → claude --permission-mode bypassPermissions "tarefa aqui"

5. Periodicamente:
   → /fewer-permission-prompts (atualiza com novos padrões de uso)
   → /doctor (diagnostica problemas de permissão inesperados)
```

---

## Diagnóstico de Problemas

| Sintoma | Causa provável | Solução |
|---|---|---|
| Claude ainda pede para comandos no allow | Padrão errado (falta espaço antes do `*`) | Verifique `"Bash(cmd *)"` com espaço |
| Claude não está usando o settings.json | Arquivo no lugar errado | Confirme `.claude/settings.json` na raiz do projeto |
| Modo não persiste entre sessões | `permissionMode` só foi passado como flag | Adicione ao `settings.json` |
| Claude recusa algo no allow | Está em `deny` também (deny tem prioridade) | Remova do deny |
| Comportamento inesperado em geral | Conflito global vs projeto | Use `/status` e `/doctor` para ver qual config está ativa |

---

*Ver também: [[08 - Modos de Permissão]] | [[04 - Flags de Inicialização CLI]] | [[05 - Skills e Fluxos Avançados]]*
