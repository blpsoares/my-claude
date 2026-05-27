# {{TITULO_HUMANO}}

<!--
  Template pro corpo da página criada no database "PDD - Findings" do Notion.
  Preencher em linguagem amigável ao QA (não técnica).
  
  Regras:
  - Nunca usar: função, endpoint, query, controller, repositório, state
  - Sempre usar: tela, botão, campo, valor, lista, formulário, tabela
  - Citar sempre a tela e a interação (ex: "na tela de pedidos, ao clicar Salvar")
-->

## Sobre o que se trata

{{SOBRE}}

<!--
  2-3 frases em linguagem leiga explicando o problema que foi corrigido.
  O QA precisa entender o CONTEXTO, não o código.
  
  Exemplo bom:
    "Na tela de resumo do pedido, o valor total aparecia errado — mostrava
    apenas 1 item mesmo quando o pedido tinha 5. Isso fazia o operador
    confirmar o pedido com valor diferente do que seria cobrado."
  
  Exemplo ruim:
    "A função calcularTotal do repositório de pedidos estava usando DATEDIFF
    incorretamente ao invés da stored procedure correta."
-->

## O que foi corrigido

Na tela **{{TELA}}**, {{INTERACAO_AFETADA}}:

- **Antes**: {{ANTES}}
- **Agora**: {{AGORA}}

<!--
  TELA: nome amigável da tela ou área (ex: "Resumo do Pedido", "Formulário de Cadastro")
  INTERACAO_AFETADA: onde o QA vai ver a mudança
    ex: "ao clicar em Confirmar", "ao digitar no campo Desconto",
        "após adicionar itens", "ao exportar o relatório"
  ANTES: comportamento errado, descrito como o usuário o percebia
  AGORA: comportamento correto
-->

## Por que isso importa

{{IMPACTO}}

<!--
  1-2 frases sobre o impacto real na operação.
  
  Exemplo bom:
    "Sem essa correção, o operador poderia confirmar pedidos com valores
    divergentes do que será cobrado, gerando retrabalho e reclamações."
  
  Exemplo ruim (técnico):
    "Correção de lógica de agregação no módulo de pedidos."
-->

## Dados de referência pra testar

{{CASOS_REFERENCIA}}

<!--
  Lista dos casos que o QA deve usar para reproduzir.
  Formato:
    - Caso 12345 (sistema novo) / 67890 (sistema de referência) — descrição do cenário
  
  Se o finding não definiu caso específico, omitir esta seção.
-->

## Links de referência

- **Detalhes técnicos da correção** (pra devs curiosos): [{{PATH_RESOLUTION}}]({{URL_GITHUB_RESOLUTION}})
- **PR no GitHub**: [{{PR_TITULO}}]({{PR_URL}})
- **Finding original**: [{{PATH_README}}]({{URL_GITHUB_README}})

<!--
  URL_GITHUB_* = URL absoluta no GitHub apontando pro arquivo no branch main
  PR_URL = URL do pull request mergeado
-->

---

*Página gerada automaticamente em {{DATA_CRIACAO}} pela skill `/audit-qa`.
Se algo aqui estiver errado, fale com {{DEV_RESPONSAVEL}}.*
