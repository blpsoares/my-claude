---
name: "audit-status"
description: "Dashboard read-only do estado do PDD no projeto. Mostra findings abertos, em investigação e resolvidos, agrupados por área e severidade. Útil no início de uma sessão pra saber onde pegar."
argument-hint: "(opcional) 'detalhado' para listar cada finding; 'area:<nome>' para filtrar por área; 'severidade:<nivel>' para filtrar por severidade"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Parte do método **PDD (Parity-Driven Development)**. Esta skill é **read-only** — não altera nenhum arquivo. Objetivo: saber o estado geral do projeto sem ter que abrir dezenas de arquivos.

## Outline

### 1. Verificações iniciais

- Verifique se `.audit/BOOTSTRAP.md` existe. Se NÃO: reporte "PDD ainda não inicializado neste projeto. Rode `/audit-bootstrap` primeiro." e pare.
- Leia `.audit/BOOTSTRAP.md` brevemente para extrair: Missão, Data-alvo, `AREAS_PROJETO`.
- Leia `.audit/board.md` se existir.
- Liste `.audit/findings/` (todos os subdirs) — findings abertos.
- Liste `.audit/resolved/` (todos os subdirs) — findings resolvidos.

### 2. Parse dos findings

Para cada subdir em `findings/` e `resolved/`:
- Leia o YAML frontmatter do `README.md` (id, titulo, slug, area, severidade, status, data-descoberta, descoberto-por)
- Detecte se `investigation.md` existe (flag investigado)
- Detecte se `resolution.md` existe (flag resolvido)
- Liste conteúdo de `refs/` (quantidade de evidências)

### 3. Aplicar filtros (se houver `$ARGUMENTS`)

- Sem args: visão geral padrão.
- Com `detalhado`: inclui listagem individual de cada finding.
- Com `area:<nome>`: filtra por área (ex: `area:checkout`).
- Com `severidade:<nivel>`: filtra por severidade (ex: `severidade:critica`).
- Com `aberto` ou `investigado`: filtra por status.

### 4. Output — formato padrão

```
═══════════════════════════════════════════════════════════════
  PDD Status · <nome do projeto>
═══════════════════════════════════════════════════════════════

Missão: <primeira linha da Missão do BOOTSTRAP>
Data-alvo: <data do BOOTSTRAP> · <X dias restantes ou "sem data fixa">

───────────────────────────────────────────────────────────────
  Resumo
───────────────────────────────────────────────────────────────

Findings abertos:        <N>   (arquivos em .audit/findings/)
  └─ já investigados:    <M>
  └─ aguardando:         <N-M>

Resolvidos:              <R>   (últimos 7 dias: <R7>)

───────────────────────────────────────────────────────────────
  Por área do projeto
───────────────────────────────────────────────────────────────

<Para cada área em AREAS_PROJETO do BOOTSTRAP:>
<nome da área>:  <n abertos> / <m resolvidos>

Outro (não categorizado):  <n> / <m>

───────────────────────────────────────────────────────────────
  Por severidade
───────────────────────────────────────────────────────────────

🔴 critica:  <n>
🟠 alta:     <n>
🟡 media:    <n>
🟢 baixa:    <n>

───────────────────────────────────────────────────────────────
  Em andamento (do board.md)
───────────────────────────────────────────────────────────────

<conteúdo da seção "Em andamento" do board.md>

───────────────────────────────────────────────────────────────
  Próximos passos sugeridos
───────────────────────────────────────────────────────────────

<lógica condicional>:
- Se há findings críticos abertos: "🚨 Priorize NNN (critica) com /audit-investigate NNN"
- Se há findings já investigados aguardando fix: "Há M findings prontos pra /audit-resolve"
- Se não há findings abertos: "Nenhum finding aberto. Tudo em dia."
```

### 5. Output com `detalhado`

Adiciona ao final:

```
───────────────────────────────────────────────────────────────
  Findings abertos (detalhado)
───────────────────────────────────────────────────────────────

[NNN-slug]  <titulo>
  Área: X · Severidade: Y · Descoberto por @Z em YYYY-MM-DD
  Status: <aberto | investigado | fora-de-escopo>
  Evidências em refs/: <N arquivos>
  Caminho: .audit/findings/NNN-slug/

(repete para cada finding)

───────────────────────────────────────────────────────────────
  Resolvidos (últimos 7 dias)
───────────────────────────────────────────────────────────────

[NNN-slug]  <titulo>
  Resolvido em YYYY-MM-DD por @X
  Caminho: .audit/resolved/NNN-slug/

(repete)
```

### 6. Regras

- NÃO modificar nenhum arquivo.
- NÃO executar comandos de build/test (skill é puramente leitura de disco).
- Se `.audit/board.md` estiver dessincronizado com o sistema de arquivos, **reportar a inconsistência** mas não corrigir.
- Output deve ser scan-friendly — dev rodando essa skill está em modo "resumo rápido".
- Se `AREAS_PROJETO` não estiver preenchido no BOOTSTRAP, agrupar por `area` do frontmatter de cada README.md, ou listar como "não categorizado".

## Regras de qualidade

- NUNCA inventar números — se não achou a info, dizer "indisponível".
- SEMPRE ler o BOOTSTRAP pra pegar a missão/data-alvo atualizadas.
- Se houver findings com YAML frontmatter mal formado, listar separadamente como "findings com problema estrutural".
