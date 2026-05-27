---
name: "audit-investigate"
description: "Investiga um finding existente — decide abordagem (estática/dinâmica/visual/combinada) em conversa com o dev, executa, e documenta descobertas em investigation.md. NÃO modifica código — só entende e diagnostica."
argument-hint: "ID do finding (ex: 007 ou 007-checkout-valor-incorreto)"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Parte do método **PDD (Parity-Driven Development)**. Esta skill **investiga** — não corrige. O objetivo é entender a causa raiz com evidência suficiente pra que `/audit-resolve` tenha tudo o que precisa.

## Outline

### 1. Verificações iniciais

- Leia `.audit/BOOTSTRAP.md`. Se NÃO existir: pare e instrua `/audit-bootstrap`.
- Do BOOTSTRAP, extraia: `REFERENCIA_NOME`, `REFERENCIA_ACESSO`, MCPs disponíveis.
- Se existe documento de regras do projeto, leia para conhecer restrições (ex: quais operações de banco são proibidas).
- Parse `$ARGUMENTS`:
  - Se vazio: liste findings abertos em `.audit/findings/` e peça qual investigar.
  - Se formato `NNN`: encontre pasta `.audit/findings/NNN-*`.
  - Se formato `NNN-<slug>`: encontre diretamente.
  - Se não encontrou: erro claro listando findings disponíveis.
- Leia `.audit/findings/<pasta>/README.md` completo.
- Liste `.audit/findings/<pasta>/refs/` e leia qualquer `.md` ou `.txt` de evidência. Anote nome das imagens.
- Verifique se `investigation.md` já existe:
  - Se SIM: pergunte "Já existe investigação neste finding. Quer (a) continuar de onde parou, (b) adicionar mais achados, (c) sobrescrever?"
  - Se NÃO: prossiga.
- Carregue o template em `.claude/skills/audit-investigate/template.md`.

### 2. Apresentação do finding (visão rápida pro dev)

Antes de decidir abordagem, resuma em 4-6 linhas o que o finding diz:

```
Finding NNN — <titulo>
Área: <area>, Severidade: <severidade>
Sintoma: <sintoma em 1 frase>
Esperado: <comportamento do sistema de referência em 1 frase>
Evidências em refs/: <lista ou "nenhuma">
Observações durante reprodução: <sim, resumo | não coletadas>
```

### 3. ⭐ DECISÃO de abordagem — conversa com o dev

Apresente as 4 opções:

```
Pra investigar este finding vejo 4 caminhos possíveis. Qual cabe aqui?

A) ANÁLISE ESTÁTICA (eu sozinho — mais rápido)
   Faço: grep do código novo, grep/leitura do sistema de referência,
          identifico divergências de lógica ou estrutura.
   Bom quando: já sabemos o módulo, a causa provável é diferença
          de implementação código-a-código.
   Custo: 5-15 min meu. Você nem precisa acompanhar.

B) ANÁLISE DINÂMICA (eu sozinho, com banco ou API)
   Faço: rodo queries/chamadas via MCP disponível com o caso de
          referência, comparo resultados, calculo diffs.
   Bom quando: suspeita de divergência de DADOS (valor errado,
          quantidade errada, cálculo diferente), não de código.
   Custo: operações de leitura. 10-20 min meu.

C) REPRODUÇÃO VISUAL (você dirige, eu leio)
   Faço: você reproduz o bug em ambos os sistemas ao vivo, joga
          screenshots e outputs em refs/, eu correlaciono com código.
   Bom quando: bug é visual/UX, ou depende de login complexo/
          estado que só você consegue montar.
   Custo: seu tempo (variável).

D) COMBINADA — A + B (ou A + C, ou B + C)
   Quando a causa é multicamada (código + dados, por exemplo).
   Você escolhe a ordem.

Qual?
```

**Execute o caminho escolhido.**

### 4. Execução — caminho A (estática)

1. Identifique os arquivos do sistema novo e do sistema de referência listados no README.
2. Use `Read`/`Grep`/`Glob` pra ler os trechos relevantes.
3. **Para cada função/query divergente**:
   - Cite o trecho do sistema novo (arquivo + linha)
   - Cite o trecho equivalente no sistema de referência (arquivo + linha)
   - Descreva a divergência em 1 frase
   - Classifique: ❌ divergência crítica / ⚠️ divergência de forma sem impacto / ✅ equivalente
4. Se a divergência não for óbvia, consulte histórico do sistema de referência (`git log` se for repositório).

### 5. Execução — caminho B (dinâmica com banco/API)

1. Use o MCP disponível para banco ou chamadas de API:
   - Execute a query/operação do sistema novo com o caso de referência, capture resultado.
   - Execute a query/operação equivalente do sistema de referência, capture resultado.
   - Compare coluna por coluna ou campo por campo.
2. **NUNCA** execute operações destrutivas — apenas leitura. Antes de rodar qualquer query, confirme com o dev: "Vou rodar esta query: `<query>`. Tudo bem?"
3. Documente: resultado do sistema novo, resultado do sistema de referência, diff.
4. Se a diferença sugere que o BANCO tem dados divergentes (não código), marque e pause — pode não ser bug do sistema novo, e sim de ambiente.

### 6. Execução — caminho C (visual via dev)

1. Instrua o dev:
   > Abra os dois sistemas agora. Reproduza os passos do README.md. Dropa em `refs/`:
   > - Screenshot do sistema de referência (nome: `referencia-<area>.png`)
   > - Screenshot do sistema novo (nome: `novo-<area>.png`)
   > - Se houver valor divergente: export da tela ou output da API (`novo-api-<rota>.json`)
   > Me avise quando terminar.

2. Após confirmação, liste `refs/`, leia imagens e textos.
3. Descreva em 5-10 linhas o que observou + correlação com o código (se já olhou).
4. Se precisar de mais evidências, peça especificamente.

### 7. Execução — caminho D (combinado)

Execute na ordem acordada. Documente cada sub-fase separadamente.

### 8. Síntese — hipóteses ranqueadas

Ao final, SEMPRE escreva:

1. **Fatos observados**: lista numerada de coisas CONFIRMADAS (não suposições)
2. **Hipóteses de causa raiz**: lista ranqueada por probabilidade (% aproximado)
   - Cada hipótese: descrição + evidência que a sustenta + evidência que a enfraquece
3. **Recomendação pra resolver**: qual hipótese atacar primeiro e como
4. **Riscos conhecidos do fix**: regressões possíveis, testes a rodar

### 9. Escrita do `investigation.md`

Use o template em `.claude/skills/audit-investigate/template.md`. Inclua:
- Abordagem escolhida (A/B/C/D) e justificativa em 1 frase
- Tudo que foi observado durante a execução
- Síntese (fatos, hipóteses, recomendação, riscos)
- Timestamp e autor

Grave em `.audit/findings/<pasta>/investigation.md`.

### 10. Atualização do board

Em `.audit/board.md`, mova o finding de "Disponíveis" para "Investigados (prontos pra resolver)":

```markdown
## Investigados (prontos pra resolver)
- [ ] NNN-<slug> — <recomendação em 1 linha>
```

### 11. Encerramento

Reporte:
- Path do `investigation.md`
- Resumo em 3 linhas das hipóteses ranqueadas
- Próximo passo: "Quando quiser corrigir, rode `/audit-resolve NNN`."
- Se a investigação indicou que o bug NÃO é do sistema novo (ex: dados divergentes no banco dev, feature pendente), aponte claramente e sugira mover o finding pra `resolved/` com nota de "fora de escopo".

## Regras de qualidade

- NUNCA modificar código (este comando é read-only em termos de fonte).
- NUNCA executar operações de escrita no banco sem confirmação explícita do dev.
- SEMPRE confirmar qualquer query destrutiva antes de rodar.
- SEMPRE mostrar hipóteses mesmo que a causa pareça óbvia.
- SEMPRE citar arquivo:linha ao mencionar código.
- SEMPRE separar "fato observado" de "hipótese" — não misturar.
