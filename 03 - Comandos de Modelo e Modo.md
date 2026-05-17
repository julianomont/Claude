# Comandos de Modelo e Modo

> Controle qual modelo usar e como o Claude se comporta na sessão.

← [[00 - Principal]]

---

## `/model`

**O que faz:** Exibe o modelo atual ou troca de modelo durante a sessão.

**Quando usar:**
- Para alternar entre velocidade (Haiku) e capacidade (Opus/Sonnet) conforme a tarefa
- Quando você quer economizar tokens em tarefas simples
- Para usar o modelo mais capaz em tarefas críticas de arquitetura ou refatoração

```
/model                          → exibe modelo atual
/model claude-sonnet-4-6        → troca para Sonnet 4.6
/model claude-opus-4-7          → troca para Opus 4.7 (mais capaz)
/model claude-haiku-4-5         → troca para Haiku 4.5 (mais rápido/barato)
```

### Guia de escolha de modelo

| Tarefa | Modelo recomendado | Motivo |
|---|---|---|
| Geração de boilerplate, CRUD | Haiku 4.5 | Rápido, econômico |
| Debug, testes, refatoração | Sonnet 4.6 | Equilíbrio custo/capacidade |
| Arquitetura, código complexo, revisão | Opus 4.7 | Máxima capacidade |
| Sessões longas de desenvolvimento | Sonnet 4.6 | Melhor custo-benefício |

---

## `/fast`

**O que faz:** Ativa o **Fast Mode** — usa Claude Opus com saída mais rápida, sem downgrade para modelo menor.

**Quando usar:**
- Quando você está usando Opus e quer respostas mais ágeis sem mudar de modelo
- Em fluxos interativos onde latência importa mais que throughput máximo

> Disponível para Opus 4.6 e Opus 4.7. Toggle: execute `/fast` novamente para desativar.

```
/fast
```

---

## `/vim`

**O que faz:** Ativa o modo de edição Vim no input do terminal, para navegar e editar prompts com keybindings Vim.

**Quando usar:**
- Se você é usuário Vim e quer editar prompts longos com eficiência
- Para usar movimentos `hjkl`, `w`, `b`, `gg`, `G` no campo de texto

```
/vim        → ativa modo Vim
/vim        → desativa (toggle)
```

**Atalhos principais no modo Vim:**
- `Esc` → modo normal
- `i` → modo insert
- `A` → ir ao fim da linha em insert
- `dd` → apagar linha
- `u` → desfazer

---

## `/config`

**O que faz:** Abre o menu de configurações do Claude Code — tema, modelo padrão, modo de permissão, etc.

**Quando usar:**
- Para configurar o modelo padrão que abre em toda sessão nova
- Para mudar o tema do terminal
- Para ajustar configurações persistentes sem editar arquivos manualmente

```
/config
```

**Configurações disponíveis:**
- Modelo padrão
- Tema (dark/light/system)
- Modo de auto-aprovação de ferramentas
- Integração com IDE

---

## `/terminal-setup`

**O que faz:** Configura o terminal para exibir o Claude Code corretamente — instala fontes Nerd Font, ajusta encoding, configura cores.

**Quando usar:**
- Na primeira instalação em um novo ambiente
- Quando caracteres especiais aparecem quebrados no terminal
- Para melhorar a experiência visual do CLI

```
/terminal-setup
```

---

## `/ide`

**O que faz:** Gerencia a integração do Claude Code com IDEs (VS Code, JetBrains).

**Quando usar:**
- Para conectar o Claude Code à extensão da sua IDE
- Para sincronizar o contexto entre terminal e editor
- Quando a extensão da IDE não está reconhecendo o Claude CLI

```
/ide
```

---

*Ver também: [[04 - Flags de Inicialização CLI]] | [[08 - Modos de Permissão]]*
