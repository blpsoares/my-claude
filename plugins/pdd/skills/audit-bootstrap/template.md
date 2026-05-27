# BOOTSTRAP — Auditoria de Paridade (PDD)

> **⚠️ TODA NOVA SESSÃO CLAUDE DEVE LER ESTE ARQUIVO** antes de qualquer trabalho
> de auditoria. Sem esse contexto, decisões serão tomadas sem referências.
>
> {{REGRAS_DOC_REF}}
> <!-- Se existe documento de regras do projeto, preencher com:
>      "Complementa (não substitui) as regras em [<path>](<path>) — leia também."
>      Se não existe, remover este bloco. -->

---

## 1. Missão

{{MISSAO}}

**Data-alvo**: {{DATA_ALVO}}
**Hard-launch produção**: {{DATA_HARD_LAUNCH}}

---

## 2. Sistema de referência

**Nome**: {{REFERENCIA_NOME}}
**Tipo**: {{REFERENCIA_TIPO}}
**Acesso**: {{REFERENCIA_ACESSO}}

**Restrições**:
{{REFERENCIA_RESTRICOES}}

---

## 3. Comandos de build e teste

| Comando | Propósito |
|---|---|
| `{{CHECK_CMD}}` | Verificação estática (typecheck / lint / compile) |
| `{{TEST_CMD}}` | Suite de testes |
| {{OUTROS_CMDS}} | {{OUTROS_CMDS_DESC}} |

> Ambos devem estar verdes antes de qualquer finding ser considerado resolvido.

---

## 4. Pessoas e papéis

{{PESSOAS_TABELA}}

**Autoridade final de escopo**: {{AUTORIDADE_ESCOPO}}

---

## 5. Repositórios

{{REPOSITORIOS_TABELA}}

<!-- Formato:
| Repo | Path local | Função |
|---|---|---|
| sistema-novo | /home/... | Este projeto |
| sistema-referencia | /home/... | Fonte de verdade |
-->

**Para outros devs**: se os paths variarem por máquina, usar `find ~ -maxdepth 5 -type d -name <repo>` para localizar.

---

## 6. Áreas do projeto

Módulos/telas/etapas que podem aparecer em findings. Usados pelo `/audit-status` para agrupar e pelo `/audit-new` para categorizar.

{{AREAS_PROJETO}}

<!-- Lista, uma por linha. Exemplos:
- login
- dashboard
- formulário de pedido
- exportar-excel
- outro
-->

---

## 7. Ambientes e URLs

{{AMBIENTES_TABELA}}

<!-- Formato:
| Ambiente | Sistema novo | Sistema de referência | Observações |
|---|---|---|---|
| local | http://localhost:3000 | /path/local ou URL | VPN? |
| staging | https://... | https://... | — |
| prod | https://... | https://... | — |
-->

**Regras operacionais**:
{{AMBIENTES_NOTAS}}

---

## 8. Bancos de dados

{{BANCOS_TABELA}}

<!-- Formato:
| Banco | Host/Nome | Papel | Status |
|---|---|---|---|
| principal | host/db | prod/dev/homolog | ativo |
| antigo | host/db | dev congelado | ⛔ não usar |
-->

**Onde estão as credenciais** (ponteiros — nunca valores):
{{CREDENCIAIS_PONTEIROS}}

> 🔒 NUNCA escrever credenciais reais neste arquivo.

---

## 9. MCPs disponíveis

{{MCPS_TABELA}}

<!-- Formato:
| MCP | Função | Observações |
|---|---|---|
| playwright | automação de browser | login manual se OAuth |
| mssql | queries read-only | confirmar banco-alvo antes de usar |
-->

**Notas operacionais**:
{{MCPS_NOTAS}}

---

## 10. Casos de referência (gabarito de validação)

{{CASOS_REFERENCIA_TABELA}}

<!-- Formato:
| ID (novo) | ID (referência) | Por que serve | Áreas cobertas |
|---|---|---|---|
| 12345 | 67890 | Cenário com X e Y | login, formulário |
-->

> Toda validação de paridade DEVE usar um destes casos (ou justificar por que criou um novo).

---

## 11. Integração com Notion (QA Board)

**Status**: {{NOTION_STATUS}}

<!-- UMA das opções:
     - Ativado — databases configurados abaixo
     - Desativado — /audit-qa não operacional neste projeto
-->

{{NOTION_URLS_TABELA}}

<!-- Quando ativado:
| Database | URL | Database ID |
|---|---|---|
| PDD - Findings | https://notion.so/... | <id> |
| PDD - Testes QA | https://notion.so/... | <id> |
-->

**Estrutura esperada** (fixa — não alterar):

```
PDD - Findings
├── Nome (title)                   título humano do finding
└── Audit (select)                 ID técnico (001-<slug>, 002-<slug>, ...)

PDD - Testes QA
├── Teste (title)                  descrição do caso de teste
├── Finding (relation → Findings)  liga o teste à página do finding pai
└── Status do Teste (select)       "Aguardando teste" | "Aprovado" | "Rejeitado"
```

> 🔒 Essas URLs são a fonte de verdade pra `/audit-qa`. Sem elas, a skill não opera.

---

## 12. Regras invioláveis

**Do PDD (sempre, em todo projeto)**:
- Push feito APENAS pelo humano. Claude NUNCA executa `git push`, nem com `--force`.
- Claude NUNCA commita autonomamente — apenas sugere o comando.

**Do projeto**:
{{REGRAS_PROJETO}}

<!-- Lista as regras extraídas do documento de regras (se existir) + regras específicas deste ciclo -->

---

## 13. Fluxo PDD (não mexer — parte fixa do modelo)

### Ciclo de vida de um finding

```
1. Dev acha uma divergência observando o sistema.
2. Dev roda /audit-new <descricao>
   → entrevista estruturada (2 vias: dev + Claude)
   → gera .audit/findings/NNN-<slug>/README.md + refs/
   → atualiza board.md
3. Dev ou Claude roda /audit-investigate NNN
   → decisão A/B/C/D de abordagem (estática, dinâmica, visual, combinada)
   → investigação executada
   → gera investigation.md no mesmo finding
4. Dev ou Claude roda /audit-resolve NNN
   → implementa fix (respeitando as regras do projeto)
   → valida: {{CHECK_CMD}} + {{TEST_CMD}} + validação contra sistema de referência
   → gera resolution.md
   → move pasta de findings/ para resolved/
   → sugere comando de commit (NUNCA auto-commita, NUNCA auto-pusha)
5. Dev revisa, commita, abre PR, mergeia manualmente.
6. Dev roda /audit-qa NNN (só depois do PR mergeado):
   → cria página do finding + N cards de teste no Notion
   → cards em linguagem leiga pro QA testar
   → mesma skill depois (reexecução) mostra status do QA e trata reprovações
```

### Regras de qualidade de cada finding

- **Sintoma obrigatório**: um fato observável (número, texto de erro, screenshot). Nunca "tá errado".
- **Reprodução obrigatória**: passos que outro dev (ou outra sessão Claude) consegue seguir sozinho.
- **Evidência do sistema de referência obrigatória**: screenshot, query result ou output do comportamento esperado.
- **Critério de aceitação obrigatório**: condição binária testável (passa/não passa).

### Onde ficam os arquivos

```
.audit/
├── BOOTSTRAP.md              ← este arquivo
├── board.md                  ← kanban leve
├── findings/NNN-<slug>/      ← findings abertos
│   ├── README.md
│   ├── investigation.md (se /audit-investigate rodou)
│   ├── resolution.md (se /audit-resolve rodou)
│   └── refs/                 ← screenshots, exports, evidências
└── resolved/NNN-<slug>/      ← findings resolvidos (pasta move inteira)
```

### Como começar uma nova sessão Claude neste projeto

```
1. Leia .audit/BOOTSTRAP.md (este arquivo)
2. Leia .audit/board.md para saber o estado atual
3. Se existir documento de regras do projeto, leia também
4. Leia o finding específico em .audit/findings/NNN-<slug>/ se estiver trabalhando em um
5. Comece o trabalho
```

---

**Bootstrap gerado em**: {{DATA_GERACAO}}
**Gerado por**: `/audit-bootstrap` (skill do método PDD)
