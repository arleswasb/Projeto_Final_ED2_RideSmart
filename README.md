

## RideSmart — Modelagem e Análise de Rotas Urbanas com Grafos

![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python)
![Framework](https://img.shields.io/badge/Framework-OSMnx%20%7C%20NetworkX-orange?style=flat-square)
![Escopo](https://img.shields.io/badge/Escopo-Natal%20%7C%20RN-green?style=flat-square)

## Descrição do Projeto
O **RideSmart** é o projeto final da disciplina de **Estrutura de Dados II (ED2)**. O sistema simula o motor de roteamento de um aplicativo de mobilidade urbana multimodal aplicado à cidade de **Natal/RN**. 

O objetivo principal é resolver o seguinte problema de otimização viária:
> Dado um ponto de origem `A`, um destino `B` e uma distância máxima `X` que o usuário aceita caminhar, o algoritmo realiza uma varredura espacial para determinar o **Ponto de Embarque Ideal `P`** que minimiza o tempo global da viagem (Caminhada + Trajeto de Carro enfrentando trânsito).

```text
 Trecho 1: A ➔ P (Caminhada via G_pedestre)
 Trecho 2: P ➔ B (Carro com trânsito via G_carro)

```

##  Arquitetura e Modelagem do Grafo

Para evitar anomalias de roteamento e contaminação de regras viárias, a cidade de Natal foi modelada em **Duas Camadas de Grafos Independentes**:

1. **Rede de Carros (`G_carro`):** Focada estritamente em ruas transitáveis por veículos automotores. Respeita as restrições de sentido proibido (`oneway: True`) e possui custos ponderados por **trânsito sintético** (atrasos de pico variando de $1.5\times$ a $3.0\times$ em grandes artérias como a Via Costeira).
2. **Rede de Pedestres (`G_pedestre`):** Focada em calçadas, praças e passarelas. As restrições de mão única são ignoradas (`oneway: False`) e o custo é baseado puramente na distância física linear (`length`), considerando uma velocidade de caminhada humana constante de $1.2\text{ m/s}$.

---

##  Algoritmos Implementados e Comparados

O projeto avalia o desempenho empírico e a corretude de **5 algoritmos de caminhos mínimos**:

* **Dijkstra Simples:** Busca linear por menor custo com complexidade $O(V^2)$.
* **Dijkstra com Heap:** Otimizado com fila de prioridades binária, com complexidade $O(E \log V)$.
* **Dijkstra Bidirecional:** Algoritmo adicional da literatura que expande frentes de onda simultâneas da origem e do destino.
* **Algoritmo A-Estrela:** Busca heurística direcionada utilizando a **Fórmula de Haversine** adaptada para tempo como função de custo estimado.
* **Bellman-Ford:** Contraexemplo acadêmico utilizado para avaliar o impacto do relaxamento exaustivo de arestas $O(V \cdot E)$.

---

---

## 📊 Resultados e Análise Comparativa

### 🎯 Cenário de Teste Configurado (Oficial)
Para a validação final e auditoria das estruturas de dados, configuramos um cenário de teste real utilizando a ferramenta de projeção de coordenadas na malha de Natal/RN:

* **Origem (Nó A - Pedestre):** `3801088987` (Coordenadas: -5.79801, -35.21905)
* **Destino (Nó B - Carro):** `554860582` (Coordenadas: -5.80668, -35.20303)
* **Raio Máximo de Caminhada:** 500 metros na malha de pedestres (**169 pontos elegíveis avaliados**).
* **Ponto de Embarque Otimizado (P):** `302599917`

### ⏱️ Tabela de Desempenho (Padrão IEEE)

| Algoritmo (ED2) | Origem (Nó A) | Destino (Nó B) | Decisão | Ponto P Ideal | Tempo Global (min) | Nós Rota Carro | Runtime (ms) |
| :--- | :---: | :---: | :--- | :---: | :---: | :---: | :---: |
| **Dijkstra Simples O(V²)** | 3801088987 | 554860582 | Embarque em P (Multimodal) | 302599917 | 7.42 | 49 | 655.487,617 |
| **Dijkstra + Heap O(E log V)** | 3801088987 | 554860582 | Embarque em P (Multimodal) | 302599917 | 7.42 | 49 | 2.507,038 |
| **Dijkstra Bidirecional** | 3801088987 | 554860582 | Embarque em P (Multimodal) | 302599917 | 7.42 | 49 | 3.872,972 |
| **Algoritmo A* (Haversine)** | 3801088987 | 554860582 | Embarque em P (Multimodal) | 302599917 | 7.42 | 49 | **2.273,397** |
| **Bellman-Ford O(V·E)** | 3801088987 | 554860582 | Embarque em P (Multimodal) | 302599917 | 7.42 | 49 | 1.041.944,939 |

### 🧠 Discussão Crítica dos Resultados
* **Inteligência do Trade-off (Caminhada vs. Tempo):** O sistema provou sua robustez e validou a hipótese central do projeto ao decidir pelo **Embarque em P (Multimodal)**. O motor de roteamento detectou que caminhar até o nó `302599917` (escapando de gargalos ou vias lentas no entorno da origem) economiza tempo suficiente no trajeto do carro para compensar o esforço físico do pedestre, resultando em um tempo global otimizado de 7.42 minutos.
* **Explosão Combinatória no Bellman-Ford:** Como o algoritmo precisou varrer todas as arestas da malha veicular e multiplicá-las pelos 169 pontos elegíveis do raio de caminhada, o custo de processamento foi catastrófico: foram necessários mais de **1041 segundos (aproximadamente 17.3 minutos)** para encontrar a rota. Isso sacramenta matematicamente sua inviabilidade em aplicações de mobilidade urbana em tempo real.
* **Supremacia do A*:** O A* com a heurística de Haversine cravou o menor tempo de processamento absoluto (2.273 ms), superando o Dijkstra com Heap. Isso demonstra a excelente eficiência da heurística espacial ao "puxar" a busca na direção geográfica do destino, evitando abrir ramos viários desnecessários no sentido oposto ao movimento.

---

##  Visualização Multimodal no Mapa

O sistema utiliza a biblioteca `Folium` para renderizar o mapa interativo das rotas calculadas. A rota multimodal exibe o trecho a pé (**linha azul**) até o ponto de embarque otimizado, seguido pelo trajeto vehicular (**linha vermelha**) desviando dos gargalos de trânsito em Natal.

<img width="1013" height="726" alt="image" src="images/resultado.png" />


---

##  Como Executar o Projeto

1. Clone o repositório:

```bash
git clone [https://github.com/seu-usuario/Projeto_Final_ED2_RideSmart.git](https://github.com/seu-usuario/Projeto_Final_ED2_RideSmart.git)

```

2. Instale as dependências exigidas:

```bash
pip install osmnx networkx folium pandas matplotlib

```

3. Abra e execute o arquivo principal:

```bash
jupyter notebook PROJETO_FINAL.ipynb

```

---

## Componentes do Grupo

* **Werber Arles de Souza Barradas** - 20250070655
* **Nilton Fontes Barreto Neto** - 20250070771

---

**Junho de 2026** - Universidade Federal do Rio Grande do Norte (UFRN)

``

```
