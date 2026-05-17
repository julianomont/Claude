# Skills e Fluxos Avançados

> Skills são automações pré-configuradas que executam tarefas complexas com um único comando.

← [[00 - Principal]]

---

## O que são Skills?

Skills são comandos `/` que ativam agentes especializados com comportamento predefinido. Diferente dos comandos básicos, cada skill pode coordenar múltiplos agentes, fazer buscas, ler arquivos e produzir resultados estruturados.

---

## `/init`

**Skill de:** Documentação de projeto

**O que faz:** Analisa o repositório e gera o `CLAUDE.md` — o arquivo de contexto que o Claude lê em toda sessão nesse projeto.

**Quando usar:**
- Primeira vez usando Claude Code no projeto
- Quando o projeto evoluiu e o `CLAUDE.md` está defasado
- Ao onboar um novo dev — o `CLAUDE.md` serve como documentação viva

```
/init
```

**Output:** Arquivo `CLAUDE.md` na raiz do projeto com stack, estrutura, comandos e convenções.

> Sempre revise e enriqueça o `CLAUDE.md` gerado com decisões de arquitetura que o Claude não consegue inferir.

---

## `/review`

**Skill de:** Code Review

**O que faz:** Executa uma revisão completa do branch atual (commits à frente do branch base) — analisa lógica, padrões, bugs potenciais, legibilidade e cobertura de testes.

**Quando usar:**
- Antes de abrir um PR para pegar problemas cedo
- Como segunda opinião após sua própria revisão
- Em PRs grandes onde uma revisão manual seria lenta

```
/review                    → revisa o branch atual
/review 42                 → revisa o PR #42 do GitHub
```

**O que ele analisa:**
- Correctness — lógica e edge cases
- Segurança — vulnerabilidades óbvias
- Performance — operações custosas desnecessárias
- Padrões — consistência com o restante do código
- Testes — cobertura das mudanças

---

## `/security-review`

**Skill de:** Revisão de segurança

**O que faz:** Análise focada em segurança das mudanças do branch atual — injection, autenticação, autorização, exposição de dados, dependências vulneráveis.

**Quando usar:**
- Antes de mergear código que toca autenticação, autorização ou dados sensíveis
- Ao introduzir novas dependências de terceiros
- Em features que expõem novos endpoints públicos

```
/security-review
```

**Categorias analisadas:**
- OWASP Top 10
- Exposição de secrets/credenciais
- Validação de input
- SQL/NoSQL injection
- XSS e CSRF
- Controle de acesso

---

## `/ultrareview`

**Skill de:** Revisão multi-agente profunda

**O que faz:** Lança múltiplos agentes em paralelo para uma revisão em nuvem mais abrangente que `/review`. Faturado separadamente.

**Quando usar:**
- PRs críticos ou de alto risco
- Antes de releases importantes
- Quando você quer uma segunda opinião independente da revisão humana

```
/ultrareview              → revisa branch atual
/ultrareview 42           → revisa PR #42 do GitHub
```

> Requer repositório git. Não precisa de remote para a versão de branch local.

---

## `/schedule`

**Skill de:** Agendamento de tarefas

**O que faz:** Cria agentes remotos que executam em schedule (cron) — tarefas recorrentes sem você precisar estar presente.

**Quando usar:**
- Para rodar análises de código toda noite
- Para gerar relatórios automáticos
- Para monitorar métricas periodicamente

```
/schedule
```

**Exemplo de uso:**
```
/schedule → "toda segunda de manhã, analise os PRs abertos e envie resumo"
```

---

## `/loop`

**Skill de:** Execução recorrente

**O que faz:** Executa um prompt ou outro skill em intervalos regulares durante a sessão atual.

**Quando usar:**
- Para monitorar o status de um deploy em andamento
- Para fazer polling de uma condição até ela ser satisfeita
- Para repetir uma verificação em intervalos definidos

```
/loop 5m /review          → roda /review a cada 5 minutos
/loop 30s "status do build"
```

---

## `/simplify`

**Skill de:** Refatoração e qualidade

**O que faz:** Revisa o código alterado em busca de oportunidades de simplificação — remove duplicação, melhora nomes, elimina complexidade desnecessária.

**Quando usar:**
- Após implementar uma feature para limpar o código
- Quando um diff está maior do que deveria
- Para garantir que a solução é a mais simples possível

```
/simplify
```

---

## `/claude-api`

**Skill de:** Desenvolvimento com Claude API / Anthropic SDK

**O que faz:** Auxilia a construir, debugar e otimizar aplicações que usam a API da Anthropic — prompt caching, tool use, batch, streaming.

**Quando usar:**
- Ao escrever código que importa `anthropic` ou `@anthropic-ai/sdk`
- Para migrar código entre versões de modelo (4.5 → 4.6 → 4.7)
- Para implementar prompt caching e otimizar custos

```
/claude-api
```

---

## `/frontend-design`

**Skill de:** Interfaces frontend

**O que faz:** Gera interfaces web com alta qualidade de design — evita o estilo genérico de IA e produz código de produção com identidade visual.

**Quando usar:**
- Para criar componentes UI, páginas ou aplicações web
- Quando o resultado visual importa tanto quanto o código

```
/frontend-design
```

---

## Tabela Resumo

| Skill | Foco | Melhor momento |
|---|---|---|
| `/init` | Documentação | Início do projeto |
| `/review` | Code review geral | Antes do PR |
| `/security-review` | Segurança | Features sensíveis |
| `/ultrareview` | Revisão profunda | PRs críticos |
| `/simplify` | Qualidade de código | Pós-implementação |
| `/schedule` | Automação recorrente | Tarefas periódicas |
| `/loop` | Polling na sessão | Monitoramento ativo |
| `/claude-api` | SDK Anthropic | Projetos com a API |
| `/frontend-design` | UI/UX | Interfaces web |

---

*Ver também: [[02 - Comandos de Contexto e Memória]] | [[04 - Flags de Inicialização CLI]]*
