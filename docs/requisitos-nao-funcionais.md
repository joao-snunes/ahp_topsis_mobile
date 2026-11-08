# Requisitos não funcionais

App de avaliação comparativa de produtos (AHP-TOPSIS) — PDM 2026/2, UFU.

**Prioridade:** **Essencial** entra obrigatoriamente na entrega; **Desejável** entra se houver tempo.

| ID | Requisito | Prioridade |
| --- | --- | --- |
| RNF01 | O aplicativo deve ser desenvolvido para Android utilizando Kotlin e Jetpack Compose. | Essencial |
| RNF02 | O aplicativo deve utilizar arquitetura MVVM, separando a interface, o controle de estado, as regras de negócio e o acesso aos dados. | Essencial |
| RNF03 | A interface deve ser simples e fácil de entender, permitindo que o usuário navegue pelas telas sem precisar de conhecimentos técnicos sobre AHP ou TOPSIS. | Essencial |
| RNF04 | Os cálculos de AHP e TOPSIS devem ser realizados de forma rápida, sem travar a interface do aplicativo durante a utilização. | Essencial |
| RNF05 | Os dados dos produtos, categorias e critérios devem ser obtidos de uma fonte remota em formato JSON, utilizando Retrofit e Gson. | Essencial |
| RNF06 | O aplicativo deve apresentar uma mensagem de erro quando não conseguir carregar os dados da fonte remota, permitindo que o usuário tente novamente. | Essencial |
| RNF07 | O aplicativo deve manter os dados preenchidos pelo usuário durante a navegação entre as telas da avaliação, evitando que sliders e notas sejam perdidos ao voltar para uma tela anterior. | Essencial |
| RNF08 | O aplicativo deve funcionar em dispositivos Android compatíveis com a versão mínima definida no projeto, mantendo a interface adaptada para diferentes tamanhos de tela. | Essencial |
| RNF09 | O código deve ser organizado em classes e arquivos separados de acordo com suas responsabilidades, facilitando a manutenção e a divisão do trabalho entre os integrantes da equipe. | Essencial |
| RNF10 | Os textos apresentados ao usuário devem estar em português e utilizar uma linguagem simples e clara. | Essencial |
| RNF11 | O aplicativo deve utilizar componentes visuais consistentes entre as telas, mantendo padrões semelhantes de botões, cards, textos, cores e espaçamentos. | Desejável |
| RNF12 | O aplicativo deve apresentar os resultados dos cálculos com valores numéricos formatados de forma legível, evitando excesso de casas decimais desnecessárias. | Desejável |
| RNF13 | O aplicativo deve registrar no Logcat os problemas encontrados durante a validação dos dados recebidos do JSON, facilitando a identificação de erros durante o desenvolvimento. | Essencial |
| RNF14 | O projeto deve utilizar controle de versão Git, com o código armazenado em um repositório compartilhado da equipe. | Essencial |
| RNF15 | O código deve possuir comentários apenas nos pontos em que sejam necessários para explicar regras ou cálculos que não sejam óbvios. | Desejável |
| RNF16 | O aplicativo não deve exigir cadastro ou login para realizar uma avaliação comparativa. | Essencial |

## Observações

- O aplicativo depende de conexão com a internet para obter os dados do JSON remoto.
- Os dados recebidos do JSON são somente para leitura durante a avaliação, não sendo necessário permitir que o usuário altere os dados dos produtos.
- As notas subjetivas alteradas pelo usuário valem somente para a avaliação atual e não precisam ser salvas permanentemente.
