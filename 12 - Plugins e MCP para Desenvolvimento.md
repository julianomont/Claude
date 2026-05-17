# Plugins e MCP para Desenvolvimento

> Os principais plugins e servidores MCP para acelerar desenvolvimento frontend e backend com o Claude Code.

← [[00 - Principal]]

---

## O que são Plugins e MCP

**MCP (Model Context Protocol)** é o padrão aberto que permite ao Claude se conectar a ferramentas externas — bancos de dados, APIs, repositórios, navegadores, serviços de cloud — diretamente na sessão, sem copiar e colar.

Cada **MCP server** expõe um conjunto de ferramentas que o Claude pode chamar:
- Ler/escrever no banco de dados
- Criar PRs no GitHub
- Executar testes no navegador
- Buscar documentação atualizada
- Consultar logs de erro em produção

**Plugins** são MCP servers pré-empacotados com autenticação e configuração simplificada, disponíveis via `/plugin install`.

```
Sem MCP: você copia dados → cola no Claude → Claude responde → você aplica
Com MCP: Claude acessa direto → age direto → resultado imediato
```

---

## Como Instalar

### Plugins oficiais (recomendado)
```
/plugin install <nome>@claude-plugins-official
/reload-plugins
```

### Via CLI (para servidores externos)
```bash
claude mcp add --transport http <nome> <url>
claude mcp add --transport stdio <nome> -- npx -y <pacote>
```

### Via settings.json (manual)
```json
// .claude/settings.json
{
  "mcpServers": {
    "meu-servidor": {
      "command": "npx",
      "args": ["-y", "@pacote/mcp-server"],
      "env": {
        "API_KEY": "sua-chave"
      }
    }
  }
}
```

### Verificar o que está instalado
```
/doctor          → diagnóstico geral incluindo MCP
/status          → servidores ativos na sessão
```

---

## Plugins por Categoria

---

### Documentação

#### Context7
**O mais importante para desenvolvimento diário.**

Busca a documentação atualizada de qualquer biblioteca ou framework diretamente durante a sessão — sem depender do conhecimento de treinamento do modelo, que pode estar desatualizado.

```
/plugin install context7@claude-plugins-official
```

**Ferramentas expostas:**
- `resolve-library-id` — encontra o ID da biblioteca no catálogo
- `query-docs` — busca trechos relevantes da documentação

**Quando usar:**
- Sempre que perguntar sobre uma biblioteca específica (React, Prisma, Fastify, Tailwind, Django, etc.)
- Para verificar mudanças de API entre versões
- Ao debugar erros de configuração de frameworks

**Como o Claude usa automaticamente:**
```
"como configuro o middleware de CORS no Fastify?"
→ Claude consulta Context7 → responde com a API atual
```

> O Context7 está sempre ativo quando instalado — o Claude o consulta automaticamente para perguntas sobre bibliotecas.

---

### Controle de Versão

#### GitHub
Acesso completo ao GitHub: repositórios, issues, PRs, Actions, releases.

```
/plugin install github@claude-plugins-official
```

**Ferramentas principais:**
- Criar, ler e atualizar issues e PRs
- Consultar diffs de commits e PRs
- Verificar status de CI/CD (GitHub Actions)
- Gerenciar branches e releases
- Buscar código em repositórios

**Fluxos que isso habilita:**
```
Claude lê a issue #45 → escreve o código → abre o PR → linka a issue
Claude vê que o CI falhou → lê o log → corrige o erro → faz push
```

#### GitLab
Equivalente ao GitHub para instâncias GitLab.

```
/plugin install gitlab@claude-plugins-official
```

**Ferramentas:** Merge requests, pipelines, issues, repositórios, CI/CD.

---

### Banco de Dados

#### PostgreSQL — DBHub (recomendado)
Acesso multi-banco com suporte a versioning e interface unificada.

```bash
claude mcp add --transport http dbhub https://mcp.dbhub.io
```

**Ferramentas:**
- Consultas SQL (SELECT, INSERT, UPDATE, DELETE)
- Inspeção de schema (tabelas, colunas, índices, constraints)
- Visualização de relacionamentos
- Comparação de schemas entre ambientes

**Quando usar:**
- Para deixar o Claude escrever e executar queries diretamente
- Para inspecionar schema ao gerar migrations
- Para validar dados durante debug

**Alternativa local (self-hosted):**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://user:pass@localhost/mydb"]
    }
  }
}
```

#### SQLite
Para projetos com banco local (mobile, desktop, testes).

```bash
npm install -g mcp-server-sqlite-npx
```

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "mcp-server-sqlite",
      "args": ["--db-path", "./database.db"]
    }
  }
}
```

#### Supabase
PostgreSQL gerenciado com Auth, Realtime, Storage e Edge Functions integrados.

```
/plugin install supabase@claude-plugins-official
```

**Ferramentas:** Banco de dados, autenticação, storage, edge functions, RLS policies.

---

### Cloud e Deploy

#### Firebase
Gerenciamento completo de projetos Firebase sem sair do Claude Code.

```
/plugin install firebase@claude-plugins-official
```

**Ferramentas:**
- Firestore (leitura/escrita de documentos)
- Realtime Database
- Hosting (deploy de sites)
- Cloud Functions
- Authentication (usuários, regras)
- Storage
- Regras de segurança

**Fluxo típico:**
```
"crie uma coleção 'pedidos' no Firestore com os campos X, Y, Z 
e configure as regras de segurança para autenticação obrigatória"
→ Claude cria a estrutura e escreve as regras diretamente
```

#### Vercel
Deploy e gerenciamento de projetos Vercel (Next.js, SvelteKit, etc.).

```bash
claude mcp add --transport http vercel https://mcp.vercel.com
```

**Ferramentas:**
- Listar e inspecionar deployments
- Ler logs de deployment e runtime
- Gerenciar variáveis de ambiente
- Inspecionar projetos e domínios

**Quando usar:**
- Para debugar falhas de build sem sair do Claude
- Para configurar env vars de prod com segurança
- Para comparar deployments

---

### Browser e Frontend

#### Playwright MCP (recomendado)
Automação de browser para testes, scraping e verificação visual. Mantido pela Microsoft.

```bash
npm install -g playwright
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

**Ferramentas:**
- Navegar para URLs
- Clicar, preencher, submeter formulários
- Capturar screenshots
- Executar JavaScript no browser
- Ler árvore de acessibilidade (a11y)
- Suporte a Chromium, Firefox, WebKit

**Fluxos que isso habilita:**
```
"implemente o checkout e teste o fluxo completo no browser"
→ Claude escreve o código → abre o browser → testa o fluxo → tira screenshot → corrige problemas

"verifique se o formulário de login tem problemas de acessibilidade"
→ Claude abre a página → analisa a árvore de acessibilidade → reporta os problemas
```

> Prefira Playwright ao Puppeteer — o servidor oficial do Puppeteer foi arquivado em 2025.

#### Figma
Acesso direto a designs para workflows design-to-code.

```
/plugin install figma@claude-plugins-official
```

**Ferramentas:**
- Ler arquivos e componentes de design
- Extrair design tokens (cores, tipografia, espaçamento)
- Ler comentários e anotações
- Exportar assets

**Fluxo:**
```
"implemente o componente Button do Figma exatamente como está no design"
→ Claude lê os tokens e specs → gera o componente
```

---

### IDE Integration

#### IDE MCP (nativo do Claude Code)
Integração com VS Code e JetBrains — já disponível sem instalação adicional.

**Ferramentas:**
- `mcp__ide__getDiagnostics` — erros e warnings do TypeScript/LSP em tempo real
- `mcp__ide__executeCode` — executa código no notebook/REPL da IDE

**Como usar:**
```
"corrija todos os erros que o TypeScript está reportando"
→ Claude consulta getDiagnostics → lê os erros da IDE → corrige
```

#### Language Servers (LSP)
Análise estática em tempo real por linguagem.

```
/plugin install typescript-lsp@claude-plugins-official
/plugin install pyright-lsp@claude-plugins-official
/plugin install rust-analyzer-lsp@claude-plugins-official
```

| Plugin | Linguagem | Binário necessário |
|---|---|---|
| `typescript-lsp` | TypeScript/JavaScript | `typescript-language-server` |
| `pyright-lsp` | Python | `pyright-langserver` |
| `rust-analyzer-lsp` | Rust | `rust-analyzer` |
| `gopls-lsp` | Go | `gopls` |
| `clangd-lsp` | C/C++ | `clangd` |

**Benefício:** O Claude vê type errors, referências e definições em tempo real — sem precisar rodar o compilador manualmente.

---

### Monitoramento e Observabilidade

#### Sentry
Acesso direto a erros de produção para debugging contextualizado.

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

**Ferramentas:**
- Buscar e listar issues por critério
- Ler stack traces completos
- Ver releases e commits associados
- Analisar frequência e impacto de erros

**Fluxo:**
```
"o erro 'TypeError: cannot read property of undefined' está acontecendo 
em prod desde o último deploy — investigue e corrija"
→ Claude consulta Sentry → lê o stack trace → lê o código → propõe fix
```

---

### Gestão de Projetos

#### Linear
Issues, projetos e ciclos de sprint.

```
/plugin install linear@claude-plugins-official
```

**Ferramentas:** Criar/atualizar issues, mover entre status, vincular a PRs, listar cycles.

#### Jira + Confluence (Atlassian)
Para times enterprise.

```
/plugin install atlassian@claude-plugins-official
```

**Ferramentas:** Issues, sprints, boards, páginas de documentação, comentários.

#### Slack
Notificações e acesso a canais.

```
/plugin install slack@claude-plugins-official
```

**Ferramentas:** Ler mensagens, enviar notificações, postar em canais, reagir a mensagens.

---

### Execução Segura de Código

#### E2B
Sandbox isolado na nuvem para executar código sem impacto local.

```bash
claude mcp add --transport http e2b https://mcp.e2b.dev
```

**Ferramentas:**
- Executar Python, JavaScript, Node.js
- Instalar pacotes dentro do sandbox
- Rodar shell commands com isolamento total

**Quando usar:**
- Para testar código gerado antes de executar localmente
- Para rodar scripts que você não quer que afetem seu ambiente
- Para validar outputs em ambiente limpo

---

## Setup Inicial Recomendado

### Para qualquer projeto (instale primeiro)
```
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
/reload-plugins
```

### Adicione conforme sua stack

| Stack | Plugins adicionais |
|---|---|
| **Frontend (React/Next.js)** | `playwright`, `figma`, `vercel` |
| **Backend (Node/Python/Go)** | LSP da linguagem, `postgres` ou `sqlite` |
| **Full-stack Firebase** | `firebase` |
| **Full-stack Supabase** | `supabase` |
| **Time com Jira** | `atlassian`, `slack` |
| **Time com Linear** | `linear`, `slack` |
| **Monitoramento** | `sentry` |

---

## Fluxos Completos com MCP

### Feature do zero ao deploy (Next.js + Vercel + GitHub)
```
1. Claude lê a issue no GitHub (#42)
2. Escreve o código da feature
3. Abre o Playwright e testa no browser
4. Corrige os problemas encontrados
5. Abre PR no GitHub linkando a issue
6. Verifica o deployment no Vercel após merge
```

### Debug de erro de produção
```
1. Claude lê o stack trace no Sentry
2. Lê o código relevante nos arquivos
3. Consulta Context7 se a biblioteca mudou de API
4. Propõe e aplica o fix
5. Atualiza o status da issue no Linear/Jira
```

### Design-to-code
```
1. Claude lê o componente no Figma
2. Extrai tokens (cores, tamanhos, espaçamento)
3. Gera o componente React/Vue
4. Abre no Playwright para comparar visualmente
5. Ajusta até o resultado estar fiel ao design
```

---

## Descobrindo Mais Plugins

```
/plugin marketplace update claude-plugins-official
/plugin       → abre o browser interativo de plugins
```

**Diretórios externos:**
- Registro oficial MCP: `registry.modelcontextprotocol.io` (500+ servidores)
- Anthropic marketplace: `claude.ai/directory` (75+ conectores)
- GitHub: `github.com/modelcontextprotocol/servers` (referências oficiais)

---

## Segurança ao Instalar MCP

Antes de instalar um servidor MCP de terceiros:

1. **Prefira servidores oficiais** (Anthropic ou do próprio vendor — Vercel, Sentry, Microsoft)
2. **Verifique o repositório** no GitHub — commits recentes, issues abertas, quem mantém
3. **Não use `latest`** em produção — pin a versão específica
4. **Limite OAuth scopes** — só conceda as permissões que o plugin realmente precisa
5. **Revise o `allow` no settings.json** após instalar — ferramentas MCP também aparecem lá

```json
// settings.json — permitir só leitura de uma ferramenta MCP
{
  "permissions": {
    "allow": [
      "mcp__sentry__get_issue",
      "mcp__sentry__list_issues"
    ],
    "deny": [
      "mcp__sentry__delete_issue"
    ]
  }
}
```

---

*Ver também: [[11 - Configurando Permissões e Autonomia]] | [[05 - Skills e Fluxos Avançados]] | [[02 - Comandos de Contexto e Memória]]*
