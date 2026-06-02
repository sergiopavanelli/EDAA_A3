# 📚 A3 — Estudo de Caso Aplicado

> **Disciplina:** Estruturas de Dados e Análise de Algoritmos
> **Instituição:** Centro Universitário de Belo Horizonte — UNIBH
> **Semestre:** 2026/1 · Modalidade: Presencial – Prática (E2A)

---

## 🎯 Tema

### iFood e Otimização de Entregas

Como uma das maiores plataformas de *food delivery* do mundo resolve, em tempo real e a cada segundo, um problema que a teoria da computação classifica como **NP-difícil**.

---

## 👥 Integrantes

| 👤 Nome | 🎓 RA |
|---|---|
| Sérgio Pinton Pavanelli | 123220202 |
| Júlia Starling Negrini Fudoli | 124222027 |

---

## 📁 Arquivos do repositório

| Arquivo | Descrição |
|---|---|
| 📄 `A3_iFood_Otimizacao_de_Entregas.md` | Documento completo do trabalho |
| 📋 `A3_20260601161026.pdf` | Enunciado original da atividade |

> Os slides de apresentação serão adicionados separadamente.

---

## 🗂️ Estrutura do trabalho

```
A3 — Estudo de Caso
│
├── 1️⃣  Apresentação do Caso ........... O que é o iFood e qual era o problema
├── 2️⃣  Raio-X Técnico
│   ├── 2a  Stack & Arquitetura ........ AWS, Kafka, Redis, SageMaker, microsserviços
│   ├── 2b  Problema Computacional ..... VRP/TSP, explosão combinatória, dinamismo
│   ├── 2c  Classificação Conceitual ... NP-difícil, otimização, roteamento
│   └── 2d  Estratégia da Empresa ...... Heurísticas, algoritmos genéticos, ML, cache
├── 3️⃣  Proposta Alternativa ........... Clustering H3 + Busca Tabu (do grupo)
├── 4️⃣  Análise Crítica ................ Trade-offs, escalabilidade, limitações
└── 5️⃣  Conclusão & Referências
```

---

## 🧠 Conceitos abordados

| Conceito | Onde aparece |
|---|---|
| 🔢 Complexidade O(n!) | Força bruta no TSP/VRP — inviável em produção |
| ⚡ Heurísticas | Matching guloso de entregadores |
| 🧬 Algoritmos Genéticos | Refinamento de rotas pelo iFood |
| 🗺️ VRP / TSP | Problema central de despacho e roteamento |
| 🏷️ Classificação NP-difícil | Fundamentação teórica do caso |
| 🗃️ Cache (Redis) | Dados de localização em tempo real |
| 📨 Filas (Kafka/SQS) | Eventos assíncronos de pedidos |
| 🤖 Machine Learning | Previsão de demanda e ETA |
| 🔷 Clustering Geoespacial (H3) | Proposta alternativa do grupo |
| 🔁 Busca Tabu | Metaheurística na proposta do grupo |

---

## 📊 Resultados reais do iFood com otimização algorítmica

| Métrica | Antes | Depois |
|---|---|---|
| SLA de entrega no prazo | 80% | **95%** |
| Distância percorrida pelos entregadores | base | **−12%** |
| Tempo ocioso dos entregadores | base | **−50%** |

*Fonte: AWS Case Study – iFood AI.*

---

## 📅 Entrega

**08 de junho de 2026**
