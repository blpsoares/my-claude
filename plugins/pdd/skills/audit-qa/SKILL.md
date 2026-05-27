---
name: "audit-qa"
description: "Ponte entre findings resolvidos (git) e QA (Notion). Roda depois do PR do finding ser mergeado. Na primeira execução: cria página do finding + N cards de teste em linguagem leiga. Nas execuções seguintes: lê status do QA, trata aprovações e reprovações. NUNCA roda antes do PR mergear."
argument-hint: "ID do finding (ex: 007 ou 007-checkout-valor-incorreto)"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Parte do método **PDD (Parity-Driven Development)**. Esta skill faz a ponte entre o mundo do dev (git, `.audit/resolved/`) e o mundo do QA (Notion).

**Regra de ouro**: só roda depois do PR do finding estar mergeado. Antes disso, o fix não está no código que QA vai testar.

**Comportamento sensível ao estado**: uma única skill que age conforme o que encontra — cria cards na primeira execução, mostra status nas seguintes.

## Outline

### 1. Verificações iniciais (bloqueantes)

- Leia `.audit/BOOTSTRAP.md`. Se NÃO existir: pare e instrua `/audit-bootstrap` primeiro.
- Na Seção 11 do BOOTSTRAP, verifique o status da integração Notion:
  - Se "Desativado": pare e reporte: "Notion não está ativado neste projeto. Rode `/audit-bootstrap refazer` pra ativar, ou use QA sem Notion."
  - Se "Ativado": capture as 2 URLs/IDs dos databases (`PDD - Findings` e `PDD - Testes QA`).
- Verifique se o MCP do Notion está disponível (`mcp__claude_ai_Notion__*` ou equivalente). Se NÃO:
  > "O MCP do Notion não está conectado nesta sessão. Conecta o MCP e tenta de novo."
  — e pause.
- Parse `$ARGUMENTS`:
  - Se vazio: pergunte qual finding; liste os em `.audit/resolved/` como opções.
  - Se `NNN`: encontre pasta em `.audit/resolved/NNN-*`.
  - Se `NNN-<slug>`: busca direta.
  - Se não encontrou em `resolved/`: verifique se ainda está em `findings/`. Se sim: pare e reporte:
    > "Finding {{NNN}} ainda não foi resolvido. Rode `/audit-resolve {{NNN}}` primeiro."
- Leia `README.md`, `investigation.md` (se existe) e `resolution.md` do finding.

### 2. Validação do PR mergeado

1. Rode: `gh pr list --state merged --search "{{NNN}}"` (ou `--json number,title,mergedAt,url`)
2. Filtre PRs que mencionem o ID no título.
3. Se NÃO encontrou PR mergeado:
   - Reporte: "Não achei PR mergeado referenciando este finding."
   - Pergunte:
     > "(a) ainda não abriu o PR, (b) o PR não segue a convenção de título, ou (c) quer forçar mesmo assim?"
   - Se (a): instrua commitar e abrir PR.
   - Se (b): peça URL do PR e valide via `gh pr view <url> --json state,mergedAt`.
   - Se (c): registre na página do Notion que o PR não foi validado automaticamente.
4. Se encontrou: registre a URL do PR para incluir na página do finding.

### 3. Estado atual no Notion

Consulte o database "PDD - Findings":
- Busque página com coluna `Audit` = `{{NNN}}-<slug>`.

**Caso A — Finding ainda não existe no Notion** → modo CRIAR (Seção 4).
**Caso B — Finding já existe** → modo STATUS/FEEDBACK (Seção 5).

### 4. Modo CRIAR (primeira execução após PR mergeado)

#### 4.1 Criar página do Finding (database "PDD - Findings")

Carregue o template em `.claude/skills/audit-qa/template-finding-page.md`.

**Mapear dados técnicos pra linguagem leiga**:
Leia `resolution.md`. Tente deduzir:
- Qual **tela/área** o usuário vê afetada
- Qual **interação** (botão, campo, ação) dispara o comportamento
- Qual **dado** muda na tela

Se a dedução for ambígua, **pergunte ao dev**:
> "Não consegui mapear esse fix pra uma interação de tela de forma clara. Me diz em 1-2 frases: o que o QA vai VER de diferente depois desse fix?"

**Não invente termos técnicos**. Evite: função, query, endpoint, repositório, controller, state. Use: "tela", "botão", "campo", "valor", "lista", "formulário".

Crie a página em "PDD - Findings" via MCP com:
- `Nome` (title): título humano legível
- `Audit` (select): `{{NNN}}-<slug>` — exatamente o ID do finding
- Conteúdo do corpo: template preenchido

Capture o `page_id` retornado.

#### 4.2 Criar N cards de teste (database "PDD - Testes QA")

Do `README.md` original (seção "Critério de aceitação") extraia os critérios. Cada critério vira 1 card.

Se não há critérios estruturados:
> "O finding não tem critérios de aceitação listados. Quais cenários você quer expor pro QA?"

Carregue o template em `.claude/skills/audit-qa/template-test-card.md`.

Para cada cenário, crie página em "PDD - Testes QA" com:
- `Teste` (title): descrição clara do que testar em linguagem leiga
- `Finding` (relation): apontando pro `page_id` de 4.1
- `Status do Teste` (select): `Aguardando teste`
- Conteúdo do corpo: template preenchido em linguagem leiga

Capture `page_id` e URL de cada card.

#### 4.3 Back-reference no git

Adicione (ou atualize) seção no final de `.audit/resolved/{{NNN}}-<slug>/resolution.md`:

```markdown
## 8. Cards de QA no Notion

Criados em {{DATA}}:

- **Página do Finding**: [Notion]({{URL_PAGINA_FINDING}})
- **Cards de teste**:
  - [{{TITULO_CARD_1}}]({{URL_CARD_1}})
  - ...
```

#### 4.4 Relatório ao dev

Reporte:
- Quantos cards foram criados
- URL da página do finding e dos cards
- "QA pode começar a testar agora. Rode `/audit-qa {{NNN}}` de novo quando quiser ver o status."

### 5. Modo STATUS/FEEDBACK (execuções subsequentes)

#### 5.1 Buscar cards vinculados

Via MCP Notion, no database "PDD - Testes QA", busque páginas onde `Finding` aponta pro finding atual.

#### 5.2 Classificar o estado geral

- **Todos "Aguardando teste"**: QA ainda não começou.
  > "QA ainda não testou nenhum dos {{N}} cenários. Nada pra fazer agora."

- **Todos "Aprovado"**: finding validado por QA.
  > "🎉 QA aprovou todos os {{N}} cenários. Finding {{NNN}} fechado pelo QA."
  Ofereça adicionar `qa-status: aprovado` no frontmatter do `README.md`.

- **Misto (alguns aprovado + alguns aguardando)**: reporte progresso parcial. Aguarde.

- **Pelo menos um "Rejeitado"**: entre em modo FEEDBACK (5.3).

#### 5.3 Modo FEEDBACK (há cards "Rejeitado")

Pra cada card rejeitado, leia título, passos e comentários (`mcp__claude_ai_Notion__notion-get-comments`).

Apresente ao dev:

```
❌ QA reprovou {{M}} de {{N}} cenários do finding {{NNN}}.

═══════════════════════════════════════════════════════════
Card 1: "{{TITULO}}"
URL: {{URL}}

QA disse (comentários):
  {{COMENTARIO_1}}

Minha análise inicial:
  {{HIPOTESE_IA}}

═══════════════════════════════════════════════════════════
```

Depois pergunte:

```
O que fazemos?

(a) Abrir um novo finding consolidando todas as regressões
(b) Abrir um finding por cenário rejeitado separadamente
(c) Discutir primeiro — você quer que eu investigue antes de abrir finding?
(d) Descartar — QA entendeu errado ou estava em ambiente errado
(e) Nada agora — vou conversar com QA antes
```

Se (a) ou (b): gere rascunho de `/audit-new` com os comentários do QA como contexto, **sem criar o arquivo** — o dev aprova antes.

#### 5.4 Comentários no Notion (com discrição)

Após ação no modo feedback, se new finding foi aberto, pode adicionar comentário no card rejeitado:

> "Registrado como finding {{ID}} em {{DATA}}. Será re-testado após próxima correção."

**Não deixe comentários redundantes** — só informação operacional relevante pro QA.

### 6. Atualização do board PDD

Se criou cards (Seção 4), atualize `.audit/board.md`:

```
- [x] NNN-<slug> — <resumo> (enviado pra QA em YYYY-MM-DD, {{N}} cenários)
```

Se processou feedback e resultou em novo finding, adicione referência ao novo ID.

## Regras de qualidade

- **NUNCA crie cards antes do PR mergeado.** Guarda crítica — se não achou PR mergeado, pare ou peça confirmação.
- **NUNCA use jargão técnico nos cards.** QA não é dev. Converta termos técnicos em ações de tela.
- **SEMPRE grave URLs do Notion no `resolution.md`** — rastreabilidade bi-direcional.
- **SEMPRE pergunte ao dev** se não conseguiu deduzir a interação de tela. Não invente.
- **NUNCA altere status de cards no Notion** por conta própria. Só com pedido explícito do dev (ex: opção d do feedback).
- Se o MCP Notion falhar no meio, PARE e reporte estado parcial — não tente rollback automático.
