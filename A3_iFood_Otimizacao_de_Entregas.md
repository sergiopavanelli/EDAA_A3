# A3 — Estudo de Caso Aplicado

---

**Centro Universitário de Belo Horizonte — UNIBH**

**Unidade Curricular:** Estruturas de Dados e Análise de Algoritmos
**Código:** 0006963
**Modalidade:** Presencial – Prática (E2A)
**Semestre:** 2026/1

**Tema:** iFood e Otimização de Entregas

**Professor:** Alexandre de Oliveira

**Integrantes do grupo:**

| Nome | RA |
|---|---|
| Sérgio Pinton Pavanelli | 123220202 |
| Júlia Starling Negrini Fudoli | 124222027 |

**Data de entrega:** 08/06/2026

---

## Sumário

1. [Escolha e Apresentação do Caso](#1-escolha-e-apresentação-do-caso)
2. [Raio-X Técnico do Caso](#2-raio-x-técnico-do-caso)
   - 2a. [Arquitetura e Stack Tecnológica](#2a-arquitetura-e-stack-tecnológica)
   - 2b. [Problema Computacional Envolvido](#2b-problema-computacional-envolvido)
   - 2c. [Classificação Conceitual do Problema](#2c-classificação-conceitual-do-problema)
   - 2d. [Estratégia Utilizada pela Empresa](#2d-estratégia-utilizada-pela-empresa)
3. [Proposta Alternativa do Grupo](#3-proposta-alternativa-do-grupo)
4. [Análise Crítica](#4-análise-crítica)
5. [Conclusão](#5-conclusão)
6. [Referências](#referências)

---

## 1. Escolha e Apresentação do Caso

### 1.1 A empresa

O iFood é a maior plataforma de *food delivery* da América Latina. Fundado em 2011, atualmente conecta restaurantes, entregadores e consumidores em mais de **1.000 cidades brasileiras**. Seus números de operação evidenciam o nível de escala envolvido:

| Indicador | Valor |
|---|---|
| Pedidos processados por mês | ~39 milhões |
| Restaurantes parceiros | ~220 mil |
| Entregadores cadastrados | ~170 mil |
| Cidades atendidas | +1.000 |

*Fonte: AWS Case Study – iFood AI (2019/2020).*

Pertencente ao grupo Prosus desde 2018, o iFood é hoje um dos principais cases de tecnologia do Brasil, com um time de engenharia de centenas de pessoas e investimento declarado de **US$ 20 milhões** apenas na criação de sua Academia de Inteligência Artificial.

### 1.2 O problema enfrentado

Quando um usuário faz um pedido pelo aplicativo, uma sequência de decisões computacionais precisa ocorrer em **segundos**: qual entregador acionar, como agrupá-lo com outros pedidos próximos, qual rota seguir, e quando notificar cada parte. Em escala de dezenas de milhões de pedidos mensais, o que parece simples no cotidiano — "alguém vai buscar meu lanche" — é, na verdade, um conjunto de **problemas de otimização combinatória** que desafiam as fronteiras da computabilidade prática.

Antes de investir pesadamente em algoritmos e IA, o iFood convivia com dois sintomas claros:
- **SLA de entrega de apenas 80%**: um em cada cinco pedidos chegava fora do prazo estimado.
- **Entregadores com 50% do tempo ocioso**: a distribuição ineficiente de demanda deixava parte da frota parada enquanto outra ficava sobrecarregada.

Esses problemas tinham impacto direto no negócio: insatisfação do cliente, cancelamentos, abandono da plataforma e perda de renda dos entregadores. A solução exigiu uma combinação de arquitetura tecnológica robusta e algoritmos de otimização sofisticados.

> **Análise do grupo:** O ponto mais revelador desse contexto é que o problema não era de falta de dados ou de infraestrutura, mas sim de **eficiência algorítmica no uso desses dados em tempo real**. Ter 170 mil entregadores registrados não resolve nada se o sistema não souber alocar o entregador certo, para o pedido certo, no momento certo — e essa alocação precisa acontecer em milissegundos, continuamente.

---

## 2. Raio-X Técnico do Caso

### 2a. Arquitetura e Stack Tecnológica

#### Evolução arquitetural

O iFood iniciou sua operação com uma arquitetura monolítica. Com o crescimento explosivo da plataforma, essa abordagem tornou-se um gargalo: qualquer pico de demanda (datas comemorativas, promoções) comprometia o sistema inteiro. Em **2016**, a empresa realizou a migração para uma **arquitetura de microsserviços orientada a eventos** (Event-Driven Architecture) hospedada na **AWS**.

Essa migração foi a base que viabilizou tudo que veio depois: cada domínio do negócio (pagamentos, logística, catálogo, recomendações) passou a ser um serviço independente, escalável separadamente.

#### Stack tecnológica

```
+-------------------+       +--------------------+       +------------------+
| Camada de         |       | Camada de          |       | Camada de        |
| Comunicação       |       | Processamento      |       | Persistência     |
|                   |       |                    |       |                  |
| Apache Kafka      |  -->  | Microsserviços     |  -->  | PostgreSQL       |
| Amazon SQS        |       | (Java/Kotlin/Go/   |       | Amazon DynamoDB  |
| Amazon Kinesis    |       | Python/Scala)      |       | Redis (cache)    |
+-------------------+       +--------------------+       +------------------+
                                     |
                            +--------------------+
                            | Camada de ML/IA    |
                            |                    |
                            | Amazon SageMaker   |
                            | Amazon EKS         |
                            +--------------------+
```

| Categoria | Tecnologia |
|---|---|
| **Cloud** | AWS (principal) |
| **Linguagens** | Java, Kotlin, Go, Python, Scala |
| **Mensageria/Filas** | Apache Kafka, Amazon SQS |
| **Streaming em tempo real** | Amazon Kinesis |
| **Banco relacional** | PostgreSQL |
| **Banco NoSQL** | Amazon DynamoDB |
| **Cache in-memory** | Redis |
| **Treinamento de modelos ML** | Amazon SageMaker |
| **Deploy de modelos** | Amazon EKS (Kubernetes) |
| **Armazenamento de features** | Amazon S3 |
| **Ferramenta interna de CI/CD** | "Tompero" (CLI proprietária) |

*Fontes: AWS Blog Brasil – arquitetura de eventos iFood; AWS Case Study iFood AI; iFood Tech Medium.*

#### Por que essa stack importa para o problema de entregas?

A separação entre **Kafka** (eventos assíncronos de alta vazão), **Kinesis** (streaming em tempo real) e **Redis** (cache ultra-rápido) é essencial: o sistema de matching de pedidos precisa consultar dados fresquíssimos sobre localização dos entregadores, status dos restaurantes e situação do trânsito — Redis serve esses dados em microsegundos, sem necessidade de consulta ao banco de dados principal.

### 2b. Problema Computacional Envolvido

#### O problema central: despacho e roteamento em escala

O coração do iFood é o **sistema de despacho**: dado um novo pedido, encontrar o melhor entregador e a melhor rota de coleta e entrega. Parece simples, mas envolve múltiplas decisões simultâneas e interdependentes:

1. **Qual entregador acionar?** — considerando distância ao restaurante, tempo estimado de preparo do prato, avaliação histórica do entregador, taxa de aceitação e se já está fazendo outra entrega.
2. **Agrupar pedidos?** — se um entregador pode pegar dois pedidos do mesmo restaurante (ou restaurantes próximos) para uma mesma região, isso reduz custos e melhora o SLA.
3. **Qual rota seguir?** — considerando trânsito em tempo real, condições climáticas, tipo de veículo e janelas de tempo de entrega prometidas.

#### Por que é difícil?

O problema de roteamento de veículos (**VRP — Vehicle Routing Problem**) e seu caso especial, o problema do caixeiro-viajante (**TSP — Travelling Salesman Problem**), são os problemas fundamentais aqui. A dificuldade emerge de quatro fatores combinados:

**1. Explosão combinatória:** Para `n` pontos de entrega, o número de rotas possíveis é da ordem de `n!` (fatorial de n). Com apenas 20 pedidos simultâneos em uma região, o número de combinações supera 2,4 × 10¹⁸ — inviável para qualquer busca exaustiva.

**2. Dinamismo em tempo real:** Novos pedidos chegam continuamente; entregadores entram e saem da plataforma; restaurantes têm atrasos; trânsito muda. A solução calculada há 30 segundos pode já estar desatualizada.

**3. Restrições múltiplas e heterogêneas:** Capacidade do veículo, janelas de tempo, prioridade do cliente, tipo de comida (ex.: sorvete vs. pizza), raio máximo de deslocamento do entregador — cada restrição adiciona uma dimensão ao espaço de busca.

**4. Escala geográfica e temporal:** 39 milhões de pedidos por mês, em 1.000+ cidades, com sazonalidade brutal (horário de almoço, finais de semana, dia do jogo do Brasil).

| Fator de dificuldade | Manifestação no iFood |
|---|---|
| Complexidade combinatória | n! rotas possíveis; NP-difícil |
| Escalabilidade | 39M pedidos/mês, 170K entregadores simultâneos |
| Crescimento de dados | Histórico de entregas, perfis, mapas, tráfego em tempo real |
| Dinamismo | Cancelamentos, atrasos, entregadores entrando/saindo |
| Restrições práticas | Tipo de veículo, tempo de preparo, janelas de entrega |

### 2c. Classificação Conceitual do Problema

O problema central do iFood pertence à categoria de **problemas de otimização combinatória de roteamento**, mais especificamente ao domínio **NP-difícil**.

#### TSP e VRP: os problemas clássicos

O **Travelling Salesman Problem (TSP)** pede a rota mais curta que visita `n` cidades exatamente uma vez e retorna à origem. O **Vehicle Routing Problem (VRP)** é uma generalização: múltiplos veículos, depósitos de origem, restrições de capacidade e janelas de tempo — exatamente o cenário do iFood.

Ambos são **NP-difíceis**: não existe algoritmo conhecido que os resolva em tempo polinomial no pior caso. Isso significa que, à medida que `n` cresce, o tempo computacional cresce de forma tão explosiva que a solução ótima se torna **computacionalmente inviável** mesmo com hardware moderno.

#### Por que não é possível resolver de forma exata?

| Tamanho do problema | Tempo para força bruta (O(n!)) |
|---|---|
| n = 10 pedidos | ~3,6 milhões de combinações |
| n = 20 pedidos | ~2,4 × 10¹⁸ combinações |
| n = 30 pedidos | ~2,6 × 10³² combinações |
| n = 100 pedidos | Inviável para qualquer computador |

Com 39 milhões de pedidos por mês e picos de milhares de pedidos simultâneos por região, não há poder computacional que resolva isso de forma exata em tempo real. **A solução ótima teórica é inviável na prática; o que se busca é a melhor solução viável no menor tempo possível.**

> **Análise do grupo:** Aqui reside um dos ensinamentos mais importantes da disciplina: a classificação de um problema como NP-difícil não é um obstáculo insuperável na prática — é um sinal de que a abordagem precisa mudar. Empresas como o iFood não "resolvem" o TSP/VRP; elas o **aproximam** com heurísticas inteligentes que encontram soluções boas o suficiente em tempo aceitável. A distinção entre ótimo teórico e viável prático é central em engenharia de software de alto desempenho.

O problema do iFood se enquadra, portanto, como:
- **Classe:** NP-difícil (VRP com múltiplas restrições)
- **Domínio:** Otimização + Roteamento + Escalonamento
- **Variante de decisão:** NP-Completo (ex.: "existe rota com custo ≤ k?")

### 2d. Estratégia Utilizada pela Empresa

O iFood adota uma abordagem em camadas, combinando múltiplas estratégias computacionais:

#### Camada 1 — Estruturação geoespacial e agrupamento (matching)

O primeiro passo é identificar quais entregadores e pedidos fazem sentido serem considerados juntos. O sistema:
- **Particiona o espaço geográfico** em células (clusters por região urbana).
- **Agrupa pedidos próximos** (batching): um entregador pode ser acionado para dois ou mais pedidos no mesmo cluster, desde que os restaurantes e os destinos sejam compatíveis em termos de distância e tempo.
- O critério de seleção do entregador considera: distância ao restaurante, tempo estimado de preparo, avaliação histórica, taxa de aceitação e carga atual.

Esse agrupamento é, em si, uma heurística gulosa: pega-se o melhor candidato disponível a cada momento, sem garantia de otimalidade global — mas com velocidade suficiente para operar em tempo real.

#### Camada 2 — Cálculo de rota e heurísticas/metaheurísticas

Uma vez definido o lote de pedidos para um entregador, o sistema calcula a sequência de coleta e entrega mais eficiente. Conforme documentado pelo próprio iFood, são utilizados:

- **Algoritmos heurísticos**: produzem soluções de qualidade satisfatória em tempo polinomial (ex.: vizinho mais próximo, heurística de inserção mais barata).
- **Algoritmos genéticos**: metaheurística bioinspirada que mantém uma "população" de soluções e as evolui por operações de cruzamento e mutação, iterativamente melhorando a rota ao longo de gerações.

O resultado declarado pela empresa é uma redução de até **23,35% nos quilômetros rodados** pelos entregadores em relação à rota calculada sem otimização.

#### Camada 3 — Machine Learning para previsão e ETA

Além da rota em si, o iFood aplica modelos de ML para:

- **Previsão de demanda**: com base em histórico de pedidos, localização, sazonalidade e padrões de consumo, o sistema antecipa regiões de alta demanda e posiciona entregadores proativamente — reduzindo o tempo ocioso em **50%**.
- **Estimativa de tempo de entrega (ETA)**: modelos treinados no Amazon SageMaker preveem o tempo de preparo do restaurante, tempo de deslocamento (considerando trânsito via Kinesis em tempo real) e entregam ao usuário uma estimativa confiável.
- **Ajustes dinâmicos**: se um cancelamento ocorre ou um restaurante atrasa, o sistema recalcula em tempo real.

#### Camada 4 — Infraestrutura de suporte

Todo esse processamento é sustentado por:

- **Redis**: cache de dados de localização dos entregadores, status dos restaurantes e resultados de cálculos recentes — evita recalcular o que já foi processado.
- **Kafka/Kinesis**: eventos de pedido novo, atualização de localização e cancelamento fluem como streams, permitindo reatividade em tempo real sem polling.
- **Kubernetes (EKS)**: os modelos de ML são servidos como microsserviços independentes, com escalabilidade horizontal automática nos picos.

| Estratégia | Onde é aplicada no iFood |
|---|---|
| Algoritmo guloso | Seleção do entregador no matching |
| Heurísticas | Cálculo inicial da rota de entrega |
| Algoritmo genético | Refinamento de rotas (metaheurística) |
| Clustering | Agrupamento de pedidos por região (batching) |
| Machine Learning | Previsão de demanda, ETA |
| Cache (Redis) | Dados de localização e status em tempo real |
| Processamento em fila (Kafka) | Eventos de pedido, cancelamento, localização |
| Paralelismo (Kubernetes/EKS) | Escalabilidade horizontal dos modelos de ML |
| Indexação geoespacial | Particionamento do mapa em células/clusters |

---

## 3. Proposta Alternativa do Grupo

### 3.1 Contextualização da proposta

A abordagem atual do iFood com algoritmos genéticos e heurísticas clássicas é eficaz, mas apresenta uma limitação estrutural: os algoritmos genéticos operam sobre o problema de roteamento **depois** que os lotes já foram formados de maneira relativamente simples (batching guloso). Propomos um pipeline de dois estágios que torna o **processo de formação de lotes mais inteligente** antes de rodar a metaheurística de refinamento, potencialmente obtendo soluções de melhor qualidade global com menor custo computacional.

### 3.2 Pipeline proposto: Clustering Geoespacial + Busca Tabu

O pipeline é composto de dois estágios sequenciais:

```
Pedidos novos (janela de 30s)
         |
         v
+-------------------------+
| ESTÁGIO 1               |
| Clustering geoespacial  |
| com índice H3/Geohash   |
|                         |
| - Mapa dividido em      |
|   células hexagonais    |
|   (H3 nível 7-9)        |
| - Pedidos na mesma      |
|   célula ou células     |
|   adjacentes formam     |
|   candidatos a lote     |
| - Heurística gulosa:    |
|   compatibilidade de    |
|   janela de tempo       |
+-------------------------+
         |
         v (lotes candidatos)
+-------------------------+
| ESTÁGIO 2               |
| Busca Tabu por lote     |
|                         |
| - Para cada lote:       |
|   permuta sequência de  |
|   coleta/entrega        |
| - Lista tabu evita      |
|   revisitar soluções   |
|   recentes              |
| - Critério de parada:   |
|   max iterações ou      |
|   ganho < threshold     |
+-------------------------+
         |
         v
Lote finalizado → matching com entregador disponível
```

#### Estágio 1: Clustering com índice H3/Geohash

**O que é:** O índice H3 (desenvolvido pelo Uber e de uso público) divide o mapa em células hexagonais hierárquicas. Cada pedido e cada entregador pode ser mapeado para uma célula H3 em O(1). Pedidos na mesma célula ou em células adjacentes (1 hop) são naturalmente candidatos a batching.

**Vantagem em relação ao batching guloso atual:** Em vez de agrupar pedidos simplesmente pelos mais próximos no momento (guloso), o H3 permite:
- Criar lotes com compatibilidade de **janela de tempo** (pedidos com prazos similares ficam juntos).
- Controlar o **tamanho máximo do lote** por célula (evitar super-aglomeração).
- Recalcular clusters em O(k) onde k é o número de pedidos na janela, não em função do mapa inteiro.

A complexidade desse estágio é **O(k log k)** (ordenação + consulta de vizinhança no índice), muito inferior ao custo de um VRP exato.

#### Estágio 2: Busca Tabu para refinamento de rota

**O que é:** A Busca Tabu (Tabu Search) é uma metaheurística de melhoria local que:
1. Parte de uma solução inicial (ex.: rota gerada pela heurística do vizinho mais próximo).
2. Explora vizinhanças da solução atual (trocas 2-opt, 3-opt: inverte segmentos da rota).
3. Mantém uma **lista tabu** de movimentos recentemente realizados, impedindo revisitar soluções — o que a diferencia de uma simples descida de gradiente e permite escapar de ótimos locais.
4. Para após um número fixo de iterações ou quando o ganho é marginal.

**Por que Busca Tabu e não Algoritmo Genético (estratégia atual)?**

| Critério | Algoritmo Genético (atual) | Busca Tabu (proposto) |
|---|---|---|
| Natureza | Baseado em população | Baseado em trajetória |
| Overhead de memória | Alto (mantém população) | Baixo (mantém lista tabu) |
| Velocidade de convergência | Mais lento (muitas gerações) | Mais rápido por instância pequena |
| Aplicação ideal | Problemas de grande dimensão, exploração ampla | Instâncias menores, refinamento de qualidade |
| Paralelização | Natural (indivíduos independentes) | Possível por cluster independente |

Para instâncias de tamanho pequeno a médio — como os lotes formados por cluster (tipicamente 2 a 6 pedidos) — a Busca Tabu converge mais rapidamente e com overhead de memória menor do que manter uma população genética. O Algoritmo Genético continua sendo superior para exploração de espaços de solução muito grandes, mas o pré-agrupamento por H3 já reduziu drasticamente o tamanho do problema a ser resolvido por cada instância.

### 3.3 Vantagens da proposta

1. **Redução do espaço de busca**: O pré-agrupamento por H3 transforma um problema VRP global (n pedidos, m entregadores) em vários subproblemas locais (tipicamente 2–6 pedidos por cluster), cada um resolvível em milissegundos com Busca Tabu.
2. **Garantia de consistência temporal**: A compatibilidade de janela de tempo é garantida na fase de clustering, não remediada na fase de roteamento.
3. **Escalabilidade horizontal**: Cada cluster é um subproblema independente — os vários lotes podem ser otimizados em paralelo por diferentes instâncias do serviço de roteamento (já hospedado em EKS/Kubernetes).
4. **Melhor controle de qualidade**: A lista tabu garante diversificação da busca, reduzindo o risco de ficar preso em ótimos locais ruins.

### 3.4 Limitações da proposta

1. **Granularidade do índice H3**: A escolha do nível hierárquico (resolução) do H3 é crítica. Resolução alta demais gera lotes muito pequenos (sub-aproveitamento do batching); baixa demais agrupa pedidos distantes demais. É necessário calibração por cidade/região.
2. **Lotes de borda**: Pedidos em fronteiras de células podem ser negligenciados. É necessário lógica para lidar com pedidos em células vizinhas com lotes já formados.
3. **Dinamismo ainda é um desafio**: A Busca Tabu é uma otimização offline por janela. Cancelamentos e novos pedidos após o fechamento do lote exigem recalcular, o que pode gerar instabilidade em períodos de alta volatilidade.
4. **Não implementado**: Como toda proposta conceitual neste trabalho, a eficácia exata dependeria de experimentos com dados reais de produção do iFood — que não são públicos.

---

## 4. Análise Crítica

### 4.1 Trade-offs: solução ótima vs. solução viável

O maior trade-off de todo o sistema do iFood pode ser resumido em uma sentença: **velocidade sobre otimalidade**. Um solver exato de VRP poderia, em teoria, encontrar a rota matematicamente perfeita para qualquer conjunto de pedidos — mas levaria horas ou dias para instâncias reais. Uma heurística gulosa resolve em milissegundos, mas a rota pode estar 10-20% acima do ótimo.

O iFood escolheu pragmaticamente o meio-termo: **heurísticas rápidas complementadas por metaheurísticas de refinamento**. Os resultados (SLA de 95%, −12% de distância) mostram que esse trade-off é eficaz na prática — o que vai ao encontro da observação do professor: *"nem sempre a melhor solução teórica é a melhor solução prática"*.

### 4.2 Custo computacional

| Estratégia | Complexidade | Viabilidade em tempo real |
|---|---|---|
| Força bruta (TSP exato) | O(n!) | Inviável para n > 15 |
| Programação dinâmica (Held-Karp) | O(2ⁿ · n²) | Inviável para n > 25 |
| Heurística gulosa (vizinho mais próximo) | O(n²) | Viável |
| Algoritmo genético | O(g · p · n) | Viável com controle de gerações |
| Busca Tabu | O(i · n²) | Viável por cluster pequeno |

*n = pontos de entrega; g = gerações; p = tamanho da população; i = iterações.*

A complexidade das abordagens heurísticas e metaheurísticas é **polinomial** com parâmetros controláveis (número de gerações, de iterações, tamanho da população). Isso é o que as torna viáveis em produção.

### 4.3 Impacto da escalabilidade

O crescimento do iFood impõe pressão constante: cada novo restaurante parceiro, cada nova cidade, cada novo entregador cadastrado amplia o espaço do problema. Uma solução que funciona para 10 pedidos simultâneos por região pode degrada para 100.

A arquitetura de microsserviços + Kubernetes mitiga isso horizontalmente: mais instâncias do serviço de roteamento são provisionadas automaticamente nos picos. Mas isso não resolve o problema algorítmico — apenas distribui a carga. Por isso, manter algoritmos com complexidade controlada (polinomial) é uma exigência estrutural, não uma escolha estética.

> **Análise do grupo:** A escalabilidade revela um paradoxo interessante: quanto mais o iFood cresce (mais pedidos, mais entregadores, mais cidades), mais dados tem para treinar modelos de ML e mais precisa fica a previsão de demanda. Mas também mais complexo fica o problema de despacho. O crescimento simultâneo de dados e complexidade cria um ciclo que exige reinvestimento constante em melhorias algorítmicas — o que explica o investimento de US$ 20 mi em 2019 e a existência de uma academia de IA interna.

### 4.4 Limitações práticas

**Fairness com entregadores:** Algoritmos puramente ótimos tendem a concentrar pedidos nos entregadores mais bem avaliados e mais bem posicionados, criando desigualdade de renda. O iFood precisa balancear eficiência logística com equidade na distribuição de pedidos — o que é um objetivo que não aparece no modelo matemático do VRP clássico.

**Cancelamentos e volatilidade:** Um pedido cancelado após o despacho desperdicia o percurso até o restaurante. A previsão de probabilidade de cancelamento (baseada em histórico do restaurante e do cliente) poderia ser incorporada ao critério de despacho, mas adiciona complexidade.

**Dependência de dados de terceiros:** A otimização de rotas depende da qualidade dos dados de tráfego (geralmente vindos de APIs do Google Maps ou HERE Maps). Falhas ou imprecisões nessas APIs afetam diretamente a qualidade das rotas calculadas.

**Restaurantes como gargalo imprevisível:** O tempo de preparo do restaurante é o fator mais variável e menos controlável do sistema. Mesmo a melhor rota calculada falha se o restaurante atrasar 20 minutos na produção do prato. O modelo de ML de ETA precisa aprender a incorporar essa incerteza.

---

## 5. Conclusão

O caso do iFood é um exemplo paradigmático de como problemas clássicos da ciência da computação — que parecem abstratos no contexto acadêmico — se manifestam concretamente em sistemas que milhões de pessoas usam diariamente.

O problema de otimizar as entregas do iFood é, em sua essência, uma instância do Vehicle Routing Problem, pertencente à classe NP-difícil. Isso significa que a solução exata é computacionalmente inviável em escala real. A resposta da empresa foi a mesma adotada pela engenharia de software moderna: **não buscar o ótimo perfeito, mas construir sistemas que encontrem soluções boas o suficiente, rápido o suficiente, a custo aceitável**.

As estratégias combinadas — heurísticas de matching, algoritmos genéticos para roteamento, ML para previsão de demanda e ETA, cache e streaming em tempo real — geraram resultados concretos e mensuráveis: SLA de 80% para 95%, 12% menos quilômetros rodados, 50% menos tempo ocioso. Esses números não são fruto de hardware mais potente, mas de **algoritmos mais inteligentes**.

Nossa proposta alternativa — clustering geoespacial com H3 na formação de lotes seguido de Busca Tabu no refinamento da rota — não é necessariamente superior à abordagem atual do iFood (que opera com dados de produção que não temos acesso). Ela é, contudo, uma alternativa fundamentada em princípios algorítmicos sólidos, com justificativa clara de trade-offs e alinhada com as práticas da indústria (o índice H3 foi criado e publicado pelo próprio Uber para problemas análogos).

O aprendizado central deste estudo é que, em sistemas de grande escala, a escolha algorítmica **é** uma decisão de negócio. Escolher entre algoritmo exato e heurística, entre batch e tempo real, entre exploração e explotação no espaço de soluções — essas são escolhas que impactam diretamente receita, custo operacional e experiência do usuário.

---

## Referências

### Fontes oficiais do iFood

1. **iFood**. *Com tecnologia AWS, iFood implementa área de IA para aprimorar experiência de clientes e restaurantes*. AWS Case Study. Disponível em: <https://aws.amazon.com/solutions/case-studies/ifoodai/>. Acesso em: jun. 2026.

2. **iFood Institucional**. *Otimização de rotas com planejador em 2025*. Disponível em: <https://institucional.ifood.com.br/inovacao/otimizacao-de-rotas-a-ia-na-distribuicao-de-alimentos/>. Acesso em: jun. 2026.

3. **iFood Institucional**. *Como foi o desenvolvimento de software para logística no iFood*. Disponível em: <https://institucional.ifood.com.br/inovacao/desenvolvimento-de-software-logistica-ifood/>. Acesso em: jun. 2026.

4. **iFood Institucional**. *Plataforma de aprendizado de máquina*. Disponível em: <https://institucional.ifood.com.br/inovacao/plataforma-de-aprendizado-de-maquina/>. Acesso em: jun. 2026.

5. **CAPELEIRO, Thiago**. *Os bastidores do seu pedido no iFood*. iFood Tech — Medium. Disponível em: <https://medium.com/ifood-tech/os-bastidores-do-seu-pedido-no-ifood-e351c50ef841>. Acesso em: jun. 2026.

### Fontes técnicas sobre a arquitetura (AWS)

6. **AWS Blog Brasil**. *Como iFood se beneficiou da arquitetura orientada a eventos para modernizar seu middleware financeiro*. Disponível em: <https://aws.amazon.com/pt/blogs/aws-brasil/como-ifood-se-beneficiou-da-arquitetura-orientada-a-eventos-para-modernizar-seu-midware-financeiro/>. Acesso em: jun. 2026.

7. **AWS Blog Brasil**. *Como iFood desenvolveu sua nova plataforma de recomendações*. Disponível em: <https://aws.amazon.com/pt/blogs/aws-brasil/como-ifood-desenvolveu-sua-nova-plataforma-de-recomendacoes/>. Acesso em: jun. 2026.

### Referências acadêmicas e técnicas — TSP/VRP

8. **DANTZIG, G. B.; RAMSER, J. H**. The Truck Dispatching Problem. *Management Science*, v. 6, n. 1, p. 80–91, 1959. (Artigo fundador do VRP.)

9. **DANTZIG, G. B.; FULKERSON, D. R.; JOHNSON, S. M**. Solution of a Large-Scale Traveling-Salesman Problem. *Journal of the Operations Research Society of America*, v. 2, n. 4, p. 393–410, 1954.

10. **GAREY, M. R.; JOHNSON, D. S**. *Computers and Intractability: A Guide to the Theory of NP-Completeness*. San Francisco: W. H. Freeman, 1979. (Referência clássica para NP-completude e NP-dificuldade.)

11. **CHRISTOFIDES, N**. Worst-case analysis of a new heuristic for the travelling salesman problem. *Report 388*, Graduate School of Industrial Administration, Carnegie Mellon University, 1976. (Algoritmo de aproximação com garantia de 3/2 do ótimo.)

12. **TOTH, P.; VIGO, D**. (Eds.). *The Vehicle Routing Problem*. Philadelphia: SIAM Monographs on Discrete Mathematics and Applications, 2002.

13. **GLOVER, F**. Tabu Search — Part I. *ORSA Journal on Computing*, v. 1, n. 3, p. 190–206, 1989. (Artigo original da Busca Tabu.)

### Referências sobre indústria e comparativos

14. **DoorDash Engineering**. *Using ML and Optimization to Solve DoorDash's Dispatch Problem*. Disponível em: <https://careersatdoordash.com/blog/using-ml-and-optimization-to-solve-doordashs-dispatch-problem/>. Acesso em: jun. 2026.

15. **UBER Engineering**. *H3: Uber's Hexagonal Hierarchical Spatial Index*. Disponível em: <https://www.uber.com/blog/h3/>. 2018.

16. **LI, M. et al**. An effective matching algorithm with adaptive tie-breaking strategy for online food delivery problem. *Complex & Intelligent Systems*, Springer, 2021. DOI: 10.1007/s40747-021-00340-x.

17. **ROARBIT**. *Algoritmos de Roteirização e Logística: como apps de delivery otimizam as rotas*. Disponível em: <https://blog.roarbit.com.br/algoritmos-de-roteirizacao-e-logistica-como-apps-de-delivery-otimizam-as-rotas/>. Acesso em: jun. 2026.

---

*Documento elaborado pelo grupo para a atividade A3 da UC Estruturas de Dados e Análise de Algoritmos (UNIBH, 2026/1). Inteligência artificial foi utilizada como apoio à pesquisa e estruturação; a análise crítica, a proposta alternativa e as conclusões são de autoria dos integrantes do grupo.*
