---
id: "{{ID}}"
titulo: "{{TITULO}}"
slug: "{{SLUG}}"
descoberto-por: "{{DESCOBERTO_POR}}"
data-descoberta: "{{DATA_DESCOBERTA}}"
area: "{{AREA}}"
modulo-provavel: "{{MODULO_PROVAVEL}}"
severidade: "{{SEVERIDADE}}"
status: "aberto"
---

> ## ⚠️ Contexto obrigatório antes de mexer neste finding
>
> Leia **nesta ordem**:
> 1. [`.audit/BOOTSTRAP.md`](../../BOOTSTRAP.md) — contexto operacional (URLs, comandos, casos de referência)
> 2. O sistema de referência (ver Seção 2 do BOOTSTRAP)
>
> Sem esse contexto, suas decisões vão divergir de decisões anteriores.

---

# {{TITULO}}

## 1. Sintoma observado (no sistema novo)

{{SINTOMA}}

## 2. Comportamento esperado (no sistema de referência)

{{COMPORTAMENTO_ESPERADO}}

## 3. Como reproduzir

{{PASSOS_REPRODUCAO}}

**Caso de referência**: {{CASO_REFERENCIA}}

## 4. Arquivos prováveis envolvidos

**Sistema novo**:
{{ARQUIVOS_SISTEMA_NOVO}}

**Sistema de referência** — fonte de verdade:
{{ARQUIVOS_SISTEMA_REFERENCIA}}

## 5. Hipótese de causa

{{HIPOTESE}}

## 6. Observações durante a reprodução

{{OBSERVACOES_REPRODUCAO}}

## 7. Evidências (arquivos em `./refs/`)

{{EVIDENCIAS_LISTA}}

> Dropar prints, exports de query, screenshots do sistema de referência e do sistema novo em `refs/`.
> Nomes descritivos (`referencia-tela-pedidos.png`, `novo-tela-pedidos.png`, `query-result.txt`).

## 8. Critério de aceitação

O finding é considerado **resolvido** quando:

{{CRITERIOS_ACEITACAO}}

E adicionalmente (padrão do PDD — não remover):

- [ ] `{{CHECK_CMD}}` passa sem erros
- [ ] `{{TEST_CMD}}` passa sem erros
- [ ] Comparação lado a lado sistema novo vs sistema de referência feita para este caso de referência

---

## 9. Fluxo deste finding

```
[x] criado via /audit-new em {{DATA_DESCOBERTA}}
[ ] investigado via /audit-investigate
[ ] resolvido via /audit-resolve
[ ] movido para .audit/resolved/
```
