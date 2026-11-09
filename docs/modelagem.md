# Modelagem — App de avaliação comparativa de produtos (AHP-TOPSIS)

Fase 1 — Requisitos + Modelagem · PDM 2026/2 · UFU

Este documento reúne os diagramas de **arquitetura (MVVM)**, **modelo de dados** e **navegação**, mais os diagramas de sequência dos dois fluxos principais.

---

## 1. Visão geral da arquitetura (MVVM em 4 camadas)

```mermaid
flowchart TB
    subgraph UI["ui/ — Jetpack Compose + Material 3"]
        direction LR
        T1["SplashScreen"]
        T2["CategoriesScreen"]
        T3["ProductsScreen"]
        T4["ProductDetailScreen"]
        T5["AhpScreen"]
        T6["EvaluationScreen"]
        T7["SummaryScreen"]
        T8["RankingScreen"]
        T9["DetailsScreen"]
    end

    subgraph VM["viewmodel/"]
        DVM["DecisionViewModel<br/>expõe StateFlow de UiState"]
    end

    subgraph DOMAIN["domain/"]
        direction LR
        subgraph ENG["engine/ — Kotlin puro"]
            AHP["AhpEngine"]
            TOP["TopsisEngine"]
        end
        subgraph MOD["model/"]
            M["Category, Criterion, Product,<br/>AhpResult, TopsisResult, DecisionResult"]
        end
    end

    subgraph DATA["data/"]
        REPO["DecisionRepository"]
        API["ApiService (Retrofit + Gson)"]
        DTO["DTOs do JSON"]
    end

    REMOTE[("JSON estático<br/>no GitHub")]

    UI -- "ações do usuário" --> DVM
    DVM -- "UiState" --> UI
    DVM --> AHP
    DVM --> TOP
    DVM --> REPO
    AHP --> M
    TOP --> M
    REPO --> API
    REPO --> DTO
    REPO -- "converte DTO em modelo e valida" --> M
    API -- "HTTPS GET, só leitura" --> REMOTE
```

**Regras de dependência**

- A `ui` só enxerga o `DecisionViewModel`. Ela desenha o estado e envia ações.
- O `DecisionViewModel` é o único que fala com o `Repository` e com os motores.
- Os motores (`AhpEngine`, `TopsisEngine`) são Kotlin puro, sem import do Android, e por isso testáveis com JUnit simples.
- O `Repository` é criado direto pelo ViewModel (`DecisionRepository(api)`), sem injeção de dependência.
- Chamadas de rede rodam em `viewModelScope.launch`, nunca na thread principal.
- Um único `DecisionViewModel` é compartilhado por todas as telas do fluxo, criado no escopo do `NavHost`/Activity, para os dados não se perderem ao navegar.

### Responsabilidade de cada camada

| Camada | Pacote | Responsabilidade | Não deve |
| --- | --- | --- | --- |
| Apresentação | `ui/` | Desenhar telas e enviar eventos | Calcular nada, acessar rede |
| Estado | `viewmodel/` | Guardar estado da avaliação, orquestrar Repository e motores | Importar componentes de tela |
| Domínio | `domain/engine/` e `domain/model/` | Cálculo AHP e TOPSIS; data classes | Importar `android.*` |
| Dados | `data/` | Buscar o JSON, converter DTO em modelo, validar (RN05, RN06, RN08) | Conter regra de cálculo |

---

## 2. Modelo de dados

### 2.1 Domínio

O modelo é **genérico**: um `Product` guarda um mapa `criterionId → valor`. Não existe um campo por critério, então trocar ou adicionar critérios no JSON não exige mudar código.

```mermaid
classDiagram
    direction LR

    class CriterionType {
        <<enumeration>>
        BENEFIT
        COST
    }

    class Category {
        +String id
        +String name
        +String description
        +String iconUrl
        +List~Criterion~ criteria
        +List~Product~ products
    }

    class Criterion {
        +String id
        +String name
        +CriterionType type
        +String unit
        +Boolean editable
        +String description
    }

    class Product {
        +String id
        +String name
        +String brand
        +String model
        +String imageUrl
        +String source
        +Map~String, Double~ values
    }

    class AhpResult {
        +DoubleArray weights
        +Double lambdaMax
        +Double ci
        +Double ri
        +Double rc
        +Boolean consistent
    }

    class DecisionResult {
        +Product product
        +Double distanceToIdeal
        +Double distanceToAntiIdeal
        +Double closenessCoefficient
        +Int rankPosition
        +Boolean isTie
    }

    class TopsisResult {
        +Array~DoubleArray~ normalizedMatrix
        +Array~DoubleArray~ weightedMatrix
        +DoubleArray idealSolution
        +DoubleArray antiIdealSolution
        +List~DecisionResult~ ranking
        +Boolean allIdentical
    }

    Category "1" *-- "2..10" Criterion : possui
    Category "1" *-- "2..*" Product : possui
    Criterion --> CriterionType : type
    Product ..> Criterion : values usa Criterion.id como chave
    TopsisResult "1" *-- "2..*" DecisionResult : ranking
    DecisionResult --> Product : product
```

> `isTie` (RF23) marca produtos empatados (CC igual com 4 casas) e `allIdentical` marca o caso em que todos os produtos são idênticos, para o app avisar que não há como diferenciá-los. Se o empate for em 1º lugar, todos os empatados recebem o destaque de vencedor.
>
> `TopsisResult` é a proposta para devolver os valores intermediários (matriz normalizada, ponderada, A+ e A−) que a tela de **Detalhes do cálculo** precisa mostrar. Ele agrupa a lista de `DecisionResult` do ranking.

**Glossário usado no modelo**

- **Peso:** importância do critério para o usuário (sliders do AHP).
- **Característica:** dado objetivo e fixo do produto (preço, kg, bateria...), vindo do JSON.
- **Nota subjetiva:** nota de 1 a 10 de critérios com `editable = true`. O JSON traz a nota de referência e o usuário pode ajustar apenas para a avaliação atual, sem salvar.

### 2.2 Camada de dados (DTOs e conversão)

```mermaid
classDiagram
    direction LR

    class CatalogDto {
        +List~CategoryDto~ categories
    }
    class CategoryDto {
        +String id
        +String name
        +String description
        +String iconUrl
        +List~CriterionDto~ criteria
        +List~ProductDto~ products
    }
    class CriterionDto {
        +String id
        +String name
        +String type
        +String unit
        +Boolean editable
    }
    class ProductDto {
        +String id
        +String name
        +String brand
        +String model
        +String imageUrl
        +String source
        +Map~String, Double~ values
    }
    class ApiService {
        <<interface>>
        +getCatalog() CatalogDto
    }
    class DecisionRepository {
        +loadCategories() List~Category~
    }
    class Category
    class Criterion
    class Product

    CatalogDto "1" *-- "*" CategoryDto
    CategoryDto "1" *-- "*" CriterionDto
    CategoryDto "1" *-- "*" ProductDto
    ApiService ..> CatalogDto : devolve
    DecisionRepository --> ApiService : usa
    DecisionRepository ..> Category : converte e valida
    DecisionRepository ..> Criterion : converte
    DecisionRepository ..> Product : converte
```

**Validação no Repository (RN05, RN06, RN08):** ao converter cada `CategoryDto`, a categoria é descartada (não aparece no app) se tiver menos de 2 produtos, menos de 2 ou mais de 10 critérios, ou qualquer valor menor ou igual a zero. Notas subjetivas também precisam ser inteiros de 1 a 10. O `type` do JSON (`"BENEFIT"` ou `"COST"`) é convertido para o enum `CriterionType`.

### 2.3 Estado do ViewModel

```mermaid
classDiagram
    direction LR

    class LoadState {
        <<sealed>>
        Loading
        Success(categorias)
        Error(tipo, mensagem)
    }

    class LoadErrorType {
        <<enumeration>>
        NETWORK
        INVALID_DATA
    }

    class DecisionViewModel {
        +StateFlow~LoadState~ catalog
        +Category selectedCategory
        +List~Int~ sliderPositions
        +AhpResult ahpResult
        +Map adjustedScores
        +TopsisResult topsisResult
        +loadCatalog()
        +selectCategory(id)
        +setSlider(pairIndex, position)
        +setScore(productId, criterionId, score)
        +process()
        +reset()
    }

    class AhpEngine {
        +sliderToValue(k) Double
        +buildMatrix(positions, n) Array~DoubleArray~
        +calculate(matrix) AhpResult
    }

    class TopsisEngine {
        +calculate(matrix, weights, types) TopsisResult
    }

    DecisionViewModel --> LoadState
    LoadState --> LoadErrorType : tipo do erro
    DecisionViewModel --> AhpEngine
    DecisionViewModel --> TopsisEngine
```

- `sliderPositions` guarda uma posição inteira de −8 a +8 por par de critérios (n(n−1)/2 posições). Todos começam em 0 (critérios iguais).
- `adjustedScores` guarda só os ajustes do usuário nos critérios `editable`. Valores não ajustados usam a nota de referência do JSON.
- O erro tem dois tipos, porque as mensagens são diferentes: `NETWORK` (RF03: "Não foi possível carregar os dados. Verifique sua conexão com a internet.") e `INVALID_DATA` (RF24: "Os dados dos produtos estão com problema. Tente novamente mais tarde."). Ambos mostram "Tentar novamente".
- `selectCategory(id)` começa uma avaliação do zero (RF18): sliders em 0 e notas ajustadas descartadas.
- Voltar para telas anteriores não limpa nada (RF19, RNF07), porque o estado fica no ViewModel compartilhado. Só `reset()` e `selectCategory(id)` limpam.
- `reset()` limpa tudo (usado por "Nova avaliação").

### 2.4 Estados de carregamento

```mermaid
stateDiagram-v2
    [*] --> Loading : abre o app (Splash)
    Loading --> Success : JSON baixado e validado
    Loading --> Error : sem internet, tempo esgotado ou erro do servidor (RF03)
    Loading --> Error : JSON ilegível ou nenhuma categoria válida (RF24)
    Error --> Loading : "Tentar novamente"
    Success --> [*] : segue para Categorias
```

---

## 3. Navegação

```mermaid
flowchart LR
    SPL["splash<br/>Splash"]
    CAT["categories<br/>Categorias"]
    PRD["products/{categoryId}<br/>Produtos"]
    DET["product/{productId}<br/>Detalhe do produto"]
    AHPS["ahp<br/>Pesos AHP"]
    EVA["evaluation<br/>Avaliação"]
    SUM["summary<br/>Resumo"]
    RNK["ranking<br/>Ranking TOPSIS"]
    CLC["details<br/>Detalhes do cálculo"]

    SPL -- "dados carregados (~2 s)" --> CAT
    CAT -- "escolhe categoria" --> PRD
    PRD -- "toca no produto" --> DET
    DET -- "voltar" --> PRD
    PRD -- "Iniciar avaliação" --> AHPS
    AHPS -- "RC ≤ 0,10 e Avançar" --> EVA
    EVA -- "Avançar" --> SUM
    SUM -- "Processar" --> RNK
    RNK -- "Ver detalhes" --> CLC
    CLC -- "voltar" --> RNK
    RNK -- "Nova avaliação (limpa tudo)" --> CAT
    CLC -- "Nova avaliação (limpa tudo)" --> CAT
```

| Rota | Tela | Principais dados lidos do ViewModel | Ação principal |
| --- | --- | --- | --- |
| `splash` | Splash | `LoadState` | Dispara `loadCatalog()`; erro mostra "Tentar novamente" |
| `categories` | Categorias | Lista de `Category` | `selectCategory(id)`; também mostra o erro com "Tentar novamente" se o carregamento falhar (RF03, RF24) |
| `products/{categoryId}` | Produtos | `Category.products` | Abrir detalhe ou "Iniciar avaliação" |
| `product/{productId}` | Detalhe do produto | `Product` + `Criterion` | Somente leitura |
| `ahp` | Pesos AHP | `sliderPositions`, `AhpResult` | `setSlider(...)`; botão bloqueado se RC > 0,10 (RN02) |
| `evaluation` | Avaliação | Matriz de valores, critérios `editable` | `setScore(...)` (1 a 10) |
| `summary` | Resumo | Pesos + matriz de decisão | `process()` |
| `ranking` | Ranking | `TopsisResult.ranking` | "Nova avaliação" |
| `details` | Detalhes do cálculo | Matriz normalizada ponderada, A+, A−, CC | Voltar ou "Nova avaliação" |

---

## 4. Diagramas de sequência

### 4.1 Carregamento dos dados

```mermaid
sequenceDiagram
    actor U as Usuário
    participant UI as SplashScreen
    participant VM as DecisionViewModel
    participant R as DecisionRepository
    participant A as ApiService
    participant J as JSON no GitHub

    U->>UI: abre o app
    UI->>VM: loadCatalog()
    VM->>VM: estado = Loading
    VM->>R: loadCategories() em coroutine
    R->>A: getCatalog()
    A->>J: GET
    J-->>A: JSON
    A-->>R: CatalogDto
    R->>R: converte DTOs e descarta categorias inválidas
    R-->>VM: List de Category
    VM->>VM: estado = Success
    VM-->>UI: StateFlow atualizado
    UI->>U: navega para Categorias
```

### 4.2 Pesos (AHP) até o ranking (TOPSIS)

```mermaid
sequenceDiagram
    actor U as Usuário
    participant UI as Telas Compose
    participant VM as DecisionViewModel
    participant AHP as AhpEngine
    participant TOP as TopsisEngine

    U->>UI: move um slider de comparação
    UI->>VM: setSlider(par, posição)
    VM->>AHP: buildMatrix(posições, n)
    AHP-->>VM: matriz pareada recíproca
    VM->>AHP: calculate(matriz)
    AHP-->>VM: AhpResult (pesos, λmax, IC, RC)
    VM-->>UI: novo estado (pesos e RC em tempo real)

    alt RC maior que 0,10
        UI->>U: alerta e botão Avançar bloqueado
    else RC até 0,10
        U->>UI: Avançar
        UI->>U: abre Avaliação
        U->>UI: ajusta notas subjetivas (1 a 10)
        UI->>VM: setScore(produto, critério, nota)
        U->>UI: Avançar e depois Processar
        UI->>VM: process()
        VM->>TOP: calculate(matriz de decisão, pesos, tipos)
        TOP-->>VM: TopsisResult (intermediários + ranking)
        VM-->>UI: estado com resultado
        UI->>U: mostra Ranking com vencedor em destaque
    end
```

---

## 5. Contrato dos motores

| Motor | Entrada | Saída |
| --- | --- | --- |
| `AhpEngine.sliderToValue(k)` | posição inteira de −8 a +8 | `1` se k = 0; `k+1` se k > 0; `1/(|k|+1)` se k < 0 |
| `AhpEngine.buildMatrix(posições, n)` | posições dos n(n−1)/2 sliders | matriz n×n recíproca com diagonal 1 (RN01) |
| `AhpEngine.calculate(matriz)` | matriz pareada | `AhpResult`: pesos (média geométrica normalizada, somam 1), λmax, IC, IR, RC, `consistent` |
| `TopsisEngine.calculate(matriz, pesos, tipos)` | matriz de decisão m×n, pesos, `BENEFIT`/`COST` por coluna | `TopsisResult`: normalizada, ponderada, A+, A−, d+, d−, CC, posição |

**Casos de borda tratados no TOPSIS (RN07):** coluna com norma zero gera `r = 0`; se d+ + d− = 0 (produtos idênticos), CC = 0 para todos e o app mostra empate; CC igual com 4 casas decimais gera empate com a mesma posição (ex.: 1º, 2º, 2º, 4º).

---

## 6. Decisões de modelagem

| Decisão                                   | Motivo                                                                          |
| ----------------------------------------- | ------------------------------------------------------------------------------- |
| Modelo genérico com `Map<String, Double>` | Critérios e produtos vêm do JSON; o código não depende de critérios específicos |
| Motores em Kotlin puro                    | Testáveis sem emulador                                                          |
| Um único ViewModel compartilhado          | Evita perder pesos e notas ao navegar entre telas                               |
| Estados `Loading`, `Success`, `Error`     | Splash e telas reagem a carregamento e falha de rede de forma uniforme          |
| Ajustes de nota não persistidos           | Valem só para a avaliação atual; "Nova avaliação" limpa tudo                    |
