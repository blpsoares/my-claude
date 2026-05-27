---
name: "audit-bootstrap"
description: "Entrevista estruturada para preencher .audit/BOOTSTRAP.md — contexto operacional lido por TODA nova sessão Claude antes de qualquer trabalho de auditoria PDD. Roda uma vez no setup do projeto."
argument-hint: "(opcional) 'refazer' para sobrescrever bootstrap existente"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Este comando inicializa o método **PDD (Parity-Driven Development)** — metodologia para auditar paridade entre um sistema novo (refactor, rewrite, port) e um sistema de referência (legado, spec original, sistema anterior). O artefato gerado (`.audit/BOOTSTRAP.md`) é referenciado por todos os outros comandos `/audit-*`.

## Outline

### 1. Verificações iniciais

- Verifique se `.audit/` existe. Se NÃO existir, rode `mkdir -p .audit/findings .audit/resolved && touch .audit/findings/.gitkeep .audit/resolved/.gitkeep` e continue.
- Verifique se `.audit/BOOTSTRAP.md` já existe:
  - Se existe E `$ARGUMENTS` NÃO contém "refazer": pare e informe "BOOTSTRAP.md já existe. Para sobrescrever, rode `/audit-bootstrap refazer`. Para apenas visualizar, leia o arquivo direto."
  - Se existe E `$ARGUMENTS` contém "refazer": leia o conteúdo existente como "respostas anteriores" e use como default em cada pergunta.
- Verifique se existe algum arquivo de regras do projeto (ex: `.specify/memory/constitution.md`, `RULES.md`, `CONTRIBUTING.md`, `ARCHITECTURE.md`). Se encontrar, anote o path — será referenciado em vez de duplicado.
- Carregue o template em `.claude/skills/audit-bootstrap/template.md`.

### 2. Tom da entrevista

- **Regra crítica**: NUNCA preencher campo sem resposta explícita do dev. Se a resposta for vaga ou "não sei", registrar literalmente `<pendente — preencher antes de qualquer /audit-new>`.
- Apresente uma seção por vez. Não faça sequência interminável de perguntas — cada seção termina com "ok, anotei. próxima seção?" e espera confirmação.
- Quando um documento de regras do projeto já cobre um tópico, pergunte APENAS se há algo além do que lá está. Não repita a informação.

### 3. Seções da entrevista (nesta ordem)

**Seção 1 — Missão e escopo**
- Em UMA frase, qual é a missão deste projeto? (ex: "reimplementar X em Y mantendo fidelidade comportamental")
- Qual a data-alvo principal? (formato ISO YYYY-MM-DD ou "sem data fixa")
- Existe hard-launch em produção posterior a essa data? Quando?

**Seção 2 — Sistema de referência**

Esta é a seção mais importante — define o "gabarito" que todo finding vai usar.

- Qual é o **nome** do sistema de referência? (ex: "sistema legado PHP", "API v1", "spec de negócio")
- Qual é o **tipo**? (ex: aplicação PHP, serviço externo, planilha, spec/documento, outro sistema rodando)
- Como acessar: existe URL? path local? necessita VPN? login especial? MCP disponível?
- Há restrições de escrita? (ex: "nunca modificar o banco compartilhado", "só leitura via MCP")
- O sistema de referência usa banco de dados? Qual? Acesso como?

**Seção 3 — Comandos de build e teste**

- Qual o comando para checar o projeto (typecheck, lint, compile)? (ex: `bun run check`, `npm run typecheck`)
  → Registrar como `CHECK_CMD`
- Qual o comando para rodar testes? (ex: `bun run test`, `npm test`, `pytest`)
  → Registrar como `TEST_CMD`
- Esses comandos precisam estar verdes antes de qualquer fix ser considerado válido?
- Há comandos de integração específicos que também devem ser rodados? Registrar separadamente.

**Seção 4 — Pessoas e papéis**
- Quem são os humanos envolvidos? Para cada um: nome, papel (dev/QA/PO/etc), área de foco
- Quantas instâncias Claude vão colaborar? Cada uma pareada com qual humano?
- Quem tem autoridade final pra decidir escopo?

**Seção 5 — Repositórios e paths locais**
- Liste os repositórios relevantes: nome, path local (ou como clonar), função em 1 frase.
- O repositório do sistema de referência já está clonado localmente? Qual o path?
- Se outros devs participarem: como eles localizam os repos? (padronizar `find ~ -maxdepth 5 -type d -name <repo>` se caminhos variam)

**Seção 6 — Áreas do projeto**

Defina os módulos, telas, etapas ou áreas do sistema novo que podem aparecer em findings. Esta lista será usada pelo `/audit-status` para agrupar findings e pelo `/audit-new` para categorizar.

- Liste as áreas do projeto (ex: "login", "dashboard", "exportar", "formulário de pedido"). Formato livre — o dev define.
- Se o projeto é um wizard ou tem etapas numeradas, liste as etapas.
- Incluir "outro" como área catch-all automática (não precisa listar).

Registrar como `AREAS_PROJETO` — lista separada por quebra de linha.

**Seção 7 — Ambientes e URLs**
- Para CADA ambiente relevante (local, staging, produção), colete: URL do sistema novo, URL/acesso do sistema de referência.
- Algum ambiente requer VPN, login especial ou configuração manual? Documentar.
- Em desenvolvimento local: como rodar? porta padrão?

**Seção 8 — Bancos de dados**
- Para cada banco: endereço/host, nome, papel (prod/dev/homolog/congelado).
- 🔒 Credenciais: NÃO perguntar as credenciais em si. Perguntar ONDE elas estão (arquivo `.env`, secret manager, vault). Gravar só o ponteiro.
- Há bancos que NÃO devem ser usados (desatualizados, congelados)? Registrar com ⛔.

**Seção 9 — MCPs disponíveis**
- Liste os MCPs que esta sessão tem acesso. Para cada um: nome e função em 1 frase.
- Atenção especial a: MCP de banco (qual banco por padrão?), MCP de browser (funciona headless? precisa de credencial?).

**Seção 10 — Casos de referência (gabarito de validação)**

"Casos de referência" são artefatos concretos do sistema que servirão de gabarito em findings — podem ser pedidos, contratos, faturas, registros, IDs, usuários de teste, etc. O nome do artefato depende do domínio do projeto.

- Peça AO MENOS 2-3 casos de referência. Para cada: identificador no sistema novo, identificador equivalente no sistema de referência, por que serve de gabarito (ex: "tem cenário X e Y"), áreas do projeto cobertas.
- Se o projeto ainda não tem casos concretos, registrar `<a definir no primeiro /audit-new>` e avisar que cada finding precisará eleger o seu.

**Seção 11 — Integração com Notion (QA Board)**

Primeiro pergunte: **"Vai usar Notion pra gerenciar o QA desse projeto?"** (sim/não).

- Se **NÃO**: registre "Desativado — `/audit-qa` não estará operacional neste projeto." e pule pra Seção 12.

- Se **SIM**, pergunte se os 2 databases já existem:

```
O PDD precisa de 2 databases no Notion (estrutura fixa):

  1. "PDD - Findings"   — 1 página por finding resolvido
     Colunas:
       • Nome (title)        — título humano legível
       • Audit (select)      — ID técnico (ex: 001-<slug>)

  2. "PDD - Testes QA"  — N páginas por finding (1 por caso de teste)
     Colunas:
       • Teste (title)                 — descrição do caso de teste
       • Finding (relation → DB1)      — liga o teste ao finding pai
       • Status do Teste (select)      — Aguardando teste | Aprovado | Rejeitado

Você prefere:
  (a) Colar as URLs dos 2 databases que você já criou
  (b) Eu criar os 2 databases pra você agora, com essa estrutura exata
```

- Se **(a)**: pergunte as 2 URLs separadamente. Valide que são URLs de database do Notion. Registre ambas.

- Se **(b)**:
  1. Pergunte: "Cola a URL da página-pai onde eu devo criar os 2 databases"
  2. Verifique se o MCP Notion está disponível. Se NÃO: avise o dev e pause.
  3. Se disponível: crie DB1 ("PDD - Findings"), capture ID e URL. Crie DB2 ("PDD - Testes QA") com relation apontando pro DB1. Reporte ao dev e peça confirmação antes de gravar.

- **Gravar no BOOTSTRAP.md** (Seção 11 do template) ambas as URLs e IDs. Essencial — `/audit-qa` lê daqui.

**Seção 12 — Regras invioláveis**

- Se existe documento de regras do projeto (encontrado na verificação inicial): extraia automaticamente as regras mais relevantes para o PDD (push por humano, testes obrigatórios, padrões de código, restrições de banco, etc.). Pergunte: "tem algo além do que está lá, específico pra este ciclo?"
- Se não existe documento: pergunte as regras hard do projeto (ex: "nunca commitar sem PR", "banco de referência é read-only", "toda função nova tem teste").
- Registrar explicitamente: **push feito apenas pelo humano** é regra inviolável do PDD independente do projeto.

**Seção 13 — Fluxo de trabalho PDD**
- Não precisa perguntar nada — apenas copiar do template. É parte fixa do modelo.

### 4. Revisão e geração

- Ao final das seções, mostre um resumo condensado de CADA seção (2-3 linhas) e pergunte: "Posso gerar o BOOTSTRAP.md com essas respostas? (sim/editar Seção X)".
- Se o dev responder "editar Seção X", volte àquela seção.
- Ao confirmar, gere o arquivo `.audit/BOOTSTRAP.md` usando o template, substituindo os placeholders. Campos sem resposta preenchidos com `<pendente — preencher antes de qualquer /audit-new>`.
- Crie também `.audit/board.md` esqueleto se não existir.

### 5. Board inicial (criar se não existir)

```markdown
# PDD Board — <nome do projeto>

> Atualize ANTES de pegar um finding (marque [doing]) e DEPOIS de resolver (mova pra resolvidos/).
> Contexto: ver [BOOTSTRAP.md](./BOOTSTRAP.md)

## Em andamento
<vazio>

## Disponíveis
<vazio>

## Resolvidos (últimos 7 dias)
<vazio>
```

### 6. Encerramento

Reporte:
- Path do BOOTSTRAP.md gerado
- Quantos campos ficaram `<pendente>` (se houver, avisar que precisa preencher antes de `/audit-new`)
- Próximo passo: "Quando achar uma divergência, rode `/audit-new <descrição curta>`."

## Regras de qualidade

- NUNCA inventar resposta por inferência. Se dev disser "não sei", registrar literalmente.
- NUNCA escrever credenciais reais em arquivo (só ponteiros — onde elas vivem).
- SEMPRE referenciar documento de regras existente em vez de duplicar.
- Confirmação final antes de escrever em disco.
