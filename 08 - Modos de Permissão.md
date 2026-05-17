# Modos de Permissão

> Como controlar o que o Claude pode fazer de forma autônoma — equilíbrio entre velocidade e segurança.

← [[00 - Principal]]

---

## Conceito

O Claude Code pode ler arquivos, editar código, executar comandos de shell, acessar a web e mais. Os **modos de permissão** controlam se ele pede aprovação antes de cada ação ou age de forma autônoma.

---

## Modos Disponíveis

### `default` (padrão)
O Claude pergunta antes de executar ações relevantes — edições de arquivo, comandos de shell, acessos à rede.

**Quando usar:** No dia a dia. Mantém você no controle sem precisar revisar cada detalhe.

```bash
claude                              → usa modo default
claude --permission-mode default
```

---

### `acceptEdits`
O Claude pode editar e criar arquivos sem pedir confirmação. Ainda pergunta antes de executar comandos de shell.

**Quando usar:**
- Quando você confia no Claude para fazer refatorações e não quer aprovar cada edição
- Em tarefas de geração de código onde você vai revisar o diff depois
- Para acelerar sessões de implementação sem abrir mão do controle sobre execuções

```bash
claude --permission-mode acceptEdits
```

---

### `bypassPermissions`
O Claude executa tudo sem pedir confirmação — edições, shell, rede.

**Quando usar:**
- Em scripts e automações onde não há humano disponível para aprovar
- Em ambientes isolados (container, VM descartável) onde erros têm baixo impacto
- Para tarefas bem definidas onde você já revisou o plano

**Cuidado:** Use apenas quando o escopo está claro e você pode reverter os resultados. Não recomendado para sessões exploratórias.

```bash
claude --permission-mode bypassPermissions "aplique todas as correções de lint no projeto"
```

---

## Permissões Granulares por Ferramenta

Além dos modos globais, você pode controlar ferramentas individuais com as flags `--allowedTools` e `--disallowedTools`.

### Permitir apenas ferramentas específicas
```bash
# Claude só pode ler e editar, sem executar shell
claude -p "refatore src/" --allowedTools Read,Edit,Glob,Grep

# Claude pode escrever testes mas não rodá-los
claude -p "gere testes para auth.ts" --allowedTools Read,Write
```

### Bloquear ferramentas específicas
```bash
# Claude pode tudo exceto executar comandos de shell
claude --disallowedTools Bash

# Sem acesso à internet
claude --disallowedTools WebSearch,WebFetch
```

---

## Configuração Persistente via Settings

Para não precisar passar flags toda vez, configure permissões no `settings.json`:

```json
// .claude/settings.json (projeto) ou ~/.claude/settings.json (global)
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(git status)",
      "Bash(git diff*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push*)"
    ]
  }
}
```

**Padrões suportados:**
- `Bash(npm*)` — qualquer comando npm
- `Edit(src/*)` — editar arquivos em src/
- `Write(*.test.ts)` — criar arquivos de teste

> Para configurar via skill: `/update-config` ou [[05 - Skills e Fluxos Avançados]]

---

## Matriz de Decisão

```
Contexto                              → Modo recomendado
─────────────────────────────────────────────────────────
Desenvolvimento interativo diário     → default
Refatoração com revisão de diff       → acceptEdits
CI/CD pipeline automatizado           → bypassPermissions + --allowedTools restrito
Script de análise (só leitura)        → --allowedTools Read,Glob,Grep
Geração de documentação               → --allowedTools Read,Write
Tarefa explorativa em prod            → default (nunca bypass em prod)
```

---

## Segurança: Boas Práticas

1. **Nunca use `bypassPermissions` em produção** — sempre em ambientes isolados
2. **Prefira `--allowedTools` restrito** em automações ao invés de `bypassPermissions` irrestrito
3. **Revise o `settings.json`** periodicamente — permissões acumulam com o tempo
4. **Use `/doctor`** se suspeitar de comportamento inesperado de permissões
5. **Commits atômicos** — se o Claude fizer muitas mudanças em bypass, fica difícil reverter

---

## Verificar Permissões Atuais

```
/status         → mostra modo de permissão da sessão atual
/config         → abre configurações para ajustar padrões
```

---

*Ver também: [[04 - Flags de Inicialização CLI]] | [[01 - Comandos de Sessão]]*
