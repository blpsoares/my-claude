---
name: "audit-resolve"
description: "Implementa a correção de um finding já investigado, valida contra o sistema de referência, gera resolution.md e move a pasta para .audit/resolved/. NUNCA commita nem pusha — apenas sugere o comando ao dev."
argument-hint: "ID do finding (ex: 007 ou 007-checkout-valor-incorreto)"
user-invocable: true
disable-model-invocation: true
---

## User Input

```text
$ARGUMENTS
```

## Contexto

Parte do método **PDD (Parity-Driven Development)**. Esta é a fase final do ciclo de um finding — onde código é modificado.

**Regra inviolável do PDD**: push é feito APENAS pelo humano. Esta skill NUNCA commita autonomamente — apenas prepara o fix e sugere o comando.

## Outline

### 1. Verificações iniciais

- Leia `.audit/BOOTSTRAP.md`. Se NÃO existir: pare e instrua `/audit-bootstrap`.
- Do BOOTSTRAP, extraia:
  - `CHECK_CMD` e `TEST_CMD` — obrigatórios para validação
  - `REFERENCIA_NOME` e `REFERENCIA_RESTRICOES` — para saber o que NÃO fazer
  - Regras invioláveis do projeto (Seção 12 do BOOTSTRAP)
- Se existe documento de regras do projeto, leia para aplicar ao fix.
- Parse `$ARGUMENTS` (mesma lógica do `/audit-investigate`).
- Confirme que a pasta do finding existe em `.audit/findings/<NNN>-<slug>/`.
- Leia `README.md` do finding.
- Leia `investigation.md` — **se NÃO existir, pare** e instrua:
  > Este finding ainda não foi investigado. Rode `/audit-investigate {{ID}}` primeiro.
  > Corrigir sem investigação é receita pra regressão — é por isso que o PDD força essa ordem.
- Se o `investigation.md` tem seção "Fora de escopo" preenchida, pare e sugira:
  > Investigação concluiu que este finding está fora de escopo. Recomendação: mover a pasta
  > pra `.audit/resolved/` diretamente e anotar no board como "encerrado sem correção".
  > Quer que eu faça isso? (sim/não)
- Verifique se `resolution.md` já existe:
  - Se SIM: "Já existe resolução. Quer (a) revisar, (b) sobrescrever após nova tentativa?"
- Carregue o template em `.claude/skills/audit-resolve/template.md`.
- Confirme estado limpo do git: `git status --short`. Se houver mudanças não commitadas, pergunte:
  > Há mudanças não commitadas. Sugiro commitar antes de começar pra isolar o diff deste fix. Quer pausar?

### 2. Criação da branch de trabalho

Antes de qualquer modificação, crie a branch:

```bash
git checkout -b audit/NNN-<slug>
```

O prefixo `audit/` identifica que a branch veio de um finding PDD. Reporte ao dev qual branch foi criada.

Se a branch já existir, pergunte: "(a) usá-la como está, ou (b) criar com nome alternativo?"

### 3. Confirmação do plano com o dev

Antes de modificar QUALQUER arquivo, apresente:

```
Plano de resolução do finding NNN:

Baseado na investigação:
- Hipótese a atacar: <hipotese mais provável do investigation.md>
- Arquivos a modificar:
  * <arquivo 1> (motivo da mudança)
  * <arquivo 2> (motivo da mudança)
- Testes a adicionar/modificar:
  * <teste 1>
- Validação pós-fix:
  * {{CHECK_CMD}}
  * {{TEST_CMD}}
  * Comparação com sistema de referência usando caso <X>

Posso prosseguir? (sim / mudar plano / cancelar)
```

Espere resposta explícita.

### 4. Implementação

Execute o plano. Regras gerais (adapte às regras do projeto do BOOTSTRAP):

- **Respeitar as regras do projeto** extraídas do BOOTSTRAP (Seção 12) e do documento de regras, se existir
- **Fidelidade ao sistema de referência**: qualquer mudança de lógica deve ser rastreável ao arquivo/spec do sistema de referência que a motivou — citar no comentário ou na mensagem de commit
- **Toda nova função adicionada tem teste** (se as regras do projeto exigirem)
- **Não hardcodar configuração** que deveria vir de variável de ambiente
- Usar `Edit` pra mudanças pontuais, `Write` só pra arquivos novos

### 5. Validação automatizada (ordem obrigatória)

Após cada bloco de mudanças:

1. **Check estático**: `{{CHECK_CMD}}`
   - Se falhar: corrija antes de prosseguir. Não continue com erro.
2. **Testes**: `{{TEST_CMD}}`
   - Se falhar: analise. Se é regressão do fix, ajuste. Se é teste antigo inválido, documente e discuta com dev.
3. **Teste específico**: se o finding tem teste de regressão, rode isoladamente.

### 6. Validação de paridade com sistema de referência (obrigatória)

Não aceite o fix como pronto sem evidência de paridade. Pelo menos UMA das abordagens:

- **Dado-a-dado**: executar a mesma operação no sistema novo e no de referência com o caso de referência; capturar diff em texto → `refs/paridade-<data>.txt`
- **Visual**: abrir ambos os sistemas (via MCP de browser ou dev manualmente), capturar screenshots → `refs/paridade-referencia.png` e `refs/paridade-novo.png`
- **Ambos** (ideal)

Se não tiver acesso direto ao sistema de referência, instrua o dev:

```
Preciso que você valide manualmente:
1. Abra o sistema de referência com o caso <X>
2. Abra o sistema novo com o caso <Y> equivalente
3. Reproduza o cenário do finding
4. Capture screenshots em refs/paridade-referencia.png e refs/paridade-novo.png
5. Me confirme se o comportamento agora é idêntico
```

### 7. Escrita do `resolution.md`

Use o template. Preencha:
- Resumo do que foi feito em 3-5 frases
- Lista de arquivos modificados com `file:linha` e resumo da mudança
- Testes adicionados/modificados
- Resultado dos checks (check/test)
- Evidência de paridade (paths das capturas em `refs/`)
- Referência ao arquivo/spec do sistema de referência que guiou a correção

Grave em `.audit/findings/<pasta>/resolution.md`.

### 8. Mover pasta pra resolved/

```bash
mv .audit/findings/NNN-<slug>/ .audit/resolved/NNN-<slug>/
```

### 9. Atualização do board

Em `.audit/board.md`:
- Remover da seção "Investigados (prontos pra resolver)"
- Adicionar em "Resolvidos (últimos 7 dias)": `- [x] NNN-<slug> — <resumo 1 linha> (resolvido em YYYY-MM-DD por @autor)`

### 10. ⚠️ NUNCA commitar — apenas sugerir

Reporte ao dev:

```
Fix pronto e validado. Resumo:
- <X> arquivos modificados
- {{CHECK_CMD}} ✅
- {{TEST_CMD}} ✅
- Paridade com {{REFERENCIA_NOME}}: ✅ (evidências em .audit/resolved/NNN-<slug>/refs/)

Pasta movida pra .audit/resolved/NNN-<slug>/
Board atualizado.

🛑 NÃO COMMITEI (regra inviolável do PDD: push/commit é feito APENAS pelo humano).

Pra commitar, execute:

git add -A
git commit -m "fix(audit): NNN — <titulo curto do finding>

Corrige <sintoma resumido>.
Referência: .audit/resolved/NNN-<slug>/.
Baseado no comportamento de <arquivo/spec do sistema de referência>."
```

**Se o dev pedir pra você commitar**: recuse educadamente e reforce a regra.
**Se o dev pedir pra você pushar**: idem.

### 11. Encerramento

- Confirme path do `resolution.md`
- Confirme que a pasta foi movida
- Confirme estado do board
- Pergunte se quer preparar o próximo finding

## Regras de qualidade

- NUNCA commitar. NUNCA pushar.
- NUNCA pular `{{CHECK_CMD}}` nem `{{TEST_CMD}}`. Se falham, corrige antes de seguir.
- SEMPRE validar paridade com sistema de referência (ou documentar por que não foi possível).
- SEMPRE confirmar plano com dev ANTES de modificar código.
- SEMPRE referenciar o arquivo/spec do sistema de referência que motivou a mudança.
- SEMPRE preservar comportamento em testes existentes — se um teste antigo quebra, é sinal de regressão.
