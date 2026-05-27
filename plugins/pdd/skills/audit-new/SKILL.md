---
name: "audit-new"
description: "Captura um novo finding (divergência, bug ou comportamento incorreto vs sistema de referência) via entrevista estruturada mão-dupla dev + Claude. Gera .audit/findings/NNN-<slug>/README.md com disciplina forçada — respostas vagas não são aceitas."
argument-hint: "(opcional) descrição curta do problema; se vazio, começa entrevista do zero"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Este comando faz parte do método **PDD (Parity-Driven Development)**. Um *finding* é a unidade de trabalho do PDD — uma divergência observada entre o sistema novo e o sistema de referência.

**Princípio central**: a entrevista é **mão dupla**. Dev descreve, Claude tenta reproduzir (ou observa você reproduzindo), ambos conversam antes de gravar o finding.

## Outline

### 1. Verificações iniciais (bloqueantes)

- Leia `.audit/BOOTSTRAP.md`. Se NÃO existir, pare e instrua: "Rode `/audit-bootstrap` primeiro — o contexto operacional é obrigatório antes de registrar qualquer finding."
- Do BOOTSTRAP, extraia:
  - `REFERENCIA_NOME` — nome do sistema de referência (gabarito)
  - `AREAS_PROJETO` — lista de áreas/módulos/etapas para a Q1
  - `CHECK_CMD` e `TEST_CMD` — para os critérios de aceitação
  - `CASOS_REFERENCIA` — para sugerir gabarito na Q7
- Se existe documento de regras do projeto (referenciado no BOOTSTRAP), leia para conhecer restrições.
- Liste `.audit/findings/` e `.audit/resolved/` para determinar o próximo ID:
  - Pegue o maior `NNN` entre todos os subdirs de ambas as pastas.
  - Próximo ID = maior + 1, com padding de 3 dígitos (`001`, `002`, ..., `999`).
- Carregue o template em `.claude/skills/audit-new/template.md`.

### 2. Tom da entrevista — regras de ouro

**SINTOMAS VAGOS SÃO REJEITADOS.** Se o dev responder "tá errado", "não funciona", "ficou ruim", "tá quebrado", você DEVE responder:

> Preciso de um fato observável. Por exemplo:
> - **Errado**: "a tela de pedidos tá errada"
> - **Certo**: "a tela de pedidos do novo sistema mostra 3 itens; o mesmo pedido no sistema de referência mostra 5"
>
> O que você vê concretamente?

Não prossiga até ter resposta concreta.

**NUNCA invente dados.** Se o dev disser "não sei", registre literalmente "não sei" e siga. Não chute hipóteses no lugar dele.

**UMA SEÇÃO POR VEZ.** Não encadeie todas as perguntas. Termine cada seção com "ok, anotei. vamos pra próxima?" e espere confirmação.

### 3. Blocos da entrevista

#### Bloco 1 — Identificação (3 perguntas)

**Q1 — Área afetada?**
Liste as áreas do BOOTSTRAP (`AREAS_PROJETO`) como opções numeradas. Se vazio ou pendente no BOOTSTRAP, apresente: "Descreve em 1 frase qual área/tela/módulo está afetado."

Se a área não estiver na lista, aceite a descrição livre e registre — não force um enquadramento.

**Q2 — Módulo provável?**
Baseado na área escolhida, SUGIRA arquivos do sistema novo + correspondentes no sistema de referência. Exemplo:
```
Sistema novo: src/features/orders/orders.service.ts
              src/features/orders/orders.controller.ts
Sistema de referência: app/models/Order.py (ou equivalente)
Confirma ou corrige?
```

Se o dev não souber, registre "não identificado" e siga.

**Q3 — Severidade?**
Opções:
- `critica` — bloqueia o fluxo principal / impede o uso
- `alta` — funciona mas resultado incorreto (valor errado, dado faltando)
- `media` — edge case visível, uso prejudicado
- `baixa` — cosmético ou impacto mínimo

Ao fim do Bloco 1: "Identificado. Agora vamos ao sintoma."

#### Bloco 2 — Sintoma (2 perguntas, obrigatórias e concretas)

**Q4 — O que você VÊ acontecer no sistema novo (um fato)?**
Exija descrição observável. Valor numérico, texto na tela, código de erro, comportamento de UI.

**Q5 — O que DEVERIA acontecer (o que o sistema de referência faz)?**
Mesma exigência — um fato. Se o dev diz "não sei o que o sistema de referência faz", PAUSAR a entrevista e sugerir:

> Sem saber o comportamento do sistema de referência, não temos gabarito. Vamos verificar?
> - Opção A: eu abro o sistema de referência (se houver MCP de browser disponível) e mostro pra você
> - Opção B: você abre o sistema de referência e me descreve o que vê
> - Opção C: pausamos este finding; você volta quando souber

Ao fim do Bloco 2: "Sintoma claro. Próximo: reprodução."

#### Bloco 3 — Reprodução (2 perguntas)

**Q6 — Passos para reproduzir (lista numerada)**
Peça passos a partir de "abrir a URL X com usuário Y e caso Z". Se dev der passos com lacunas, peça mais detalhe.

**Q7 — Qual caso de referência vamos usar?**
Sugira os casos do BOOTSTRAP (`CASOS_REFERENCIA`). Se nenhum for aplicável, pergunte qual usar e registre.

Ao fim do Bloco 3: "Reprodução mapeada. Agora a decisão mais importante."

#### Bloco 4 — DECISÃO: reprodução em mão dupla (⭐ central do PDD)

Apresente as 3 opções textualmente:

```
Tenho 3 caminhos pra documentar melhor este finding. Qual cabe aqui?

A) Eu reproduzo agora (via MCP de browser, se disponível).
   Bom quando: bug é visual/UI simples, acesso ao sistema de referência
   é viável sem MFA, você quer me ver trabalhando.
   Tempo: ~5 min meu + seu acompanhando.

B) VOCÊ reproduz na sua tela agora, dropa screenshots/exports em
   .audit/findings/NNN-<slug>/refs/, eu leio e adiciono observações.
   Bom quando: login complexo, MFA, rede interna, ou bug depende de
   estado que só você consegue montar.
   Tempo: seu (variável).

C) Pulamos reprodução agora — o que você já documentou é suficiente
   pra investigar. Deixamos reprodução pra /audit-investigate.
   Bom quando: finding é óbvio e o próximo passo é análise de código.

Qual?
```

Execute o caminho escolhido:

**Se A**: use MCP de browser para abrir sistema de referência + sistema novo, reproduza os passos, capture o que observou. Descreva em 3-5 linhas. Pergunte: "Isso bate com o que VC viu? Algo diferente?" Se houver discrepância, refine.

**Se B**: crie antecipadamente o diretório `.audit/findings/NNN-<slug>/refs/` e informe o path. Espere dev confirmar que dropou os arquivos. Liste o conteúdo, leia imagens/textos, descreva o que observou. Peça confirmação.

**Se C**: registre `Reprodução pulada na criação; delegar ao /audit-investigate`.

#### Bloco 5 — Hipótese (1 pergunta, opcional)

**Q8 — Você tem alguma hipótese de causa?**
Aceitar "não sei" sem cobrar.

### 4. Resumo e confirmação (antes de escrever)

Mostre um resumo estruturado de TODAS as respostas (use os mesmos nomes de campos do template). Pergunte:

> Posso criar o finding `NNN-<slug>` com esses dados?
> - **sim**: eu escrevo e encerro
> - **editar X**: volta ao bloco X
> - **cancelar**: aborta sem escrever

### 5. Geração do slug

A partir do sintoma (Q4), gere um slug curto (3-5 palavras) em formato kebab-case.

**Exemplos bons**: `001-checkout-valor-incorreto`, `002-lista-vazia-filtro`, `003-export-csv-colunas-faltando`.
**Exemplos ruins**: `001-bug`, `002-problema`, `003-nao-funciona`.

### 6. Escrita dos arquivos

- Crie `.audit/findings/NNN-<slug>/README.md` usando o template, substituindo placeholders.
- Crie `.audit/findings/NNN-<slug>/refs/` (se ainda não existe).
- Se reprodução foi feita (Blocos A ou B), inclua seção `## Observações durante a reprodução` com o que você registrou.
- Atualize `.audit/board.md`:
  - Adicione em "Disponíveis": `- [ ] NNN-<slug> — <sintoma resumido em 1 linha> (severidade: X)`

### 7. Encerramento

Reporte:
- Path do README.md criado
- Path da pasta refs/ criada
- Severidade
- Próximo passo:
  - Se severidade=critica: "Rode `/audit-investigate NNN` agora — é crítico."
  - Caso contrário: "Quando alguém pegar, rode `/audit-investigate NNN`."

## Regras de qualidade

- NUNCA aceitar sintoma vago. Forçar fato observável.
- NUNCA prosseguir sem comportamento esperado do sistema de referência (ou plano explícito pra descobrir).
- NUNCA inventar hipótese quando dev responde "não sei".
- SEMPRE pedir confirmação do resumo antes de escrever em disco.
- SEMPRE criar `refs/` mesmo que vazio — dev precisa saber onde dropar evidências.
