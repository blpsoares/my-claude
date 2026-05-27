# PDD — Parity-Driven Development

Skill pack para auditar paridade entre um **sistema novo** (refactor, rewrite, port) e um **sistema de referência** (legado, spec original, sistema anterior).

Útil em qualquer migração onde a fidelidade comportamental precisa ser rastreada sistematicamente.

## Instalação

PDD só faz sentido instalado por projeto — não tem utilidade global.

**1. Registre o marketplace uma vez por máquina** (se ainda não fez):

```
/plugin marketplace add blpsoares/my-claude
```

**2. Dentro do projeto, instale com escopo de projeto:**

```bash
claude plugin install pdd@blpsoares-my-claude --scope project
```

Isso grava em `.claude/settings.json` — quem clonar o repo herda o plugin automaticamente.

## Skills

| Comando | Fase | Quando usar |
|---|---|---|
| `/audit-bootstrap` | Setup | Uma vez, no início. Entrevista para gerar `.audit/BOOTSTRAP.md` |
| `/audit-new` | Capturar | Ao achar divergência — entrevista mão-dupla dev + Claude |
| `/audit-investigate` | Investigar | Entende a causa raiz; só diagnostica, não corrige |
| `/audit-resolve` | Resolver | Implementa o fix; nunca commita/pusha |
| `/audit-status` | Ler | Dashboard do estado geral dos findings |
| `/audit-qa` | Entregar ao QA | Cria cards de teste no Notion depois do PR mergeado |

## Ciclo

```
/audit-bootstrap        ← roda uma vez, configura o projeto
/audit-new <desc>       ← acha divergência → cria finding
/audit-investigate NNN  ← entende a causa
/audit-resolve NNN      ← corrige + valida
(dev commita e abre PR)
/audit-qa NNN           ← cria cards de QA no Notion
```

## Estrutura de arquivos gerada

```
.audit/
├── BOOTSTRAP.md              ← contexto operacional do projeto
├── board.md                  ← kanban leve
├── findings/NNN-<slug>/      ← findings abertos
│   ├── README.md
│   ├── investigation.md
│   ├── resolution.md
│   └── refs/
└── resolved/NNN-<slug>/      ← findings resolvidos
```
