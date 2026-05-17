# Como Criar Skills

> Guia completo para criar skills personalizadas no Claude Code — do zero até publicar.

← [[00 - Principal]] | Exemplos de uso: [[09 - Exemplos Práticos de Skills]]

---

## O que é uma Skill

Uma skill é um arquivo `SKILL.md` que ensina o Claude a executar um workflow especializado. Quando o Claude vê um prompt que corresponde à descrição da skill, ele carrega as instruções e as segue — como um agente com expertise específica naquela área.

Skills são diferentes de um prompt salvo: elas têm estrutura, recursos embutidos (scripts, referências, assets) e um mecanismo de ativação automático.

---

## Estrutura de Arquivos

```
minha-skill/
├── SKILL.md              ← obrigatório — instruções + frontmatter
├── scripts/              ← scripts reutilizáveis (Python, bash, etc.)
├── references/           ← documentação carregada sob demanda
└── assets/               ← templates, fontes, ícones usados no output
```

Apenas o `SKILL.md` é obrigatório. Os demais diretórios são opcionais e adicionados conforme a skill cresce em complexidade.

---

## Onde Guardar

| Localização | Escopo | Quando usar |
|---|---|---|
| `~/.claude/skills/minha-skill/` | Global — disponível em qualquer projeto | Skills pessoais de uso geral |
| `.claude/skills/minha-skill/` | Projeto — só no repositório atual | Skills específicas do domínio do projeto |

---

## Anatomia do SKILL.md

### Frontmatter (obrigatório)

```yaml
---
name: minha-skill
description: >
  O que essa skill faz e QUANDO usar. Esse campo é o gatilho principal
  de ativação — seja específico sobre os contextos que devem acionar a skill.
  Inclua exemplos de frases do usuário que devem dispará-la.
  Use this skill whenever the user mentions X, Y or Z, even if they
  don't explicitly ask for it.
compatibility:
  tools:
    - Bash
    - Read
    - Write
---
```

**Campos:**
- `name` — identificador da skill (sem espaços, kebab-case)
- `description` — **o campo mais importante** — controla quando a skill é ativada
- `compatibility` — ferramentas necessárias (opcional — só inclua se a skill não funcionar sem elas)

### Corpo (as instruções)

Markdown livre. Tudo o que vem depois do frontmatter são as instruções que o Claude segue ao executar a skill.

---

## Como Funciona o Carregamento (Progressive Disclosure)

O Claude carrega a skill em três camadas, da mais leve para a mais pesada:

```
Nível 1 — Metadata (sempre em contexto)
  name + description (~100 palavras)
  → Usado para decidir SE a skill deve ser ativada

Nível 2 — SKILL.md body (em contexto quando a skill ativa)
  As instruções completas (ideal: menos de 500 linhas)
  → Guia como executar o workflow

Nível 3 — Recursos embutidos (carregados sob demanda)
  scripts/, references/, assets/
  → Só carregados quando as instruções apontam para eles
```

**Implicação prática:** Mantenha o `SKILL.md` focado. Se ultrapassar 500 linhas, mova detalhes para `references/` e referencie a partir do corpo com indicação de quando ler.

---

## Escrevendo a `description` — O Gatilho

A `description` é o que determina se a skill será usada ou ignorada. O Claude lê o nome + descrição de todas as skills disponíveis e decide qual invocar com base nesse texto.

### Regras para uma boa description

**1. Inclua O QUE faz E QUANDO usar:**
```yaml
# Ruim — só diz o que faz
description: Gera documentação JSDoc para funções TypeScript.

# Bom — diz o que faz E quando acionar
description: >
  Gera documentação JSDoc completa para funções TypeScript — parâmetros,
  tipos de retorno, exemplos e @throws. Use this skill whenever the user
  asks to document TypeScript code, add JSDoc, or generate API docs,
  even if they just say "document this function".
```

**2. Seja "insistente" — o Claude tende a sub-acionar skills:**
```yaml
description: >
  Analisa pull requests do GitHub em profundidade. Make sure to use this
  skill whenever the user mentions reviewing a PR, checking a branch before
  merge, or asking for code review — even if they don't say "/review".
```

**3. Inclua variações de linguagem que o usuário pode usar:**
```yaml
description: >
  Cria migrations de banco de dados seguras. Ative sempre que o usuário
  mencionar: alterar tabela, adicionar coluna, mudar schema, migration,
  alter table, ou qualquer mudança estrutural no banco.
```

---

## Escrevendo o Corpo das Instruções

### Use a forma imperativa
```markdown
# Bom
Leia o arquivo de schema antes de gerar a migration.
Nunca faça DROP sem confirmação explícita do usuário.

# Ruim
O Claude deve ler o schema. DROP deve ser evitado.
```

### Explique o PORQUÊ, não só o QUÊ
```markdown
# Bom
Sempre valide o tipo MIME do arquivo além da extensão — extensões podem
ser falsificadas, mas o magic byte do arquivo é muito mais confiável.

# Ruim
SEMPRE valide o tipo MIME. NUNCA confie apenas na extensão.
```

LLMs são inteligentes. Explique o raciocínio e eles aplicam o princípio em casos que você não previu. Regras sem contexto criam comportamento rígido e frágil.

### Defina o formato de output quando importar
```markdown
## Formato do relatório

Use sempre esta estrutura:

# [Título da análise]
## Resumo executivo (2-3 frases)
## Problemas encontrados
  - Crítico: ...
  - Médio: ...
  - Baixo: ...
## Recomendações
## Próximos passos
```

### Inclua exemplos
```markdown
## Exemplos de commit message

**Cenário:** usuário adicionou autenticação JWT
- Input: "added JWT auth"
- Output: `feat(auth): implement JWT-based authentication`

**Cenário:** usuário corrigiu bug de cache
- Input: "fixed cache not clearing"
- Output: `fix(cache): clear stale entries on session expiry`
```

### Para skills com múltiplos domínios, use references/

```markdown
## Plataforma de deploy

Identifique a plataforma usada no projeto e leia o arquivo correspondente:
- AWS → leia `references/aws.md`
- GCP → leia `references/gcp.md`  
- Railway → leia `references/railway.md`

Só carregue o arquivo da plataforma relevante — não leia todos.
```

Isso mantém o `SKILL.md` pequeno e carrega apenas o que é necessário.

---

## Template Completo de SKILL.md

```markdown
---
name: analisar-performance
description: >
  Analisa performance de código TypeScript/JavaScript — identifica gargalos,
  operações O(n²), re-renders desnecessários e sugere otimizações com
  benchmarks estimados. Use this skill whenever the user mentions performance,
  slow code, optimization, profiling, or asks why something is slow.
---

## Objetivo

Identificar os principais gargalos de performance no código fornecido e
propor soluções concretas com estimativa de impacto.

## Processo

1. Leia os arquivos apontados pelo usuário (ou peça quais analisar)
2. Identifique padrões problemáticos (lista abaixo)
3. Estime o impacto de cada problema (alto/médio/baixo)
4. Proponha soluções com exemplos de código
5. Se houver scripts de benchmark disponíveis em `scripts/`, execute-os

## Padrões a procurar

- Loops aninhados sobre arrays grandes (O(n²) ou pior)
- Chamadas síncronas dentro de loops (devem ser extraídas para fora)
- Objetos ou arrays recriados a cada render/chamada sem memoização
- Consultas N+1 em ORM (um query por item de uma lista)
- Imports de bibliotecas inteiras quando só uma função é usada

## Formato de resposta

# Análise de Performance — [nome do arquivo]

## Resumo
[2-3 frases sobre o estado geral]

## Problemas identificados

### 🔴 Alto impacto
**[Descrição do problema]** — `arquivo.ts:linha`
```código problemático```
**Solução:** ...
```código corrigido```
**Impacto estimado:** redução de X ms / Y% de CPU

### 🟡 Médio impacto
...

## Ganho total estimado
...
```

---

## Usando o `skill-creator` para Criar Skills

Em vez de escrever do zero, use a skill oficial:

```
/skill-creator

"quero criar uma skill que [descreva o que quer fazer]"
```

O `skill-creator` vai:
1. Entender o que você quer e fazer perguntas sobre edge cases
2. Escrever um rascunho do `SKILL.md`
3. Criar casos de teste realistas
4. Rodar o Claude com e sem a skill e comparar os outputs
5. Mostrar os resultados num viewer interativo para você avaliar
6. Iterar com base no seu feedback
7. Otimizar a `description` para melhorar o acionamento

Isso é o fluxo recomendado — especialmente para skills complexas.

---

## Otimizando o Acionamento da Description

Se a skill não está sendo ativada quando deveria (ou ativa quando não deveria), o `skill-creator` tem um otimizador automático:

```
/skill-creator

"otimize a description da minha skill em ~/.claude/skills/minha-skill/"
```

Ele vai:
1. Gerar 20 prompts de teste (metade que devem acionar, metade que não devem)
2. Pedir que você revise e edite os prompts num viewer HTML
3. Rodar um loop de otimização automático (até 5 iterações)
4. Propor a melhor `description` com score de acerto

---

## Iterando e Melhorando

### Sinais de que a skill precisa de revisão

| Sintoma | Causa provável | Solução |
|---|---|---|
| Skill não ativa quando deveria | `description` muito vaga | Adicione frases que o usuário usa |
| Skill ativa quando não deveria | `description` muito ampla | Reduza o escopo, adicione negativos |
| Output inconsistente | Instruções ambíguas | Adicione exemplos e formato esperado |
| Skill muito lenta | SKILL.md muito longo | Mova conteúdo para `references/` |
| Skill ignora parte das instruções | Instruções contraditórias | Simplifique, explique o porquê de cada regra |

### Ao iterar, preserve o que funciona
Ao melhorar uma skill, identifique o que o Claude está fazendo certo e não quebre. A maioria dos problemas vem de instruções que conflitam ou de `description` mal calibrada — não do corpo das instruções.

---

## Boas Práticas

**Mantenha o escopo focado.** Uma skill que faz uma coisa bem é mais útil que uma que faz dez coisas mal. Se você precisar de múltiplos comportamentos, crie múltiplas skills.

**Não use MUST/NEVER em maiúsculas como substituto de explicação.** Se você se pegar escrevendo `SEMPRE faça X`, provavelmente é um sinal de que você deveria explicar POR QUE X é importante — aí o Claude vai aplicar o princípio corretamente em casos que você não previu.

**Escreva cases de teste antes de terminar.** Antes de considerar uma skill pronta, teste com pelo menos 3 prompts realistas — o tipo de coisa que um usuário real escreveria. Se o output não satisfizer, ajuste as instruções.

**Scripts embutidos eliminam trabalho repetido.** Se toda execução da skill acaba gerando o mesmo script auxiliar, coloque esse script em `scripts/` e referencie nas instruções — todo uso futuro vai pular essa etapa.

---

## Exemplo: Skill Completa com Recursos

```
gerador-migration/
├── SKILL.md
├── scripts/
│   └── validate_migration.py   ← valida sintaxe SQL antes de salvar
└── references/
    ├── postgres.md              ← padrões específicos de PostgreSQL
    └── mysql.md                 ← padrões específicos de MySQL
```

```yaml
---
name: gerador-migration
description: >
  Gera migrations de banco de dados seguras com rollback, testa a sintaxe
  e segue as convenções do projeto. Use this skill when the user asks to
  create a migration, alter a table, add a column, change schema, or
  anything involving database structure changes.
---

## Processo

1. Identifique o banco de dados do projeto (postgres/mysql) — leia o arquivo
   correspondente em `references/`
2. Pergunte ao usuário o que precisa mudar, se não estiver claro
3. Gere o arquivo de migration com UP e DOWN (rollback)
4. Execute `scripts/validate_migration.py` para verificar a sintaxe
5. Mostre o resultado e peça confirmação antes de salvar

## Regras críticas

Inclua sempre o DOWN (rollback) — migrations sem rollback são um risco
operacional: se algo der errado no deploy, a reversão precisa ser automática.

Nunca gere DROP de coluna diretamente — renomeie para `_deprecated_nome`
primeiro. Remoção definitiva fica para uma migration separada após confirmação
de que nenhum código usa a coluna.
```

---

*Ver também: [[05 - Skills e Fluxos Avançados]] | [[09 - Exemplos Práticos de Skills]]*
