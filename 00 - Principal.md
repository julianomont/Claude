# Claude CLI — Menu Principal

> Vault de referência para uso otimizado do Claude Code CLI no desenvolvimento de sistemas.

---

## Navegação Rápida

| Arquivo | Conteúdo | Quando usar |
|---|---|---|
| [[01 - Comandos de Sessão]] | `/clear`, `/compact`, `/exit`, `/cost` | Gerenciar a conversa ativa |
| [[02 - Comandos de Contexto e Memória]] | `/init`, `/memory`, `@arquivo` | Dar contexto ao Claude sobre o projeto |
| [[03 - Comandos de Modelo e Modo]] | `/model`, `/fast`, `/vim` | Ajustar comportamento e performance |
| [[04 - Flags de Inicialização CLI]] | `--continue`, `--print`, `--model` | Controlar o Claude via terminal/scripts |
| [[05 - Skills e Fluxos Avançados]] | `/review`, `/init`, `/security-review` | Automações e revisões de código |
| [[06 - Referência de Arquivos e Shell]] | `!comando`, `@path`, `#` | Integrar shell e arquivos à conversa |
| [[07 - Atalhos de Teclado]] | Ctrl+C, Ctrl+R, setas | Navegar e controlar sem digitar |
| [[08 - Modos de Permissão]] | `--permission-mode` | Controlar o que o Claude pode fazer |
| [[09 - Exemplos Práticos de Skills]] | cenários reais por skill | Ver como cada skill se aplica no dia a dia |
| [[10 - Como Criar Skills]] | SKILL.md, frontmatter, description, skill-creator | Criar suas próprias skills personalizadas |
| [[11 - Configurando Permissões e Autonomia]] | settings.json, allowlist, modos, auto-allow | Eliminar prompts desnecessários |
| [[12 - Plugins e MCP para Desenvolvimento]] | Context7, GitHub, Postgres, Playwright, Firebase | Conectar Claude a ferramentas externas |
| [[13 - MCPs Detalhados com Exemplos]] | exemplos reais por MCP, fluxos combinados | Referência aprofundada com casos de uso |

---

## Fluxos Recomendados por Situação

### Iniciando um novo projeto
1. `claude` — abre o CLI
2. `/init` — gera o `CLAUDE.md` com contexto do projeto
3. `/memory` — verifica memórias persistentes relevantes

### Revisando código antes de PR
1. `/review` — executa revisão do branch atual
2. `/security-review` — análise de segurança das mudanças

### Sessão longa com muito contexto
1. `/compact` — comprime o histórico mantendo contexto essencial
2. `/cost` — verifica consumo de tokens

### Rodando Claude em scripts/CI
```bash
claude --print "analise esse arquivo" --output-format json
claude -p "gere testes para src/auth.ts" --allowedTools Edit,Write
```

### Retomando trabalho anterior
```bash
claude --continue          # retoma última sessão
claude --resume <id>       # retoma sessão específica
```

---

## Estrutura de Arquivos da Vault

```
Claude/
├── 00 - Principal.md                    ← você está aqui
├── 01 - Comandos de Sessão.md
├── 02 - Comandos de Contexto e Memória.md
├── 03 - Comandos de Modelo e Modo.md
├── 04 - Flags de Inicialização CLI.md
├── 05 - Skills e Fluxos Avançados.md
├── 06 - Referência de Arquivos e Shell.md
├── 07 - Atalhos de Teclado.md
├── 08 - Modos de Permissão.md
├── 09 - Exemplos Práticos de Skills.md
├── 10 - Como Criar Skills.md
├── 11 - Configurando Permissões e Autonomia.md
├── 12 - Plugins e MCP para Desenvolvimento.md
└── 13 - MCPs Detalhados com Exemplos.md
```

---

*Atualizado em: maio 2026 — Claude Code (claude-sonnet-4-6)*
