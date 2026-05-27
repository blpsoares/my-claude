# Resolução — {{TITULO}}

**Finding**: [`./README.md`](./README.md) · [`./investigation.md`](./investigation.md)
**Resolvido por**: {{AUTOR}}
**Data**: {{DATA}}
**Branch**: {{BRANCH}}

> Leia antes: [`.audit/BOOTSTRAP.md`](../../BOOTSTRAP.md), o [`README.md`](./README.md) e a [`investigation.md`](./investigation.md) deste finding.

---

## 1. O que foi feito

{{RESUMO}}

<!--
3-5 frases explicando a correção. Em ordem:
- Qual hipótese da investigação foi confirmada
- Qual mudança foi aplicada
- Qual trecho do sistema de referência motivou a mudança
-->

---

## 2. Arquivos modificados

{{ARQUIVOS_MODIFICADOS}}

<!--
Tabela:

| Arquivo | Linhas | O que mudou |
|---|---|---|
| src/services/pedido.ts | 80-95 | Corrige cálculo de desconto para espelhar sistema de referência |
| tests/pedido.test.ts | +12 | Novo teste de regressão do caso do finding |
-->

---

## 3. Referência ao sistema de referência

{{REFERENCIA_SISTEMA}}

<!--
- Arquivo/spec que serviu de modelo: <path>:<linha>
- Função/regra específica: <nome>
- Cite o trecho relevante se útil para entender a decisão
-->

---

## 4. Validações executadas

### 4.1 Check estático
```
$ {{CHECK_CMD}}
{{CHECK_OUTPUT}}
```

### 4.2 Testes
```
$ {{TEST_CMD}}
{{TEST_OUTPUT}}
```

### 4.3 Paridade com sistema de referência

**Caso de referência usado**: {{CASO_REFERENCIA}}

**Evidências em `./refs/`**:
{{EVIDENCIAS_PARIDADE}}

<!--
Lista dos arquivos de evidência:
- refs/paridade-referencia.png — screenshot do sistema de referência pós-fix
- refs/paridade-novo.png — screenshot do sistema novo pós-fix (deve ser idêntico em comportamento)
- refs/paridade-dados.txt — diff de resultados entre sistema novo e de referência
-->

**Resultado da comparação**:

{{RESULTADO_COMPARACAO}}

<!--
- [x] Valor correto
- [x] Quantidade de itens idêntica
- [x] Comportamento visual equivalente
- ou qualquer critério do README.md
-->

---

## 5. Critérios de aceitação do finding

{{CRITERIOS_FINAIS}}

<!--
Copiar a seção 8 do README.md e marcar cada item [x] com evidência.

Exemplo:
- [x] Total correto (R$ X,XX → R$ X,XX)
- [x] {{CHECK_CMD}} verde
- [x] {{TEST_CMD}} verde
- [x] Paridade validada com caso de referência <ID>
-->

---

## 6. Riscos remanescentes / notas

{{RISCOS_REMANESCENTES}}

<!--
- Possíveis regressões em cenários não testados
- Áreas correlatas que podem precisar de validação adicional
- Débitos técnicos criados
-->

---

## 7. Comando de commit sugerido (⚠️ dev executa manualmente)

```bash
git add -A
git commit -m "{{COMMIT_MESSAGE}}

{{COMMIT_BODY}}"
```

> Este fix NÃO foi commitado automaticamente.
> Regra inviolável do PDD: push/commit é feito APENAS pelo humano.
