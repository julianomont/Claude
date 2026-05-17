# Comandos de Sessão

> Comandos que controlam o estado da conversa atual com o Claude.

← [[00 - Principal]]

---

## `/clear`

**O que faz:** Apaga todo o histórico da conversa atual e inicia uma sessão limpa.

**Quando usar:**
- Quando o contexto ficou irrelevante e você quer começar uma tarefa completamente diferente
- Quando a conversa está muito longa e degradando a qualidade das respostas
- Ao trocar de projeto ou domínio dentro da mesma sessão do terminal

**Cuidado:** Não há undo. Todo o histórico é perdido.

```
/clear
```

---

## `/compact`

**O que faz:** Comprime o histórico da conversa em um resumo, liberando espaço no contexto sem perder o fio condutor.

**Quando usar:**
- Em sessões longas onde o contexto está chegando no limite
- Quando você quer manter o contexto do projeto mas descartar detalhes de troca de mensagens
- Antes de pedir uma tarefa grande que vai precisar de muito contexto livre

**Diferença para `/clear`:** `/compact` preserva o essencial; `/clear` apaga tudo.

```
/compact
```

---

## `/cost`

**O que faz:** Exibe o consumo estimado de tokens e custo da sessão atual.

**Quando usar:**
- Para monitorar gasto em sessões longas
- Antes de decidir usar `/compact` (para saber se já é necessário)
- Para estimar custo de fluxos automatizados com `--print`

```
/cost
```

**Output típico:**
```
Tokens used:  12,450 input / 3,200 output
Estimated cost: $0.04
```

---

## `/exit`

**O que faz:** Encerra o Claude Code CLI.

**Quando usar:** Para sair da sessão interativa de forma limpa. Equivalente a `Ctrl+D`.

```
/exit
```

---

## `/status`

**O que faz:** Mostra o estado atual da sessão — modelo ativo, modo de permissão, diretório de trabalho.

**Quando usar:**
- Para confirmar qual modelo está sendo usado
- Para checar o modo de permissão antes de tarefas sensíveis

```
/status
```

---

## `/release-notes`

**O que faz:** Exibe as notas da versão atual do Claude Code.

**Quando usar:** Para descobrir novos comandos, features ou mudanças de comportamento após uma atualização.

```
/release-notes
```

---

## `/doctor`

**O que faz:** Verifica a saúde da instalação — credenciais, versão, conectividade, MCP servers.

**Quando usar:**
- Quando o Claude não está respondendo como esperado
- Após atualizar o Claude Code
- Para diagnosticar problemas de MCP ou permissões

```
/doctor
```

---

*Ver também: [[07 - Atalhos de Teclado]] | [[03 - Comandos de Modelo e Modo]]*
