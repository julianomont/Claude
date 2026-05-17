# Exemplos Práticos de Skills

> Cenários reais de uso de cada skill — copie, adapte e execute.

← [[00 - MENU PRINCIPAL]] | Ver referência: [[05 - Skills e Fluxos Avançados]]

---

## `/init` — Inicializar contexto do projeto

### Cenário 1: Primeiro dia num projeto existente
Você acabou de clonar um repositório legado e quer que o Claude entenda o projeto antes de começar a trabalhar.

```
# No terminal, dentro da pasta do projeto:
claude

# Na sessão:
/init
```

O Claude vai varrer o repositório e gerar `CLAUDE.md`. Depois, edite manualmente:

```markdown
## Decisões de Arquitetura
- Usamos CQRS no módulo de pedidos — não misture reads e writes
- Redis é usado apenas para cache de sessão, não como banco principal
- Toda comunicação entre serviços passa pelo event bus (não chamadas diretas)

## Regras do Time
- PRs precisam de aprovação de dois devs antes do merge
- Migrations nunca fazem drop de coluna — apenas deprecate + nova coluna
```

---

### Cenário 2: Projeto cresceu, CLAUDE.md está desatualizado
O projeto migrou de REST para GraphQL há 3 sprints mas o CLAUDE.md ainda fala de endpoints REST.

```
/init
```

Revise o arquivo gerado, remova o que ficou obsoleto e adicione o novo contexto.

---

## `/review` — Code review antes do PR

### Cenário 1: Revisão simples de feature branch
Você terminou uma feature e quer pegar problemas antes do code review humano.

```
/review
```

Output esperado (resumido):
```
## Revisão do branch feature/checkout-flow

### Problemas Críticos
- `src/checkout/payment.ts:47` — valor do cartão logado em console.log (exposição de dados)

### Sugestões
- `src/checkout/cart.ts:12` — cálculo de desconto pode ter floating point issue
- Falta teste para o caso de carrinho vazio

### Positivo
- Tipagem bem feita, sem any
- Separação de responsabilidades clara
```

---

### Cenário 2: Revisão de PR do GitHub
Seu colega abriu o PR #87 e você quer uma pré-leitura antes de revisar manualmente.

```
/review 87
```

---

### Cenário 3: Revisão com contexto específico
Você quer que o review foque em um aspecto específico.

```
/review

# Após o review automático, direcione:
"foque especialmente nos handlers de erro — estamos tendo timeouts em prod"
```

---

### Cenário 4: Revisão antes de mergear hotfix
Hotfix urgente que não pode quebrar prod. Use `/review` seguido de `/security-review`.

```
/review
# Leia o output, corrija o que for crítico

/security-review
# Confirme que não há vulnerabilidade introduzida
```

---

## `/security-review` — Análise de segurança

### Cenário 1: Nova rota de autenticação
Você implementou login com OAuth e quer garantir que não há falha.

```
# Antes de fazer o PR:
/security-review
```

O Claude vai focar em:
- Validação do `state` parameter (CSRF em OAuth)
- Expiração de tokens
- Armazenamento seguro de refresh tokens
- Rate limiting no endpoint

---

### Cenário 2: Endpoint que recebe upload de arquivo
```
# Após implementar o upload:
/security-review

# Output esperado incluirá:
# - Validação de tipo MIME (não apenas extensão)
# - Limite de tamanho
# - Sanitização do nome do arquivo
# - Onde o arquivo é armazenado (path traversal)
```

---

### Cenário 3: Nova dependência de terceiro adicionada
Você adicionou 3 pacotes npm novos e quer verificar se há riscos.

```
/security-review

# Depois complemente:
"analise especificamente os pacotes adicionados no package.json — 
verifique se são mantidos ativamente e se há CVEs conhecidos"
```

---

### Cenário 4: Fluxo completo antes de release
```
# 1. Review geral
/review

# 2. Segurança
/security-review

# 3. Se houver mudanças sensíveis, revisão profunda
/ultrareview
```

---

## `/ultrareview` — Revisão multi-agente profunda

### Cenário 1: PR de refatoração de módulo crítico
Você refatorou o módulo de pagamentos — alto risco, precisa de revisão rigorosa.

```
/ultrareview
```

Diferente do `/review`, o ultrareview roda agentes em paralelo que analisam:
- Um agente foca em correctness e edge cases
- Outro foca em segurança
- Outro verifica consistência com o restante da codebase

---

### Cenário 2: Revisão de PR de outro time
O time de infra abriu o PR #203 com mudanças no pipeline de deploy. Você precisa revisar mas não é expert em infra.

```
/ultrareview 203
```

---

## `/simplify` — Simplificação pós-implementação

### Cenário 1: Função que cresceu demais durante o desenvolvimento
Você implementou uma feature iterativamente e o código ficou com muitos if/else e variáveis temporárias.

```
# Depois de confirmar que os testes passam:
/simplify
```

O Claude vai identificar:
- Variáveis intermediárias desnecessárias
- Condições que podem ser simplificadas
- Duplicação entre funções próximas
- Nomes que podem ser mais descritivos

---

### Cenário 2: Limpar antes de abrir PR
```
# Fluxo recomendado:
# 1. Implementa a feature
# 2. Testes passando
/simplify
# 3. Revisa as sugestões de simplificação
# 4. Aceita as que fazem sentido
/review
# 5. Corrige o que o review encontrou
# 6. Abre o PR
```

---

### Cenário 3: Simplificar um arquivo específico
```
@src/services/orderProcessing.ts
/simplify

"foque especialmente na função processPayment — ela está com 80 linhas"
```

---

## `/schedule` — Tarefas automáticas recorrentes

### Cenário 1: Review diário de PRs abertos
```
/schedule

# No wizard que abre:
# Tarefa: "liste todos os PRs abertos há mais de 3 dias sem atividade e mande um resumo"
# Frequência: toda manhã às 9h
# Notificação: email ou slack
```

---

### Cenário 2: Análise semanal de cobertura de testes
```
/schedule

# Tarefa: "rode npm test -- --coverage e analise quais módulos 
#          estão abaixo de 70% de cobertura. Liste os 5 piores"
# Frequência: toda segunda às 8h
```

---

### Cenário 3: Geração automática de changelog
```
/schedule

# Tarefa: "toda sexta às 17h, gere um changelog com os commits da semana
#          agrupados por tipo (feat, fix, refactor) e salve em CHANGELOG.md"
# Frequência: sexta-feira às 17h
```

---

### Cenário 4: Tarefa única agendada (não recorrente)
```
/schedule

# Tarefa: "amanhã às 10h, lembre-me de revisar a migration de banco
#          antes do deploy de produção das 14h"
# Frequência: uma vez
```

---

## `/loop` — Monitoramento ativo na sessão

### Cenário 1: Acompanhar deploy em andamento
```
/loop 2m "! kubectl get pods -n production | grep -v Running"

# A cada 2 minutos mostra pods que NÃO estão Running
# Você vê quando todos ficarem OK e para o loop com Ctrl+C
```

---

### Cenário 2: Monitorar testes flaky
Você suspeita que um teste falha intermitentemente.

```
/loop 30s "! npm test -- --filter=checkout.spec.ts --reporter=dot"

# Roda o teste a cada 30 segundos
# Na primeira falha, para e analisa o output
```

---

### Cenário 3: Acompanhar build de CI
```
/loop 1m "! gh run list --limit 1 --json status,conclusion --jq '.[0]'"

# Verifica o status do último GitHub Actions run a cada 1 minuto
```

---

### Cenário 4: Revisão periódica durante sessão longa
```
/loop 20m /review

# A cada 20 minutos faz um review do que foi commitado
# Útil em pair programming ou hackathons
```

---

## `/claude-api` — Desenvolvimento com a API Anthropic

### Cenário 1: Criar um script de análise de código com a API
```
/claude-api

"crie um script Python que usa a API da Anthropic para analisar 
arquivos TypeScript passados como argumento e retornar uma lista 
de possíveis bugs. Use prompt caching para o system prompt."
```

---

### Cenário 2: Migrar código de modelo antigo
```
@src/ai/analyzer.py
/claude-api

"esse script usa claude-3-opus-20240229 que foi descontinuado. 
Migre para claude-opus-4-7 e verifique se há mudanças de API necessárias"
```

---

### Cenário 3: Adicionar streaming a uma feature existente
```
@src/chat/completion.ts
/claude-api

"adicione suporte a streaming nesse handler — o usuário precisa ver 
a resposta sendo gerada em tempo real"
```

---

## `/frontend-design` — Interfaces com identidade visual

### Cenário 1: Dashboard de métricas
```
/frontend-design

"crie um dashboard React para exibir métricas de um sistema de e-commerce:
- Total de pedidos do dia (com variação em relação a ontem)
- Receita do mês (gráfico de linha)
- Top 5 produtos mais vendidos (barra horizontal)
- Pedidos por status (pizza)
Paleta: tons de azul escuro e verde. Fonte: Inter."
```

---

### Cenário 2: Formulário multi-etapas
```
/frontend-design

"crie um formulário de checkout em 3 etapas para uma loja de moda:
1. Dados pessoais + endereço
2. Método de pagamento (cartão ou PIX)  
3. Revisão e confirmação
Com barra de progresso no topo e animação de transição entre etapas.
Stack: Next.js + Tailwind."
```

---

### Cenário 3: Componente de lista com filtros
```
@src/types/product.ts
/frontend-design

"crie um componente de listagem de produtos com:
- Filtro por categoria (chips clicáveis)
- Ordenação por preço/relevância
- Cards com imagem, nome, preço e botão de adicionar ao carrinho
- Skeleton loading enquanto carrega
Use o tipo Product definido no arquivo acima."
```

---

## Combinando Skills — Fluxos Completos

### Fluxo: Feature completa do início ao PR

```bash
# 1. Antes de começar — contexto do projeto
/init         # se ainda não existir o CLAUDE.md

# 2. Implementar a feature (trabalho normal com o Claude)
"implemente o módulo de notificações por email conforme o ticket #234"

# 3. Limpar o código
/simplify

# 4. Revisão técnica
/review

# 5. Se tocar em auth ou dados sensíveis
/security-review

# 6. Abrir o PR (via gh cli)
! gh pr create --title "feat: módulo de notificações por email" --body "..."
```

---

### Fluxo: Onboarding em projeto novo

```bash
# 1. Clonar e entrar no projeto
! git clone <repo>
! cd <projeto>

# 2. Gerar documentação inicial
/init

# 3. Editar CLAUDE.md com contexto que o Claude não inferiu

# 4. Primeira análise do código
"faça um tour pelo código — explique a arquitetura geral e os principais módulos"

# 5. Identificar débitos técnicos
"liste os principais débitos técnicos que você encontrou"
```

---

### Fluxo: Revisão de segurança antes de release

```bash
# 1. Revisão geral de tudo que vai no release
/review

# 2. Foco em segurança
/security-review

# 3. Para mudanças críticas, revisão profunda
/ultrareview

# 4. Checar se as correções foram aplicadas
! git diff main...HEAD --stat
/review
```

---

*Ver também: [[05 - Skills e Fluxos Avançados]] | [[04 - Flags de Inicialização CLI]] | [[06 - Referência de Arquivos e Shell]]*
