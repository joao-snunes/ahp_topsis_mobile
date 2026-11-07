# Requisitos funcionais

App de avaliação comparativa de produtos (AHP-TOPSIS) — PDM 2026/2, UFU.

Prioridade: **Essencial** entra obrigatoriamente na entrega; **Desejável** entra se houver tempo.

| ID | Requisito | Tela | Prioridade |
| --- | --- | --- | --- |
| RF01 | Exibir tela de abertura com nome e logo do app e seguir para as categorias | Splash | Essencial |
| RF02 | Baixar do JSON remoto as categorias, os critérios e os produtos ao abrir o app | Splash | Essencial |
| RF03 | Em caso de falha de conexão (sem internet, tempo esgotado ou erro do servidor), mostrar a mensagem "Não foi possível carregar os dados. Verifique sua conexão com a internet." e o botão "Tentar novamente" | Splash / Categorias | Essencial |
| RF04 | Listar as categorias disponíveis (Notebooks e Smartphones) | Categorias | Essencial |
| RF05 | Listar os produtos da categoria escolhida, com foto, nome e marca | Produtos | Essencial |
| RF06 | Exibir os detalhes de um produto: foto, nome, modelo, fabricante, característica ou nota de referência em cada critério com unidade e fonte das especificações | Detalhe do produto | Essencial |
| RF07 | Indicar, para cada critério, se ele é benefício (quanto maior, melhor) ou custo (quanto menor, melhor), nas telas de Pesos AHP, Avaliação e Resumo | Pesos AHP / Avaliação / Resumo | Essencial |
| RF08 | Permitir comparar os critérios par a par com um slider por par, na escala de Saaty (1 a 9 para cada lado), todos começando em "igual", exibindo abaixo de cada slider uma frase com a escolha atual (ex.: "Preço é 3× mais importante que Peso (kg)" ou "Preço e Peso (kg) têm a mesma importância") | Pesos AHP | Essencial |
| RF09 | Calcular a cada mudança no slider os pesos (média geométrica), λmax, IC e RC | Pesos AHP | Essencial |
| RF10 | Exibir os pesos de cada critério e o RC; se RC > 0,10, mostrar alerta e bloquear o avanço até o usuário revisar | Pesos AHP | Essencial |
| RF11 | Exibir as características de cada produto em cada critério, vindas do JSON, sem permitir edição | Avaliação | Essencial |
| RF12 | Permitir ajustar as notas subjetivas (critérios com "editable": true) de 1 a 10, partindo da nota de referência do JSON; o ajuste vale só para a avaliação atual | Avaliação | Essencial |
| RF13 | Restringir as notas subjetivas a números inteiros de 1 a 10 | Avaliação | Essencial |
| RF14 | Exibir resumo com os pesos e a matriz de decisão, com o botão "Processar" | Resumo | Essencial |
| RF15 | Calcular o TOPSIS: matriz normalizada e ponderada, solução ideal (A+) e anti-ideal (A−), distâncias d+ e d− e coeficiente CC | Resumo → Ranking | Essencial |
| RF16 | Exibir o ranking dos produtos por CC, do maior para o menor, com destaque para o vencedor e os valores de CC, d+ e d− | Ranking | Essencial |
| RF17 | Exibir os detalhes do cálculo: matriz ponderada, A+, A− e gráfico comparativo dos CC | Detalhes do cálculo | Essencial |
| RF18 | Permitir iniciar uma nova avaliação, que limpa tudo: sliders voltam para "igual" e notas ajustadas voltam às de referência. Escolher outra categoria também começa uma avaliação do zero | Ranking / Categorias | Essencial |
| RF19 | Permitir voltar para telas anteriores mantendo tudo o que foi preenchido (sliders e notas ajustadas) | Todas | Essencial |
| RF20 | Permitir escolher quais produtos da categoria entram na comparação (mínimo 2) | Produtos | Desejável |
| RF21 | Quando RC > 0,10, apontar qual comparação mais contradiz as outras | Pesos AHP | Desejável |
| RF22 | Permitir mexer nos pesos e ver o ranking mudar na hora (análise de sensibilidade) | Ranking | Desejável |
| RF23 | Considerar empatados os produtos cujo CC é igual com 4 casas decimais. Empatados ficam na mesma posição, com a indicação "empate", e a posição seguinte pula as ocupadas (ex.: 1º, 2º, 2º, 4º). Se o empate for em 1º lugar, todos os empatados recebem o destaque de vencedor, com a frase "Empate entre X e Y". Se todos os produtos forem idênticos em todos os critérios, avisar que não há como diferenciá-los | Ranking | Essencial |
| RF24 | Validar os dados ao carregar o JSON: não exibir categoria com menos de 2 produtos, com número de critérios fora de 2 a 10, com produto sem característica ou nota em algum critério, com característica ≤ 0 ou com nota subjetiva que não seja um número inteiro de 1 a 10 (registrar o motivo no Logcat). Se o JSON não puder ser lido ou nenhuma categoria for válida, mostrar a mensagem "Os dados dos produtos estão com problema. Tente novamente mais tarde." e o botão "Tentar novamente" | Splash / Categorias | Essencial |

Enquanto RF20 não for feito, todos os produtos da categoria entram na comparação.
