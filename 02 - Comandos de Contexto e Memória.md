# Comandos de Contexto e Memória

> Como dar ao Claude conhecimento persistente sobre seu projeto e preferências.

← [[00 - Principal]]

---

## `/init`

**O que faz:** Analisa o repositório atual e gera um arquivo `CLAUDE.md` com documentação do projeto — stack, estrutura, convenções, comandos importantes.

**Quando usar:**
- **Primeira vez** que usa Claude Code em um projeto
- Quando o projeto mudou significativamente e o `CLAUDE.md` está desatualizado
- Para garantir que qualquer sessão futura entenda o contexto do projeto sem você precisar explicar

**Como funciona:** O Claude lê arquivos como `package.json`, `README.md`, estrutura de pastas, arquivos de config e gera um `CLAUDE.md` estruturado na raiz.

```
/init
```

**O que o `CLAUDE.md` gerado contém:**
- Stack tecnológica
- Estrutura de diretórios
- Comandos de build, test e lint
- Convenções de código do projeto
- Instruções específicas para o Claude seguir

**Dica:** Edite o `CLAUDE.md` gerado para adicionar contexto que o Claude não conseguiu inferir — decisões de arquitetura, regras de negócio, padrões do time.

---

## `/memory`

**O que faz:** Abre o arquivo de memória persistente do usuário para visualização e edição.

**Quando usar:**
- Para revisar o que o Claude "lembra" entre sessões
- Para corrigir uma memória errada que está afetando as respostas
- Para adicionar uma preferência manualmente

**Onde fica:** `~/.claude/CLAUDE.md` (memória global) ou `.claude/CLAUDE.md` (memória do projeto)

```
/memory
```

**Hierarquia de memória:**
```
~/.claude/CLAUDE.md          → preferências globais (todos os projetos)
~/.claude/projects/<dir>/    → memória por projeto
.claude/CLAUDE.md            → instruções específicas do repositório
CLAUDE.md                    → contexto do projeto (gerado pelo /init)
```

**Exemplo de memória útil para salvar:**
```markdown
- Prefiro respostas sem emojis
- Sempre use TypeScript strict mode
- Testes usam Vitest, não Jest
- Commits seguem Conventional Commits
```

---

## `@arquivo` — Referência de Arquivos

**O que faz:** Inclui o conteúdo de um arquivo diretamente na mensagem enviada ao Claude.

**Quando usar:**
- Para pedir análise de um arquivo específico sem navegar até ele
- Para dar contexto de múltiplos arquivos de uma vez
- Para referenciar configs, schemas ou tipos que o Claude precisa conhecer

**Sintaxe:**
```
@src/auth/login.ts qual é o problema nessa função?

@package.json @tsconfig.json configure o projeto para strict mode

@src/components/Button.tsx refatore esse componente para usar Tailwind
```

**Dica:** Combine `@` com glob patterns quando disponível:
```
@src/api/*.ts documente todos esses endpoints
```

---

## `#` — Referência a Memórias/Contextos

**O que faz:** Acessa memórias salvas ou contextos nomeados diretamente na conversa.

**Quando usar:**
- Para trazer um contexto específico salvo sem abrir `/memory`

```
#projeto-auth descreva o fluxo de autenticação
```

---

## Boas Práticas de Contexto

### CLAUDE.md bem escrito economiza tokens e melhora respostas

```markdown
# Projeto: Sistema de Faturamento

## Stack
- Backend: Node.js 20 + TypeScript + Fastify
- DB: PostgreSQL com Drizzle ORM
- Testes: Vitest + Supertest
- Deploy: Railway

## Comandos
- `npm run dev` — inicia em modo dev
- `npm test` — roda todos os testes
- `npm run db:push` — aplica migrations

## Convenções
- Arquivos de rota em `src/routes/`
- Schemas Zod junto ao handler
- Nunca usar `any` — usar `unknown` + type guard
```

### Quando o Claude não está entendendo o projeto
1. Rode `/init` para regenerar o `CLAUDE.md`
2. Edite manualmente com o contexto que falta
3. Use `@CLAUDE.md` na próxima mensagem se necessário

---

*Ver também: [[05 - Skills e Fluxos Avançados]] | [[06 - Referência de Arquivos e Shell]]*
