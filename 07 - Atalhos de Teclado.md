# Atalhos de Teclado

> Navegação e controle do Claude Code CLI sem precisar digitar comandos.

← [[00 - Principal]]

---

## Controle de Execução

| Atalho | Ação | Quando usar |
|---|---|---|
| `Ctrl+C` | Cancela a resposta atual em andamento | Quando o Claude está no caminho errado e você quer redirecionar |
| `Ctrl+D` | Sai do Claude Code | Para encerrar a sessão (equivale a `/exit`) |
| `Ctrl+Z` | Suspende o processo (Unix) | Para pausar temporariamente e retomar depois |

**Dica sobre `Ctrl+C`:** Cancela apenas a geração atual — a sessão continua. Use para interromper uma resposta longa e redirecionar com mais contexto.

---

## Navegação no Input

| Atalho | Ação |
|---|---|
| `↑` / `↓` | Navega pelo histórico de mensagens enviadas |
| `Home` / `End` | Vai ao início / fim da linha |
| `Ctrl+A` | Vai ao início da linha |
| `Ctrl+E` | Vai ao fim da linha |
| `Ctrl+K` | Apaga do cursor até o fim da linha |
| `Ctrl+U` | Apaga a linha inteira |
| `Ctrl+W` | Apaga a palavra anterior |
| `Alt+←` / `Alt+→` | Move palavra por palavra |

---

## Edição Multi-linha

| Atalho | Ação |
|---|---|
| `Shift+Enter` | Quebra de linha sem enviar (para prompts multi-linha) |
| `Enter` | Envia a mensagem |

**Quando usar Shift+Enter:**
- Para escrever prompts longos e bem estruturados
- Para incluir blocos de código inline no prompt
- Para listar múltiplos requisitos de forma organizada

---

## Modo Vim (se ativado com `/vim`)

| Modo | Atalho | Ação |
|---|---|---|
| Normal | `i` | Entra no modo Insert |
| Normal | `A` | Insert no fim da linha |
| Normal | `o` | Nova linha abaixo em Insert |
| Normal | `dd` | Apaga linha |
| Normal | `u` | Desfaz |
| Normal | `Ctrl+R` | Refaz |
| Normal | `hjkl` | Mover cursor (←↓↑→) |
| Normal | `w` / `b` | Próxima / anterior palavra |
| Normal | `gg` / `G` | Início / fim do texto |
| Normal | `yy` / `p` | Copia linha / cola |
| Insert | `Esc` | Volta ao modo Normal |

> Para ativar: [[03 - Comandos de Modelo e Modo#/vim]]

---

## Seleção e Clipboard

| Atalho | Ação |
|---|---|
| `Ctrl+Shift+C` | Copia seleção (terminal) |
| `Ctrl+Shift+V` | Cola (terminal) |
| Clique duplo | Seleciona palavra no output |
| Clique triplo | Seleciona linha no output |

---

## Busca no Histórico (Shell)

Esses atalhos funcionam no shell que invocou o `claude`, não dentro da sessão:

| Atalho | Ação |
|---|---|
| `Ctrl+R` | Busca reversa no histórico de comandos |
| `Ctrl+G` | Cancela a busca reversa |

---

## Dicas de Produtividade

### Use `↑` para reutilizar prompts
O histórico de mensagens enviadas fica acessível com a seta para cima — útil para reenviar um prompt com pequenas modificações.

### `Ctrl+C` não perde o contexto
Ao contrário de fechar o terminal, `Ctrl+C` apenas para a geração. Você pode continuar a conversa normalmente.

### Prompts longos merecem Shift+Enter
```
Shift+Enter para multi-linha:

Preciso que você:
1. Leia src/auth/login.ts
2. Identifique todos os pontos de falha
3. Proponha correções com testes
```

---

*Ver também: [[03 - Comandos de Modelo e Modo]] | [[01 - Comandos de Sessão]]*
