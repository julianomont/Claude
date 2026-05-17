# Referência de Arquivos e Shell

> Como integrar o filesystem e o terminal diretamente nas conversas com o Claude.

← [[00 - Principal]]

---

## `!` — Executar Comandos de Shell

**O que faz:** Executa um comando de shell diretamente a partir do prompt do Claude Code, trazendo o output de volta para a conversa.

**Quando usar:**
- Para rodar um comando e pedir ao Claude para interpretar o output
- Para executar testes antes de pedir ajuda com os erros
- Para dar contexto do estado atual do sistema sem copiar/colar manualmente

**Sintaxe:**
```
! git log --oneline -10
! npm test
! cat src/config.ts
```

**Exemplo de fluxo:**
```
! npm test
→ (output dos testes aparece na conversa)
"corrija os 3 testes que falharam"
```

**Outro uso importante:** Quando o Claude pede que você rode algo interativo (login, auth), use `!` para trazer o resultado de volta sem sair da sessão:
```
! gcloud auth login
! aws configure
```

---

## `@` — Incluir Conteúdo de Arquivo

**O que faz:** Lê e inclui o conteúdo de um arquivo na mensagem enviada ao Claude.

**Quando usar:**
- Para referenciar código sem precisar copiar/colar
- Para dar contexto de múltiplos arquivos simultaneamente
- Para pedir análise ou modificação de arquivo específico

**Sintaxe:**
```
@caminho/para/arquivo.ts
```

**Exemplos:**
```
@src/auth/middleware.ts explique o que esse middleware faz

@package.json @tsconfig.json quais configurações preciso mudar para suportar ESM?

@src/routes/users.ts adicione validação Zod em todos os endpoints
```

**Com glob (quando suportado):**
```
@src/api/*.ts liste todos os endpoints e seus métodos HTTP
```

---

## Diferença entre `@` e pedir ao Claude para ler

| Método | Quando usar |
|---|---|
| `@arquivo.ts` na mensagem | Você sabe exatamente qual arquivo — inclui imediatamente |
| Pedir "leia o arquivo X" | Claude usa a tool Read — mais flexível, pode incluir contexto adicional |
| `!cat arquivo.ts` | Quer ver o output no terminal E incluir na conversa |

---

## Pipes e Stdin

**O que faz:** Você pode passar conteúdo via pipe para o Claude Code no modo `--print`.

**Quando usar:**
- Para processar output de outro comando
- Para analisar logs, diffs, ou JSONs grandes

```bash
git diff main...HEAD | claude -p "que problemas você vê nesse diff?"

cat error.log | claude -p "classifique esses erros por severidade"

curl -s https://api.exemplo.com/data | claude -p "normalize esse JSON para o schema em @src/types/data.ts"
```

---

## Boas Práticas de Referência

### Dar contexto antes de pedir mudanças
```
# Ruim
"corrija o bug no login"

# Bom
@src/auth/login.ts @src/auth/session.ts
! npm test -- --filter auth
"corrija o erro de session expirada que aparece nos testes acima"
```

### Incluir arquivos de tipo junto com código
```
@src/types/user.ts @src/services/userService.ts
"adicione um método para buscar usuário por email"
```

### Usar `!` para dar contexto de runtime
```
! docker ps
! kubectl get pods
"qual pod parece estar com problema?"
```

---

## Referência de Sintaxe Rápida

| Símbolo | Função | Exemplo |
|---|---|---|
| `@` | Inclui arquivo | `@src/index.ts` |
| `!` | Executa shell | `! git status` |
| `#` | Referência a memória | `#contexto-auth` |
| `--` | Separador de flags | `claude -p -- "analise"` |

---

*Ver também: [[04 - Flags de Inicialização CLI]] | [[02 - Comandos de Contexto e Memória]]*
