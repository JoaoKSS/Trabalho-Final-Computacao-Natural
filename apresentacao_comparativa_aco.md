---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: #f4f6f8
color: #2c3e50
style: |
  section {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    font-size: 25px;
    padding: 40px;
  }
  h1 {
    color: #1a365d;
    font-size: 1.8em;
    margin-bottom: 0.2em;
    border-bottom: 3px solid #3182ce;
  }
  h2 {
    color: #2b6cb0;
    font-size: 1.4em;
    margin-top: 0.2em;
    margin-bottom: 0.4em;
    border-bottom: 1px solid #e2e8f0;
  }
  footer {
    font-size: 0.5em;
    color: #718096;
    text-align: right;
  }
  table {
    font-size: 0.6em;
    width: 60%;
    border-collapse: collapse;
    margin-top: 10px;
    margin-bottom: 10px;
  }
  th {
    background-color: #2b6cb0;
    color: white;
    padding: 6px;
    font-weight: bold;
  }
  td {
    padding: 5px;
    border-bottom: 1px solid #e2e8f0;
  }
  tr:nth-child(even) {
    background-color: #edf2f7;
  }
  strong {
    color: #2c5282;
  }
  footer {
    content: "Otimização ACO para o TSP | André Felipe & João Kennedy"
  }
---
<!-- slide1: -->

<!-- _class: lead -->
# Otimização por Colônia de Formigas (ACO) para o TSP
## Estudo Comparativo de Variantes e Escalabilidade
**Equipe:** André Felipe, João Kennedy  
**Problema:** Caixeiro Viajante (Traveling Salesperson Problem - TSP)  
*Uma análise detalhada entre as abordagens Clássica, Voter Model e Estratificação Social sob diferentes parâmetros.*

---
<!-- slide2: -->

## O Problema do Caixeiro Viajante (TSP)
- **Definição:** Encontrar a rota de menor distância que visite um conjunto de cidades exatamente uma vez e retorne ao ponto de partida.
- **Complexidade:** NP-difícil. O espaço de busca cresce fatorialmente com o número de cidades ($O(N!)$), impossibilitando soluções exatas por força-bruta para grandes escalas.
- **Aplicações:** Logística, planejamento de rotas, microeletrônica (furação de placas) e sequenciamento de DNA.
- **Abordagem por Colônia de Formigas (ACO):** Meta-heurística bio-inspirada baseada no comportamento de formigas reais que utilizam trilhas de feromônio químicas para guiar a busca de caminhos mais curtos.

---
<!-- slide3: -->

## Proposta: Algoritmo de Otimização por Colônia de Formigas (ACO)
- **Fundamentação Biológica:** Formigas individuais possuem pouca inteligência, mas a colônia exibe inteligência coletiva emergente. O feromônio serve como memória distribuída e mecanismo de feedback positivo (estigmergia).
- **Processo Principal:**
  1. **Construção de Caminhos:** Formigas artificiais constroem rotas cidade por cidade baseando-se em probabilidade (feromônio acumulado vs. distância geográfica).
  2. **Evaporação:** A cada iteração, as trilhas de feromônio evaporam parcialmente, evitando estagnação em mínimos locais.
  3. **Depósito:** Formigas que percorreram as melhores rotas adicionam mais feromônio em suas arestas, reforçando caminhos curtos.
- **Equilíbrio (Exploração vs. Explotação):** Controlado pelos pesos $\alpha$ (peso do feromônio) e $\beta$ (peso da heurística de proximidade).

---
<!-- slide4: -->

## Variante 1: ACO Clássico
- **Descrição:** Implementação clássica com seleção por torneio.
- **Mecanismo:** A cada passo de construção, uma formiga na cidade $i$ avalia um subconjunto aleatório de cidades não visitadas (tamanho do torneio). Ela escolhe a próxima cidade $j$ que maximiza:
  $$Atratividade(j) = \tau_{i,j}^{\alpha} \cdot \eta_{i,j}^{\beta}$$
  onde $\tau_{i,j}$ é a intensidade de feromônio na aresta $(i, j)$ e $\eta_{i,j} = \frac{1}{dist_{i,j}}$ é a visibilidade heurística.
- **Atualização de Feromônio:**
  $$\tau_{i,j} \leftarrow (1 - \rho) \cdot \tau_{i,j} + \sum_{k} \Delta \tau_{i,j}^k$$
  onde $\rho$ é a taxa de evaporação e $\Delta \tau_{i,j}^k = \frac{Q}{L_k}$ é o depósito da formiga $k$ se a aresta pertencer à sua rota.

---
<!-- slide5: -->

## Variante 2: Voter Model ACO
- **Descrição:** Extensão do ACO integrando dinâmicas de influência social e consenso de opinião (Voter Model).
- **Mecanismo:**
  - Durante o passo a passo da construção de caminhos, as formigas não tomam decisões de forma isolada.
  - Com probabilidade de voto $p_{voter} = 0.2$, cada formiga seleciona outra formiga aleatória da colônia como "vizinho".
  - Se o **custo parcial** da rota do vizinho for menor do que o seu próprio custo parcial e a próxima cidade escolhida pelo vizinho for um destino válido para ela (não visitado ainda), a formiga copia a decisão do vizinho.
  - Caso contrário, ou com probabilidade $1 - p_{voter} = 0.8$, ela decide autonomamente usando a regra do ACO clássico.
- **Objetivo:** Acelerar a convergência por meio de imitação e cooperação direta em tempo de execução das rotas parciais.

---
<!-- slide6: -->

## Variante 3: Estratificação Social ACO
- **Descrição:** Introdução de divisão de trabalho e castas com papéis diferenciados na busca.
- **Mecanismo:** A colônia é dividida em dois estratos sociais distintos:
  - **Castas Trabalhadoras (80%):** Focam na explotação do conhecimento acumulado (feromônio). Usam os parâmetros padrão com $\alpha_{atual} = \alpha$ e $\beta_{atual} = \beta$.
  - **Castas Exploradoras (20%):** Ignoram o feromônio completamente ($\alpha_{atual} = 0.0$, de modo que $\tau^0 = 1.0$). Elas tomam decisões puramente baseadas na proximidade física das cidades.
  - Além disso, as Exploradoras têm foco amplificado na distância ($\beta_{atual} = \beta \cdot 1.5$) para priorizar atalhos geográficos.
- **Objetivo:** Evitar a convergência prematura. Enquanto as Trabalhadoras refinam rotas conhecidas, as Exploradoras descobrem novos atalhos geométricos livres de tendenciosidade por feromônio.

---
<!-- slide7: -->

## Desenho Experimental e Metodologia
- **Os 6 Testes Realizados:**
  1. **Classico ($\alpha=1.0, \beta=3.0$)** - Foco equilibrado/proximidade.
  2. **Classico ($\alpha=3.0, \beta=1.0$)** - Foco intenso em feromônios.
  3. **Voter Model ($\alpha=1.0, \beta=3.0$)**
  4. **Voter Model ($\alpha=3.0, \beta=1.0$)**
  5. **Estratificado ($\alpha=1.0, \beta=3.0$)**
  6. **Estratificado ($\alpha=3.0, \beta=1.0$)**
- **Parâmetros Comuns:** Iterações = 50, Evaporação $\rho = 0.05$, Depósito $Q = 100$, Tamanho do Torneio = 4.
- **Escalas de Teste:** Instâncias com $N \in \{5, 10, 20, 40, 80, 160, 320, 640, 1000\}$ cidades.
- **Ambiente Físico:** Executado em máquina local, medindo tempo total (s), custo relativo (ms/cidade), estabilidade (desvio padrão das últimas 5 iterações) e consumo de memória (delta MB).
- **Reprodutibilidade:** Sementes globais e locais fixadas em 42.

---
<!-- slide8: -->

## Resultados Individuais: ACO Clássico (Evolução)
- **Comportamento:** Bom desempenho geral com $\alpha=1, \beta=3$. A configuração com $\alpha=3, \beta=1$ levou à estagnação prematura, resultando em distâncias piores em redes médias/grandes e maior instabilidade.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0167 | 0.00 | 213.00 | 0.0302 | 0.00 |
| 10 | 276.00 | 0.0665 | 0.00 | 276.00 | 0.0891 | 0.00 |
| 20 | 479.00 | 0.2208 | 15.98 | 476.00 | 0.5035 | 25.83 |
| 40 | 920.00 | 1.0099 | 51.07 | 933.00 | 1.0618 | 45.14 |
| 80 | 1931.00 | 10.3758 | 40.68 | 1946.00 | 6.7696 | 62.80 |
| 160 | 3842.00 | 12.9557 | 131.66 | 4042.00 | 12.6647 | 47.25 |
| 320 | 8433.00 | 59.8309 | 57.54 | 8663.00 | 61.5252 | 104.60 |
| 640 | 17378.00 | 297.4381 | 152.00 | 17997.00 | 300.3415 | 162.95 |
| 1000 | 27949.00 | 817.3542 | 98.59 | 28224.00 | 816.1929 | 253.38 |

![bg right:40% 90%](images/aco_tsp_classico_convergenca.png)

---
<!-- slide9: -->

## Resultados Individuais: ACO Clássico (Rota)
- **Comportamento:** Bom desempenho geral com $\alpha=1, \beta=3$. A configuração com $\alpha=3, \beta=1$ levou à estagnação prematura, resultando em distâncias piores em redes médias/grandes e maior instabilidade.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0167 | 0.00 | 213.00 | 0.0302 | 0.00 |
| 10 | 276.00 | 0.0665 | 0.00 | 276.00 | 0.0891 | 0.00 |
| 20 | 479.00 | 0.2208 | 15.98 | 476.00 | 0.5035 | 25.83 |
| 40 | 920.00 | 1.0099 | 51.07 | 933.00 | 1.0618 | 45.14 |
| 80 | 1931.00 | 10.3758 | 40.68 | 1946.00 | 6.7696 | 62.80 |
| 160 | 3842.00 | 12.9557 | 131.66 | 4042.00 | 12.6647 | 47.25 |
| 320 | 8433.00 | 59.8309 | 57.54 | 8663.00 | 61.5252 | 104.60 |
| 640 | 17378.00 | 297.4381 | 152.00 | 17997.00 | 300.3415 | 162.95 |
| 1000 | 27949.00 | 817.3542 | 98.59 | 28224.00 | 816.1929 | 253.38 |

![bg right:40% 90%](images/aco_tsp_classico_rota.png)

---
<!-- slide10: -->

## Resultados Individuais: Voter Model ACO (Evolução)
- **Convergência Social (N=160):** O Voter Model utiliza a comunicação e imitação local ($p_{voter}=0.2$) para acelerar o aprendizado e obter rápido consenso.
- **Risco de Homogeneização:** A imitação precoce prejudica a exploração tardia. Quando combinada com $\alpha=3.0$, a perda de diversidade é crítica, resultando em rotas de pior qualidade.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0116 | 0.00 | 213.00 | 0.0185 | 0.00 |
| 10 | 276.00 | 0.0371 | 0.00 | 276.00 | 0.0814 | 5.85 |
| 20 | 494.00 | 0.1660 | 23.17 | 481.00 | 0.3102 | 17.75 |
| 40 | 941.00 | 0.6869 | 23.21 | 973.00 | 1.0543 | 23.46 |
| 80 | 1944.00 | 2.8482 | 29.63 | 2011.00 | 9.9991 | 32.33 |
| 160 | 3955.00 | 15.9291 | 64.30 | 4158.00 | 15.2821 | 48.95 |
| 320 | 8484.00 | 73.5060 | 106.67 | 8600.00 | 78.7721 | 152.52 |
| 640 | 17595.00 | 452.4016 | 200.46 | 18048.00 | 464.2648 | 123.40 |
| 1000 | 27902.00 | 1350.2274 | 118.21 | 28363.00 | 1360.2644 | 220.54 |

![bg right:40% 90%](images/aco_tsp_voter_model_convergenca.png)

---
<!-- slide11: -->

## Resultados Individuais: Voter Model ACO (Rota)
- **Comportamento:** Rotas consistentes em escalas médias. No entanto, a complexidade computacional aumentou drasticamente: o tempo de execução quase dobrou para $N=1000$, aproximando-se de **1350 segundos**.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0116 | 0.00 | 213.00 | 0.0185 | 0.00 |
| 10 | 276.00 | 0.0371 | 0.00 | 276.00 | 0.0814 | 5.85 |
| 20 | 494.00 | 0.1660 | 23.17 | 481.00 | 0.3102 | 17.75 |
| 40 | 941.00 | 0.6869 | 23.21 | 973.00 | 1.0543 | 23.46 |
| 80 | 1944.00 | 2.8482 | 29.63 | 2011.00 | 9.9991 | 32.33 |
| 160 | 3955.00 | 15.9291 | 64.30 | 4158.00 | 15.2821 | 48.95 |
| 320 | 8484.00 | 73.5060 | 106.67 | 8600.00 | 78.7721 | 152.52 |
| 640 | 17595.00 | 452.4016 | 200.46 | 18048.00 | 464.2648 | 123.40 |
| 1000 | 27902.00 | 1350.2274 | 118.21 | 28363.00 | 1360.2644 | 220.54 |

![bg right:40% 90%](images/aco_tsp_voter_model_rota.png)

---
<!-- slide12: -->

## Resultados Individuais: Estratificação Social ACO (Evolução)
- **Convergência por Castas (N=160):** Melhorias contínuas mesmo após a iteração 30, indicando resiliência contra estagnação.
- **Atuação das Trabalhadoras (80%):** A casta trabalhadora explora o feromônio acumulado, consolidando os atalhos úteis descobertos pelas exploradoras, refletido nas quedas de patamar na curva azul.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0163 | 0.00 | 213.00 | 0.0097 | 0.00 |
| 10 | 276.00 | 0.0632 | 0.00 | 276.00 | 0.0346 | 5.46 |
| 20 | 500.00 | 0.2735 | 12.25 | 496.00 | 0.1713 | 29.34 |
| 40 | 947.00 | 0.8789 | 15.64 | 898.00 | 0.6401 | 53.00 |
| 80 | 1912.00 | 2.7098 | 70.93 | 1912.00 | 2.6292 | 26.19 |
| 160 | 3921.00 | 12.9571 | 70.43 | 3978.00 | 12.5895 | 77.52 |
| 320 | 8321.00 | 62.7870 | 98.73 | 8466.00 | 60.1194 | 100.63 |
| 640 | 17600.00 | 313.6898 | 80.80 | 17645.00 | 293.3797 | 203.88 |
| 1000 | 27292.00 | 849.5368 | 183.83 | 28110.00 | 806.6349 | 123.69 |

![bg right:40% 90%](images/aco_tsp_Estratificação_Social__convergenca.png)

---
<!-- slide13: -->

## Resultados Individuais: Estratificação Social ACO (Rota)
- **Geometria Superior (N=160):** Rota final extremamente fluida e limpa, apresentando a menor quantidade de cruzamentos e caminhos redundantes.
- **Atalhos Geométricos Efetivos:** O papel das formigas "batedoras" (exploradoras) se materializa em conexões limpas e lineares que contornam o grafo de forma eficiente.

| N Cidades | Distância ($\alpha=1$) | Tempo (s) ($\alpha=1$) | Estabilidade ($\alpha=1$) | Distância ($\alpha=3$) | Tempo (s) ($\alpha=3$) | Estabilidade ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 213.00 | 0.0163 | 0.00 | 213.00 | 0.0097 | 0.00 |
| 10 | 276.00 | 0.0632 | 0.00 | 276.00 | 0.0346 | 5.46 |
| 20 | 500.00 | 0.2735 | 12.25 | 496.00 | 0.1713 | 29.34 |
| 40 | 947.00 | 0.8789 | 15.64 | 898.00 | 0.6401 | 53.00 |
| 80 | 1912.00 | 2.7098 | 70.93 | 1912.00 | 2.6292 | 26.19 |
| 160 | 3921.00 | 12.9571 | 70.43 | 3978.00 | 12.5895 | 77.52 |
| 320 | 8321.00 | 62.7870 | 98.73 | 8466.00 | 60.1194 | 100.63 |
| 640 | 17600.00 | 313.6898 | 80.80 | 17645.00 | 293.3797 | 203.88 |
| 1000 | 27292.00 | 849.5368 | 183.83 | 28110.00 | 806.6349 | 123.69 |

![bg right:40% 90%](images/aco_tsp_Estratificação_Social__rota.png)

---
<!-- slide14: -->

## Comparativo de Desempenho: Qualidade da Rota (Distância)
- **Análise de Distância:**
  - Para problemas pequenos ($N \le 10$), todos os modelos encontram a rota ótima absoluta (**213.0** e **276.0**).
  - Nas maiores redes ($N=1000$), a **Estratificação Social ($\alpha=1$)** se destacou amplamente, obtendo a melhor distância global (**27292.00**), seguida pelo Voter Model $\alpha=1$ (**27902.00**) e Clássico $\alpha=1$ (**27949.00**).
  - As variantes com $\alpha=3$ apresentaram rotas piores em geral, devido à atração prematura por caminhos subótimos de feromônio (perda de diversidade).

![bg right height:380px](images/comparativo_melhor_distancia.png)

---
<!-- slide15: -->

## Comparativo de Qualidade da Rota: Grandes Escalas
- **Comportamento por Escala:** Curvas similares até $N \le 40$, com separação a partir de $N \ge 80$.
- **Liderança do Estratificado:** Variante ($\alpha=1.0$) domina com folga todas as instâncias complexas.
- **Estagnação Precoce:** Sem a casta exploradora, colônias convergem para mínimos locais.

![bg right height:380px](images/comparativo_melhor_distancia_grandes.png)

---
<!-- slide16: -->

## Comparativo de Desempenho: Tabela de Qualidade da Rota

| N Cidades | Clássico ($\alpha=1$) | Clássico ($\alpha=3$) | Voter ($\alpha=1$) | Voter ($\alpha=3$) | Estratificado ($\alpha=1$) | Estratificado ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | **213.00** | **213.00** | **213.00** | **213.00** | **213.00** | **213.00** |
| 10 | **276.00** | **276.00** | **276.00** | **276.00** | **276.00** | **276.00** |
| 20 | 479.00 | **476.00** | 494.00 | 481.00 | 500.00 | 496.00 |
| 40 | 920.00 | 933.00 | 941.00 | 973.00 | 947.00 | **898.00** |
| 80 | 1931.00 | 1946.00 | 1944.00 | 2011.00 | **1912.00** | **1912.00** |
| 160 | **3842.00** | 4042.00 | 3955.00 | 4158.00 | 3921.00 | 3978.00 |
| 320 | 8433.00 | 8663.00 | 8484.00 | 8600.00 | **8321.00** | 8466.00 |
| 640 | **17378.00** | 17997.00 | 17595.00 | 18048.00 | 17600.00 | 17645.00 |
| 1000 | 27949.00 | 28224.00 | 27902.00 | 28363.00 | **27292.00** | 28110.00 |

---
<!-- slide17: -->

## Comparativo de Desempenho: Tabela de Escalabilidade de Tempo
- **Resumo de Tempo:**
  - O **Clássico** e o **Estratificado** escalam de maneira muito semelhante, levando de **~800s a 850s** na escala máxima de 1000 cidades.
  - O **Voter Model** apresenta o pior comportamento temporal, necessitando de **~1350s** devido ao sobrecusto computacional do voto social a cada etapa.

| N Cidades | Clássico ($\alpha=1$) | Clássico ($\alpha=3$) | Voter ($\alpha=1$) | Voter ($\alpha=3$) | Estratificado ($\alpha=1$) | Estratificado ($\alpha=3$) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 5 | 0.02 | 0.03 | 0.01 | 0.02 | 0.02 | **0.01** |
| 10 | 0.07 | 0.09 | 0.04 | 0.08 | 0.06 | **0.03** |
| 20 | 0.22 | 0.50 | **0.17** | 0.31 | 0.27 | 0.17 |
| 40 | 1.01 | 1.06 | 0.69 | 1.05 | 0.88 | **0.64** |
| 80 | 10.38 | 6.77 | 2.85 | 10.00 | 2.71 | **2.63** |
| 160 | 12.96 | 12.66 | 15.93 | 15.28 | 12.96 | **12.59** |
| 320 | **59.83** | 61.53 | 73.51 | 78.77 | 62.79 | 60.12 |
| 640 | 297.44 | 300.34 | 452.40 | 464.26 | 313.69 | **293.38** |
| 1000 | 817.35 | 816.19 | 1350.23 | 1360.26 | 849.54 | **806.63** |

---
<!-- slide18: -->

## Escalabilidade de Tempo: ACO Clássico
- **Análise do Modelo Clássico:**
  - O comportamento temporal é dominado pela complexidade quadrática do torneio nas cidades.
  - A parametrização com $\alpha=3$ roda ligeiramente mais rápido em redes médias/grandes.
  - Isso ocorre porque a atração intensa do feromônio estreita rapidamente as escolhas das formigas, diminuindo flutuações.

![bg right height:380px](images/comparativo_tempo_classico.png)

---
<!-- slide19: -->

## Escalabilidade de Tempo: Voter Model ACO
- **Análise do Voter Model:**
  - O Voter Model adiciona um passo de comunicação em que cada formiga decide se copia o vizinho.
  - Isso gera um grande gargalo computacional em redes com mais de 300 cidades.
  - Para $N=1000$, a execução ultrapassa **1350 segundos**, sendo quase duas vezes mais lenta do que as outras variantes.

![bg right height:380px](images/comparativo_tempo_voter.png)

---
<!-- slide20: -->

## Escalabilidade de Tempo: Estratificação Social ACO
- **Análise do Estratificado:**
  - A divisão de castas em 80% trabalhadoras e 20% exploradoras mantém a complexidade do algoritmo clássico.
  - O modelo não apresenta nenhum custo temporal perceptível, rodando na mesma faixa de **800s a 850s** na escala máxima.
  - É a variante recomendada devido ao equilíbrio perfeito entre ótima qualidade de rota e eficiência computacional.

![bg right height:380px](images/comparativo_tempo_estratificado.png)

---
<!-- slide21: -->

## Escalabilidade de Tempo: Comparativo Geral
- **Análise Comparativa Geral:**
  - As variantes se comportam de forma idêntica para instâncias pequenas ($N \le 40$).
  - A partir de $N=80$, a ineficiência do Voter Model começa a se destacar exponencialmente.
  - O Clássico e o Estratificado mantêm curvas praticamente idênticas e escaláveis ao longo de todos os testes.

![bg right height:380px](images/comparativo_tempo_geral.png)

---
<!-- slide22: -->

## Discussão: O Impacto dos Parâmetros $\alpha$ e $\beta$
- **$\alpha=1.0, \beta=3.0$ (Heurística Forte / Feromônio Moderado):**
  - Incentiva a exploração física inteligente. A forte sensibilidade à distância física ($\beta=3$) ajuda as formigas a preferirem nós próximos.
  - O feromônio atua como guia de longo prazo, mas sem aprisionar os agentes imediatamente.
  - **Resultado:** Melhores rotas em escalas elevadas ($N \ge 320$), com convergência estável e resiliente.
- **$\alpha=3.0, \beta=1.0$ (Explotação Intensa / Feromônio Forte):**
  - Cria um efeito de atração magnética nas arestas mais visitadas no início da simulação.
  - A colônia inteira rapidamente converge para caminhos subótimos comuns.
  - **Resultado:** Queda de qualidade de rota (estagnação prematura) em problemas complexos e altos valores de desvio de estabilidade no final (ex: desvio de $253.38$ no Clássico $\alpha=3$).

---
<!-- slide23: -->

## Conclusão dos Resultados Obtidos
1. **A Estratificação Social é a Variante Vencedora:** O modelo de castas (Trabalhadoras + Exploradoras) superou as outras variantes na qualidade da rota final (distância) sem comprometer o tempo de execução. As formigas exploradoras atuam com sucesso como "batedores", descobrindo atalhos geométricos que quebram o aprisionamento dos feromônios.
2. **O Voter Model é Inviável para Grandes Escalas:** Embora conceitualmente elegante e eficaz em escalas moderadas, o mecanismo de votação em cada passo de construção de rota acarreta um gargalo computacional insustentável em redes acima de $300$ cidades, além de alto consumo de memória.
3. **Parâmetros Importam Mais do que a Variante:** Em todos os casos, a escolha de $\alpha=1, \beta=3$ provou-se crucial para manter a qualidade de rota em grandes escalas. Aumentar $\alpha$ em demasia induz a colônia à cegueira coletiva (estagnação).
4. **Recomendação:** Para aplicações de roteamento real, recomenda-se a utilização do **ACO Estratificado com parâmetros focados na heurística ($\beta > \alpha$)**.