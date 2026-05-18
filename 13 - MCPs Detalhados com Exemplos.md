# MCPs Detalhados com Exemplos

> Guia aprofundado dos MCPs mais úteis para desenvolvimento de software — com exemplos reais de uso.

← [[00 - Principal]] | Visão geral: [[12 - Plugins e MCP para Desenvolvimento]]

---

## Context7 — Documentação Atualizada de Bibliotecas

**Instalação:**
```
/plugin install context7@claude-plugins-official
```

**Por que usar:** O Claude foi treinado até agosto de 2025. Prisma, Next.js, React e qualquer framework lançam mudanças de API constantemente. Sem Context7, o Claude pode recomendar APIs depreciadas. Com ele, busca a doc oficial em tempo real.

---

### Exemplo 1: Configurar Server Actions no Next.js 15

**Sem Context7:** Claude pode recomendar a sintaxe do Next.js 13/14.

**Com Context7:**
```
"como configuro Server Actions com validação Zod no Next.js 15?"
```

Claude consulta a doc atualizada e responde com:
```typescript
// app/actions/createUser.ts
'use server'

import { z } from 'zod'
import { revalidatePath } from 'next/cache'

const schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
})

export async function createUser(formData: FormData) {
  const result = schema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  })

  if (!result.success) {
    return { error: result.error.flatten() }
  }

  await db.user.create({ data: result.data })
  revalidatePath('/users')
}
```

---

### Exemplo 2: Migração de API entre versões do Prisma

```
"estou migrando do Prisma 4 para o 5 — quais mudanças de API devo saber?"
```

Context7 retorna as breaking changes documentadas oficialmente, incluindo:
- mudanças no cliente gerado
- novo comportamento de transações
- alterações em tipos TypeScript

---

### Exemplo 3: Debug de configuração do Tailwind v4

```
"meu purge do Tailwind não está funcionando no Vite — como configuro corretamente?"
```

Claude busca a doc do Tailwind v4 (que mudou a configuração significativamente) e responde com a sintaxe correta para aquela versão, não a do v3.

---

## GitHub MCP — Repositórios, PRs e Issues

**Instalação:**
```
/plugin install github@claude-plugins-official
```

---

### Exemplo 1: Implementar feature a partir de uma issue

```
"leia a issue #78 e implemente o que está descrito"
```

Claude:
1. Busca a issue #78 — lê título, descrição, comentários, labels e assignees
2. Analisa o código existente relacionado
3. Implementa a feature
4. Cria o commit com mensagem referenciando a issue
5. Abre o PR linkando `Closes #78`

---

### Exemplo 2: Triagem de PRs abertos

```
"liste os PRs abertos há mais de 5 dias sem review e resuma cada um"
```

Claude:
1. Busca todos os PRs abertos via GitHub MCP
2. Filtra pelos que não tiveram atividade recente
3. Lê o diff de cada um
4. Gera um resumo: o que muda, qual o risco, quanto tempo estima para revisar

**Output:**
```
## PRs aguardando review

### PR #91 — feat: adicionar rate limiting (8 dias sem review)
**Mudança:** Middleware Express com Redis para limitar requests por IP
**Risco:** Médio — toca o pipeline de autenticação
**Tempo estimado:** 20min de review
**Reviewer sugerido:** @backend-team

### PR #85 — chore: atualizar dependências (6 dias)
...
```

---

### Exemplo 3: Debugar falha de CI

```
"o CI do PR #103 está falhando — descubra o motivo e corrija"
```

Claude:
1. Lê o PR #103 via GitHub MCP
2. Busca os logs do GitHub Actions
3. Identifica o erro (ex: teste falhando, lint error, build error)
4. Lê o código relevante
5. Aplica a correção
6. Comita no branch do PR

---

### Exemplo 4: Gerar changelog entre versões

```
"gere o changelog entre as tags v2.3.0 e v2.4.0"
```

Claude:
1. Busca os commits entre as duas tags via GitHub MCP
2. Classifica por tipo (feat, fix, chore, refactor)
3. Filtra os que são relevantes para o usuário final
4. Gera o `CHANGELOG.md` formatado

```markdown
## v2.4.0 (2026-05-10)

### Novas funcionalidades
- Autenticação via Google OAuth (#89)
- Dashboard de métricas em tempo real (#92)

### Correções
- Erro ao exportar relatório com dados vazios (#95)
- Sessão expirando antes do tempo configurado (#97)

### Interno
- Migração para Drizzle ORM (#88)
```

---

## PostgreSQL MCP — Banco de Dados

**Instalação:**
```bash
claude mcp add --transport http dbhub https://mcp.dbhub.io
# ou para local:
# settings.json → mcpServers → postgres com connection string
```

---

### Exemplo 1: Debugar query lenta

```
"a busca de pedidos está demorando 8 segundos em produção — investigate"
```

Claude:
1. Pede a query suspeita (ou lê do código)
2. Executa `EXPLAIN ANALYZE` via MCP
3. Lê o plano de execução
4. Identifica o problema (ex: sequential scan em tabela de 2M rows)
5. Propõe o índice correto

```sql
-- Problema identificado: full scan em orders
-- Solução:
CREATE INDEX CONCURRENTLY idx_orders_user_status
  ON orders(user_id, status)
  WHERE deleted_at IS NULL;
```

---

### Exemplo 2: Validar migration antes de aplicar

```
"vou adicionar uma coluna NOT NULL à tabela users que tem 3M de rows — 
como fazer sem downtime?"
```

Claude:
1. Inspeciona a tabela `users` via MCP — vê estrutura, índices, constraints
2. Verifica o tamanho real da tabela
3. Propõe a migration em etapas seguras:

```sql
-- Etapa 1: adicionar nullable (sem lock longo)
ALTER TABLE users ADD COLUMN subscription_tier VARCHAR(20);

-- Etapa 2: backfill em lotes (rodar fora do horário de pico)
UPDATE users SET subscription_tier = 'free'
WHERE subscription_tier IS NULL
  AND id BETWEEN 0 AND 100000;
-- repetir em lotes de 100k

-- Etapa 3: adicionar constraint depois do backfill
ALTER TABLE users ALTER COLUMN subscription_tier SET NOT NULL;
ALTER TABLE users ALTER COLUMN subscription_tier SET DEFAULT 'free';
```

---

### Exemplo 3: Inspecionar schema para gerar tipos TypeScript

```
"gere os tipos TypeScript para todas as tabelas do banco"
```

Claude:
1. Lista todas as tabelas via MCP
2. Lê cada schema (colunas, tipos, nullable, defaults)
3. Gera os tipos com precisão:

```typescript
// Gerado a partir do schema real do banco

export interface User {
  id: number
  email: string
  name: string
  subscription_tier: 'free' | 'pro' | 'enterprise'
  created_at: Date
  deleted_at: Date | null
}

export interface Order {
  id: number
  user_id: number
  total_cents: number
  status: 'pending' | 'paid' | 'cancelled' | 'refunded'
  created_at: Date
}
```

---

### Exemplo 4: Detectar inconsistências de dados

```
"verifique se há pedidos sem usuário correspondente ou usuários sem perfil"
```

Claude executa as verificações via MCP:

```sql
-- Pedidos órfãos
SELECT COUNT(*) FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;

-- Usuários sem perfil
SELECT COUNT(*) FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
WHERE p.user_id IS NULL;
```

E reporta os resultados com sugestão de correção.

---

## Playwright MCP — Testes e Automação de Browser

**Instalação:**
```bash
npm install -g @playwright/mcp
```

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp"]
    }
  }
}
```

---

### Exemplo 1: Testar fluxo de checkout após implementação

```
"implementei o checkout — teste o fluxo completo no browser"
```

Claude:
1. Abre o browser em `http://localhost:3000`
2. Adiciona um produto ao carrinho
3. Vai para o checkout
4. Preenche dados de entrega
5. Seleciona forma de pagamento
6. Finaliza o pedido
7. Captura screenshot do resultado
8. Verifica se a mensagem de confirmação apareceu

Se algo falhar:
```
Erro encontrado: botão "Finalizar compra" não responde ao click.
Causa: evento onClick não está sendo registrado — o botão está 
dentro de um form sem preventDefault().
Correção aplicada em src/components/Checkout/SubmitButton.tsx:24
```

---

### Exemplo 2: Auditar acessibilidade de uma página

```
"verifique problemas de acessibilidade na página de login"
```

Claude:
1. Abre `http://localhost:3000/login`
2. Lê a árvore de acessibilidade (ARIA roles, labels, tab order)
3. Reporta problemas:

```
Problemas de acessibilidade encontrados:

❌ Input de email sem label associado (aria-label ou <label for>)
❌ Botão de mostrar/ocultar senha sem texto alternativo
❌ Mensagem de erro não está em role="alert"
⚠️  Tab order não segue a ordem visual (email → senha → submit)
✅ Contraste de cores adequado (4.8:1)
✅ Formulário tem role="form" e aria-label
```

---

### Exemplo 3: Capturar screenshots para comparação visual

```
"compare o visual do componente Card nos estados: default, hover, loading e error"
```

Claude:
1. Abre o Storybook em `http://localhost:6006`
2. Navega até o componente Card
3. Captura screenshot de cada estado
4. Descreve diferenças visuais e possíveis problemas

---

### Exemplo 4: Automatizar tarefa repetitiva no browser

```
"preciso exportar os relatórios dos últimos 12 meses do painel interno — 
automatize isso"
```

Claude:
1. Abre o painel interno
2. Faz login com as credenciais fornecidas
3. Navega até Relatórios
4. Itera pelos 12 meses, clica em Exportar em cada um
5. Confirma os downloads

---

## Firebase MCP — Backend como Serviço

**Instalação:**
```
/plugin install firebase@claude-plugins-official
```

---

### Exemplo 1: Estruturar coleção do Firestore

```
"crie a estrutura de coleções para um sistema de delivery — 
usuários, restaurantes, pedidos e itens do cardápio"
```

Claude cria os documentos de exemplo e define a estrutura:

```
/users/{userId}
  - name: string
  - email: string
  - address: map
  - createdAt: timestamp

/restaurants/{restaurantId}
  - name: string
  - cuisine: string[]
  - isOpen: boolean
  - rating: number

/restaurants/{restaurantId}/menu/{itemId}
  - name: string
  - price: number
  - category: string
  - available: boolean

/orders/{orderId}
  - userId: string (ref)
  - restaurantId: string (ref)
  - items: array
  - total: number
  - status: 'pending' | 'accepted' | 'preparing' | 'delivered'
  - createdAt: timestamp
```

E já escreve as regras de segurança correspondentes.

---

### Exemplo 2: Escrever regras de segurança do Firestore

```
"configure as regras de segurança: usuário só vê seus próprios dados, 
restaurante só edita seu próprio cardápio, admin vê tudo"
```

Claude:
1. Lê a estrutura atual do Firestore via MCP
2. Escreve as regras:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth.token.role == 'admin';
    }

    function isOwner(userId) {
      return request.auth != null && request.auth.uid == userId;
    }

    match /users/{userId} {
      allow read, write: if isOwner(userId) || isAdmin();
    }

    match /restaurants/{restaurantId} {
      allow read: if true;
      allow write: if isAdmin() ||
        request.auth.uid == resource.data.ownerId;

      match /menu/{itemId} {
        allow read: if true;
        allow write: if isAdmin() ||
          request.auth.uid == get(/databases/$(database)/documents/
            restaurants/$(restaurantId)).data.ownerId;
      }
    }

    match /orders/{orderId} {
      allow read: if isOwner(resource.data.userId) || isAdmin();
      allow create: if isOwner(request.resource.data.userId);
      allow update: if isAdmin();
    }
  }
}
```

3. Aplica via MCP diretamente no projeto

---

### Exemplo 3: Debugar consulta lenta no Firestore

```
"a consulta de pedidos do usuário está lenta — o índice está configurado?"
```

Claude:
1. Lê a query no código
2. Inspeciona os índices configurados no Firebase via MCP
3. Identifica o índice faltante
4. Cria o índice composto necessário
5. Verifica se a consulta melhorou

---

## Sentry MCP — Erros de Produção

**Instalação:**
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

---

### Exemplo 1: Investigar erro de produção

```
"temos um TypeError em prod desde ontem às 14h — investigue e corrija"
```

Claude:
1. Busca issues recentes no Sentry criadas após as 14h de ontem
2. Lê o stack trace completo do erro
3. Lê o código no arquivo e linha indicados
4. Analisa o contexto (qual usuário, qual request, qual dado causou)
5. Propõe e aplica o fix

**Stack trace que Claude leria:**
```
TypeError: Cannot read properties of undefined (reading 'price')
  at calculateTotal (src/services/cart.ts:47)
  at POST /api/checkout (src/routes/checkout.ts:23)

Contexto: cart.items retornou undefined para userId=8823
Causa: race condition — usuário deletou item enquanto checkout processava
Fix: adicionar optional chaining e validação antes do cálculo
```

---

### Exemplo 2: Priorizar bugs pela frequência e impacto

```
"liste os 5 erros mais frequentes desta semana e classifique por impacto no usuário"
```

Claude:
1. Busca as issues abertas da semana no Sentry
2. Ordena por frequência de ocorrência
3. Analisa qual % de usuários afetados
4. Gera o relatório de priorização:

```
## Top 5 Erros — Semana 20/2026

| # | Erro | Ocorrências | Usuários afetados | Impacto |
|---|---|---|---|---|
| 1 | ChunkLoadError no dashboard | 234 | 18% | Alto |
| 2 | 500 em /api/export | 89 | 3% | Médio |
| 3 | Timeout no relatório mensal | 45 | 1% | Baixo |
...
```

---

### Exemplo 3: Correlacionar erro com deploy

```
"esse erro começou no deploy de hoje? ou já existia antes?"
```

Claude:
1. Busca a release atual no Sentry
2. Compara a frequência do erro antes e depois do deploy
3. Verifica quais arquivos mudaram no deploy
4. Conclui se é regressão do deploy ou problema pré-existente

---

## Filesystem MCP — Operações Avançadas de Arquivo

**Instalação:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/caminho/permitido"]
    }
  }
}
```

---

### Exemplo 1: Auditar estrutura do projeto

```
"analise a estrutura de pastas e identifique inconsistências ou arquivos fora do padrão"
```

Claude:
1. Lista recursivamente a estrutura via MCP
2. Compara com o padrão definido no CLAUDE.md
3. Reporta inconsistências:

```
Inconsistências encontradas:

- src/components/button.tsx → deveria ser Button.tsx (PascalCase)
- src/utils/formatDate.js → arquivo .js em projeto TypeScript
- src/pages/admin/ → sem index.ts (outros módulos têm)
- .env.local commitado no repositório (remover imediatamente)
```

---

### Exemplo 2: Refatoração em múltiplos arquivos

```
"renomeie a entidade 'Customer' para 'Client' em todo o projeto, 
mantendo as convenções de cada contexto"
```

Claude:
1. Busca todas as ocorrências de `Customer`, `customer`, `CUSTOMER`
2. Analisa o contexto de cada uma (nome de variável, tipo, rota, comentário)
3. Aplica as substituições corretas mantendo a casing adequada
4. Lista todas as mudanças realizadas

---

## Memory MCP — Conhecimento Persistente Entre Sessões

**Instalação:**
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

---

### Exemplo 1: Manter contexto de decisões de arquitetura

```
"lembre que decidimos usar CQRS no módulo de pedidos e que 
o Redis é só para cache de sessão, nunca como banco principal"
```

Claude salva no grafo de conhecimento. Na próxima sessão:
```
"onde devo salvar os dados do carrinho?"
→ Claude recupera a memória e responde: 
"Segundo a decisão registrada, o Redis é reservado para sessão. 
O carrinho deve ser salvo no banco principal com TTL ou como draft order."
```

---

### Exemplo 2: Acumular contexto de debugging

```
"lembre que o bug #234 estava relacionado ao timezone — 
datas vindas do frontend sem offset UTC causavam erros de cálculo"
```

Nas próximas sessões, ao surgir erros de data:
```
→ Claude recupera o contexto e já verifica o timezone antes de qualquer outra coisa
```

---

## Sequential Thinking MCP — Raciocínio Estruturado

**Instalação:**
```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

**Para que serve:** Força o Claude a pensar passo a passo antes de agir — especialmente útil em problemas complexos ou decisões de arquitetura.

---

### Exemplo 1: Planejar migração de banco de dados

```
"preciso migrar 50M de registros da tabela orders para uma nova estrutura 
sem downtime — como fazer?"
```

Com Sequential Thinking, Claude raciocina em etapas antes de propor:
1. Analisa o tamanho e a estrutura atual
2. Considera janelas de manutenção e SLAs
3. Avalia estratégias (dual write, shadow table, blue/green)
4. Estima riscos de cada abordagem
5. Propõe o plano com rollback

---

### Exemplo 2: Debugar problema difícil de reproduzir

```
"esse bug acontece só em produção, 1 vez por hora, não aparece em staging — 
como investigar?"
```

Claude pensa estruturadamente:
1. O que produção tem que staging não tem? (volume, dados reais, concorrência)
2. O que acontece 1x por hora? (jobs agendados, cache TTL, rate limits)
3. Quais logs correlacionam com o horário?
4. Hipóteses em ordem de probabilidade
5. Como isolar cada hipótese

---

## Combinações Poderosas de MCPs

### Stack de desenvolvimento completa
```
Context7 + GitHub + Playwright + PostgreSQL

Fluxo:
1. Claude lê issue no GitHub
2. Consulta Context7 para APIs da lib usada
3. Escreve o código
4. Verifica o schema do banco via PostgreSQL MCP
5. Testa no browser com Playwright
6. Abre o PR no GitHub
```

### Stack de debug de produção
```
Sentry + GitHub + PostgreSQL

Fluxo:
1. Sentry: lê o stack trace e contexto do erro
2. GitHub: vê qual commit introduziu a mudança
3. PostgreSQL: verifica se os dados do banco explicam o erro
4. Claude aplica o fix e abre o PR
```

### Stack Firebase full-stack
```
Firebase + Playwright + Context7

Fluxo:
1. Context7: verifica API atual do Firebase SDK
2. Firebase MCP: cria estrutura, regras e indexes
3. Claude gera o código frontend e backend
4. Playwright: testa o fluxo completo no browser
```

---

## Tabela de Referência Rápida

| MCP | Melhor uso | Comando de exemplo |
|---|---|---|
| **Context7** | Documentação atualizada | "como uso X no framework Y versão Z?" |
| **GitHub** | Issues, PRs, CI | "implemente a issue #42 e abra o PR" |
| **PostgreSQL** | Schema, queries, debug | "por que essa query está lenta?" |
| **Playwright** | Testes, automação visual | "teste o fluxo de login no browser" |
| **Firebase** | Backend sem servidor | "configure o Firestore para esse schema" |
| **Sentry** | Erros de produção | "qual o erro mais frequente desta semana?" |
| **Memory** | Contexto persistente | "lembre que X sempre causa Y neste projeto" |
| **Sequential Thinking** | Decisões complexas | "planeje a migração sem downtime" |
| **Filesystem** | Operações em massa | "renomeie X para Y em todo o projeto" |

---

*Ver também: [[12 - Plugins e MCP para Desenvolvimento]] | [[11 - Configurando Permissões e Autonomia]] | [[09 - Exemplos Práticos de Skills]]*
