# {{TITULO_TESTE}}

<!--
  Template pro corpo do card de teste criado no database "PDD - Testes QA".
  Um card = um cenário de teste específico.
  
  TITULO_TESTE: frase clara do que este teste verifica
    Exemplo bom: "Conferir que o total do pedido 12345 exibe R$ 7.103,47 com 5 itens"
    Exemplo ruim: "Teste 1", "Validar cálculo", "Resumo correto"
-->

## O que este teste verifica

{{OBJETIVO}}

<!--
  1-2 frases explicando o que está sendo testado, do ponto de vista do usuário.
  
  Exemplo bom:
    "Este teste garante que ao confirmar um pedido com 5 itens de categorias
    diferentes, o sistema exibe o valor total correto de R$ 7.103,47, idêntico
    ao que o sistema anterior mostrava."
  
  Exemplo ruim:
    "Valida função calcularTotal com os dados do pedido 12345."
-->

## Antes de começar

{{PRE_REQUISITOS}}

<!--
  Lista do que o QA precisa ter/saber antes de testar:
  - Login no sistema (URL, credenciais onde ficam)
  - Login no sistema de referência (idem, se a comparação é necessária)
  - Identificador do caso de referência
  - Qualquer configuração necessária
  
  Exemplo:
    - Acesso ao sistema novo (URL de staging ou produção)
    - Acesso ao sistema de referência (combinar com o dev se necessário)
    - Caso de referência no sistema novo: ID 12345
    - Caso equivalente no sistema de referência: ID 67890
-->

## Como testar (passo a passo)

{{PASSOS}}

<!--
  Lista numerada de passos EXATOS que o QA deve seguir.
  Cada passo deve ser uma ação concreta e clara.
  
  Exemplo:
    1. Faça login no sistema com seu usuário.
    2. Na lista de pedidos, localize e abra o pedido 12345.
    3. Aguarde a tela de detalhes abrir.
    4. Clique no botão "Confirmar" no canto inferior direito.
    5. Observe o resumo exibido antes de confirmar.
    6. Anote o número de itens listados e o valor total.
-->

## O que deve acontecer (resultado esperado)

{{RESULTADO_ESPERADO}}

<!--
  Descrição clara do estado que o QA deve encontrar ao final dos passos.
  Ser específico: valor exato, número de itens, textos exatos quando possível.
  
  Exemplo bom:
    - Devem aparecer **5 itens** no resumo.
    - O valor total deve ser **exatamente R$ 7.103,47**.
    - Cada item deve exibir: nome, quantidade e valor unitário.
    - Não deve aparecer nenhuma mensagem de erro.
  
  Exemplo ruim:
    "Tudo correto."
    "Valor deve bater."
-->

## Como comparar com o sistema de referência (se aplicável)

{{COMPARACAO_REFERENCIA}}

<!--
  Passos pra fazer a mesma operação no sistema de referência e comparar.
  Só incluir se a comparação visual faz sentido pra este teste.
  
  Exemplo:
    1. Em outra aba, abra o sistema de referência.
    2. Localize o caso equivalente (ID 67890).
    3. Reproduza os mesmos passos.
    4. Confira: mesmo número de itens (5), mesmo valor total (R$ 7.103,47).
    5. Se houver qualquer diferença, anote o que divergiu.
-->

## Se o teste falhar

Se o resultado não bateu com o esperado:

1. Anote com precisão **o que apareceu** (números, textos, mensagens de erro).
2. Tire screenshots da tela do sistema novo **e** do sistema de referência (se comparação for necessária).
3. Marque este card como **Rejeitado** no status.
4. Adicione um comentário aqui no card descrevendo:
   - O que você viu de diferente
   - Em qual passo a divergência apareceu
   - Qualquer contexto útil (navegador, horário, comportamento estranho)

O dev responsável vai receber aviso e voltar pra ajustar.

## Links de referência

- **Finding original**: (ver na coluna "Finding" desta página — leva à descrição completa)
- **Detalhes técnicos**: [{{PATH_RESOLUTION}}]({{URL_GITHUB_RESOLUTION}})

---

*Card gerado em {{DATA_CRIACAO}} pela skill `/audit-qa`. Finding: `{{NNN}}-{{SLUG}}`.*
