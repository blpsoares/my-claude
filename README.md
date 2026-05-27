# my-claude

Conteúdo pessoal para o Claude Code — plugins instaláveis e skills avulsas.

## Estrutura

```
my-claude/
├── plugins/          ← plugins instaláveis via /plugin install
│   └── pdd/          ← PDD: Parity-Driven Development
└── skills/           ← skills avulsas (copiar manualmente para .claude/skills/)
    └── arch-docs.md
```

## Plugins

Plugins gerenciados pelo sistema de plugins do Claude Code. Instalar via `/plugin`.

### Registrar o marketplace (uma vez por máquina)

```
/plugin marketplace add blpsoares/my-claude
```

### Plugins disponíveis

| Plugin | Descrição | Instalar |
|---|---|---|
| `pdd` | PDD — Parity-Driven Development. Skills `/audit-*` para auditar paridade em migrações e rewrites. | ver abaixo |

#### pdd — instalação por escopo de projeto (recomendado)

```bash
claude plugin install pdd@blpsoares-my-claude --scope project
```

Grava em `.claude/settings.json` do projeto. Quem clonar o repo herda o plugin automaticamente.

Ver [`plugins/pdd/README.md`](plugins/pdd/README.md) para documentação completa.

## Skills avulsas

Skills em `skills/` não são gerenciadas pelo sistema de plugins — copiar manualmente para `.claude/skills/` do projeto quando precisar.

| Arquivo | Descrição |
|---|---|
| `arch-docs.md` | — |
