

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

##  Resultados e Análise Comparativa

### Cenário de Teste Configurado
Para a validação e auditoria dos algoritmos da suíte, foi estabelecida uma rota de estresse viário interbairros na cidade de Natal/RN em horário de pico, configurada com os seguintes parâmetros:

* **Origem (Nó A):** `12633403253` — Localizado nas proximidades da **Universidade Federal do Rio Grande do Norte (UFRN)**.
* **Destino (Nó B):** `535292181` — Localizado na região central do bairro de **Tirol**.
* **Raio Máximo de Caminhada ($X$):** $500\text{ metros}$ na malha de pedestres.
* **Ponto de Embarque Otimizado (P):** `30142155` — Nó estratégico fora da zona de retenção primária da origem.

### Tabela de Desempenho (Padrão IEEE)

| Algoritmo (ED2) | Decisão Decidida | Ponto P Ideal | Tempo Global (min) | Nós Rota Carro | Runtime (ms) |
| --- | --- | --- | --- | --- | --- |
| **Dijkstra Simples O(V²)** | Embarque em P | 30142155 | 46.66 | 45 | 7.918.322 |
| **Dijkstra + Heap O(E log V)** | Embarque em P | 30142155 | 46.66 | 45 | 456.343 |
| **Dijkstra Bidirecional** | Embarque em P | 30142155 | 46.66 | 45 | 972.479 |
| **Algoritmo A estrela (Haversine)** | Embarque em P | 30142155 | 46.66 | 45 | **526.083** |
| **Bellman-Ford O(V·E)** | Embarque em P | 30142155 | 46.66 | 45 | 379.829.769 |

###  Principais Conclusões de ED2

* **Consistência Matemática:** Todos os 5 algoritmos convergiram exatamente para o mesmo Ponto P, tempo de viagem e número de nós, validando a exatidão das implementações.
* **O Tabu do Bellman-Ford:** Levou **mais de 6 minutos** ($379\text{ segundos}$) para computar o resultado. Isso prova empiricamente a sua ineficiência em redes viárias reais que não possuem arestas de peso negativo.
* **Dijkstra Heap vs. A*:** O $A^*$ demonstrou a melhor eficiência de varredura espacial, restringindo o espaço de busca na direção do destino graças à heurística admissível de Haversine.

---

##  Visualização Multimodal no Mapa

O sistema utiliza a biblioteca `Folium` para renderizar o mapa interativo das rotas calculadas. A rota multimodal exibe o trecho a pé (**linha azul**) até o ponto de embarque otimizado, seguido pelo trajeto vehicular (**linha vermelha**) desviando dos gargalos de trânsito em Natal.

*(Dica: Adicione aqui uma captura de tela do mapa gerado no seu notebook!)*

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

* **Nome Completo 1** - Matrícula 1
* **Nome Completo 2** - Matrícula 2
* **Nome Completo 3** - Matrícula 3
* **Nome Completo 4** - Matrícula 4

---

**Junho de 2026** - Universidade Federal do Rio Grande do Norte (UFRN)

```

---

### 🏁 Próximos Passos
Com o código limpo, a tabela preenchida com os dados exatos da sua bateria de testes e o `README.md` pronto, o seu repositório está impecável. 

Agora só nos resta a última etapa do projeto: o **Relatório Científico de até 4 páginas no formato IEEE**. Quer que eu gere o esqueleto em sintaxe LaTeX desse artigo para você colar no seu editor?

```
