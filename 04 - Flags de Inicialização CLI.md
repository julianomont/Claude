# Flags de Inicialização CLI

> Opções passadas ao iniciar `claude` no terminal — essenciais para automações e scripts.

← [[00 - Principal]]

---

## Referência Completa

```bash
claude [flags] [prompt]
```

---

## Flags de Sessão

### `--continue` / `-c`
**O que faz:** Retoma a última sessão salva automaticamente.

**Quando usar:**
- Para continuar exatamente onde parou sem precisar recuperar contexto
- No início do dia de trabalho para retomar a sessão anterior

```bash
claude --continue
claude -c
claude -c "continue de onde paramos no módulo de auth"
```

---

### `--resume <session-id>`
**O que faz:** Retoma uma sessão específica pelo ID (não necessariamente a última).

**Quando usar:**
- Para voltar a uma sessão de um projeto específico enquanto trabalhou em outro
- Para retomar uma sessão que não foi a mais recente

```bash
claude --resume abc123def
```

> Para ver sessões disponíveis, use o menu interativo ao iniciar com `--continue`.

---

## Flags de Output

### `--print` / `-p`
**O que faz:** Modo não-interativo — envia um prompt, recebe a resposta e encerra. Ideal para scripts e pipes.

**Quando usar:**
- Em scripts de CI/CD para análise automatizada
- Para integrar Claude em pipelines de terminal
- Quando você quer a saída em outro programa

```bash
claude -p "quais são os endpoints não documentados em src/routes?"
claude -p "gere um changelog baseado nos últimos 10 commits" | pbcopy
cat src/api.ts | claude -p "documente esta API"
```

---

### `--output-format <format>`
**O que faz:** Define o formato da saída do modo `--print`.

**Opções:**
- `text` (padrão) — texto simples
- `json` — resposta estruturada com metadados
- `stream-json` — streaming linha a linha em JSON (para pipelines em tempo real)

**Quando usar:**
- `json` para scripts que precisam processar metadados (tokens, model, custo)
- `stream-json` para exibir progresso em tempo real em automações

```bash
claude -p "analise src/auth.ts" --output-format json
claude -p "refatore esse módulo" --output-format stream-json
```

---

## Flags de Modelo

### `--model <model-id>`
**O que faz:** Define o modelo para aquela execução específica, sem alterar o padrão.

```bash
claude --model claude-opus-4-7 "revise a arquitetura desse módulo"
claude --model claude-haiku-4-5 -p "normalize esse JSON"
```

---

## Flags de Ferramentas e Permissões

### `--allowedTools <tools>`
**O que faz:** Limita quais ferramentas o Claude pode usar naquela execução.

**Quando usar:**
- Para dar permissão explícita apenas ao necessário (princípio do menor privilégio)
- Em scripts automatizados onde você quer controle total do escopo

```bash
claude -p "atualize os testes" --allowedTools Edit,Read,Bash
claude -p "gere documentação" --allowedTools Read,Write
```

**Ferramentas disponíveis:** `Read`, `Edit`, `Write`, `Bash`, `Glob`, `Grep`, `Agent`, `WebSearch`, `WebFetch`

---

### `--disallowedTools <tools>`
**O que faz:** Bloqueia ferramentas específicas, permitindo todas as outras.

```bash
claude -p "analise o código" --disallowedTools Bash,Write
```

---

### `--permission-mode <mode>`
**O que faz:** Define o nível de autonomia do Claude naquela sessão.

**Modos:**
- `default` — pergunta antes de ações relevantes
- `acceptEdits` — aceita edições de arquivo sem perguntar
- `bypassPermissions` — executa tudo sem perguntar (use com cuidado)

```bash
claude --permission-mode acceptEdits
claude --permission-mode bypassPermissions "aplique todas as correções de lint"
```

> Ver detalhes em [[08 - Modos de Permissão]]

---

## Flags de Contexto

### `--add-dir <path>`
**O que faz:** Adiciona um diretório ao contexto do Claude além do diretório atual.

**Quando usar:**
- Quando o projeto tem múltiplos repositórios relacionados
- Para incluir uma biblioteca local no contexto sem mudar de diretório

```bash
claude --add-dir ../shared-types
claude --add-dir ../api --add-dir ../frontend
```

---

### `--system-prompt <texto>`
**O que faz:** Define um system prompt customizado para aquela sessão, sobrescrevendo o padrão.

**Quando usar:**
- Para dar um papel específico ao Claude ("você é um revisor de segurança")
- Para forçar um estilo de resposta em automações

```bash
claude --system-prompt "Você é um auditor de segurança. Responda apenas em português técnico."
```

---

### `--append-system-prompt <texto>`
**O que faz:** Adiciona texto ao system prompt padrão sem substituí-lo.

```bash
claude --append-system-prompt "Sempre sugira testes para qualquer código que escrever."
```

---

## Flags de Autenticação

### `--login`
**O que faz:** Inicia o fluxo de autenticação com sua conta Anthropic ou Claude.ai.

```bash
claude --login
```

### `--logout`
```bash
claude --logout
```

---

## Exemplos de Uso em Scripts

### Análise de PR em CI
```bash
git diff main...HEAD | claude -p "revise esse diff e liste problemas críticos" --output-format json
```

### Geração de testes automatizada
```bash
claude -p "gere testes Vitest para @src/auth/login.ts" \
  --allowedTools Read,Write \
  --model claude-sonnet-4-6
```

### Pipeline de documentação
```bash
for file in src/routes/*.ts; do
  claude -p "documente $file em JSDoc" --allowedTools Read,Edit
done
```

---

*Ver também: [[08 - Modos de Permissão]] | [[05 - Skills e Fluxos Avançados]]*
