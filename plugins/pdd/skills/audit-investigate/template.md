# Investigação — {{TITULO}}

**Finding**: [`../README.md`](./README.md)
**Investigado por**: {{AUTOR}}
**Data**: {{DATA}}
**Abordagem usada**: {{ABORDAGEM}} — {{ABORDAGEM_JUSTIFICATIVA}}

> Leia antes: [`.audit/BOOTSTRAP.md`](../../BOOTSTRAP.md) e o [`README.md`](./README.md) deste finding.

---

## 1. Execução da investigação

{{EXECUCAO}}

<!-- 
Se caminho A: trechos de código comparados, com arquivo:linha (sistema novo vs referência)
Se caminho B: queries/chamadas rodadas, resultados capturados, diffs
Se caminho C: referências às imagens em refs/, o que foi observado
Se caminho D: cada sub-fase separadamente
-->

---

## 2. Fatos observados (confirmados)

{{FATOS}}

<!--
Lista numerada de coisas que foram EFETIVAMENTE verificadas nesta investigação.
Evidência + citação de arquivo/query/screenshot.
Não incluir suposições aqui.
-->

---

## 3. Hipóteses de causa raiz (ranqueadas)

{{HIPOTESES}}

<!--
Formato por hipótese:

### Hipótese H1 (probabilidade: XX%)
**Descrição**: ...
**Evidência a favor**: ...
**Evidência contra**: ...
**Como testar**: ...

### Hipótese H2 (probabilidade: YY%)
...
-->

---

## 4. Recomendação

{{RECOMENDACAO}}

<!--
- Qual hipótese atacar primeiro e por quê.
- Qual arquivo e linha provável pra modificar.
- Estratégia do fix em 2-4 frases.
-->

---

## 5. Riscos conhecidos do fix

{{RISCOS}}

<!--
- Regressões possíveis em outras áreas do sistema.
- Testes que DEVEM ser rodados (além do padrão).
- Casos de referência que DEVEM ser validados após o fix.
-->

---

## 6. Fora de escopo? (preencher só se aplicável)

{{FORA_ESCOPO}}

<!--
Se a investigação revelou que o "bug" é na verdade:
- Divergência de dados entre banco dev e produção (não é código)
- Feature pendente, não bug
- Limitação conhecida do sistema de referência
Marcar aqui e justificar. Se marcado, /audit-resolve NÃO deve corrigir código —
deve mover pra resolved/ com nota de encerramento.
-->
