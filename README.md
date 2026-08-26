# AlgoStudies

Repositório criado para exercitar estudos de algoritmos e estruturas de dados. O foco principal é a conversão de conceitos teóricos em código limpo, de alta performance e gerenciamento eficiente de memória para grandes volumes de dados.

Este repositório contém as implementações das tarefas de programação propostas na Especialização de Algoritmos de Stanford.

---

## Estrutura do Repositório

```text
AlgoStudies/
├── README.md
└── stanford-specialization/
    ├── course2-graph-datastructures/
    │   ├── scc_kosaraju.py
    │   ├── 2sum_efficient.py
    │   └── median_maintenance.py
    └── course3-greedy-mst/
        ├── jobs_scheduling.py
        └── prim_mst.py
```

---

## 1. Algoritmo de Kosaraju (Strongly Connected Components)

### Descrição Teórica
Identifica Componentes Fortemente Conexas (SCCs) em grafos direcionados operando em tempo linear através de duas passagens de busca em profundidade (DFS). A primeira passagem computa os tempos de término no grafo reverso, estabelecendo um ordenamento de busca. A segunda passagem explora o grafo original seguindo essa ordem regressiva, isolando cada SCC perfeitamente. A implementação configura limites seguros de recursão e tamanho da pilha para evitar estouro em grafos massivos.

### Código em Python (scc_kosaraju.py)
```python
import sys
import threading

sys.setrecursionlimit(300000)
threading.stack_size(67108864)

def kosaraju_scc(num_nodes, edges):
    adj = {i: [] for i in range(1, num_nodes + 1)}
    adj_rev = {i: [] for i in range(1, num_nodes + 1)}
    for u, v in edges:
        adj[u].append(v)
        adj_rev[v].append(u)
    
    visited = [False] * (num_nodes + 1)
    finished_order = []
    
    def dfs_first(node):
        visited[node] = True
        for neighbor in adj_rev[node]:
            if not visited[neighbor]:
                dfs_first(neighbor)
        finished_order.append(node)
        
    for i in range(1, num_nodes + 1):
        if not visited[i]:
            dfs_first(i)
            
    visited = [False] * (num_nodes + 1)
    scc_sizes = []
    
    def dfs_second(node):
        visited[node] = True
        size = 1
        for neighbor in adj[node]:
            if not visited[neighbor]:
                size += dfs_second(neighbor)
        return size
        
    for node in reversed(finished_order):
        if not visited[node]:
            current_scc_size = dfs_second(node)
            scc_sizes.append(current_scc_size)
            
    scc_sizes.sort(reverse=True)
    return scc_sizes[:5]
```

---

## 2. Variante Eficiente do Algoritmo 2-SUM

### Descrição Teórica
Mapeia a quantidade de somas-alvo distintas no intervalo de -10000 a 10000 que podem ser formadas por pares de elementos de um array contendo 1 milhão de números. A otimização utiliza ordenação prévia combinada com busca binária de limites de intervalo (`bisect_left` e `bisect_right`), evitando o loop quadrático proibitivo e operando em tempo quase-linear.

### Código em Python (2sum_efficient.py)
```python
import bisect

filename = 'algo1-programming_prob-2sum.txt'
nums = []

with open(filename, 'r') as f:
    for line in f:
        if line.strip():
            nums.append(int(line.strip()))

nums = sorted(list(set(nums)))

def compute_2sum(nums):
    targets = set()
    n = len(nums)
    for i in range(n):
        x = nums[i]
        low = -10000 - x
        high = 10000 - x
        left_idx = bisect.bisect_left(nums, low, i + 1, n)
        right_idx = bisect.bisect_right(nums, high, left_idx, n)
        for j in range(left_idx, right_idx):
            targets.add(x + nums[j])
    return len(targets)

final_answer = compute_2sum(nums)
print("==================================")
print("RESPOSTA FINAL:")
print(final_answer)
print("==================================")
```

---

## 3. Manutenção de Mediana (Median Maintenance)

### Descrição Teórica
Mantém a mediana em tempo real a partir de um fluxo contínuo de números de entrada. A lógica utiliza dois heaps binários simultâneos: um max-heap que armazena a metade inferior dos números (os menores) e um min-heap que armazena a metade superior (os maiores). O algoritmo equilibra os tamanhos dos heaps a cada inserção, garantindo que o topo do max-heap seja sempre a mediana do conjunto.

### Código em Python (median_maintenance.py)
```python
import heapq

def median_maintenance(filename):
    min_heap = []
    max_heap = []
    medians = []
    with open(filename, 'r') as f:
        for line in f:
            if line.strip():
                num = int(line.strip())
                if not max_heap or num <= -max_heap[0]:
                    heapq.heappush(max_heap, -num)
                else:
                    heapq.heappush(min_heap, num)
                if len(max_heap) > len(min_heap) + 1:
                    val = -heapq.heappop(max_heap)
                    heapq.heappush(min_heap, val)
                elif len(min_heap) > len(max_heap):
                    val = heapq.heappop(min_heap)
                    heapq.heappush(max_heap, -val)
                medians.append(-max_heap[0])
    return sum(medians) % 10000
```

---

## 4. Agendamento Guloso de Tarefas (Jobs)

### Descrição Teórica
Determina a ordem ótima de execução de tarefas para minimizar a soma ponderada dos tempos de conclusão ($\sum w_j C_j$).
* **Heurística de Diferença ($weight - length$):** Abordagem sub-ótima que ordena de forma decrescente pela diferença. Os empates são decididos priorizando a tarefa de maior peso.
* **Heurística de Razão ($weight / length$):** Abordagem matematicamente provada como ótima. Ordena estritamente pela razão decrescente.

### Código em Python (jobs_scheduling.py)
```python
filename_jobs = 'jobs.txt'
jobs = []

with open(filename_jobs, 'r') as f:
    lines = f.readlines()
    num_jobs = int(lines[0].strip())
    for line in lines[1:]:
        if line.strip():
            parts = line.split()
            weight = int(parts[0])
            length = int(parts[1])
            jobs.append((weight, length))

def solve_by_difference(jobs_list):
    sorted_jobs = sorted(jobs_list, key=lambda x: (x[0] - x[1], x[0]), reverse=True)
    current_time = 0
    sum_weighted_completion = 0
    for weight, length in sorted_jobs:
        current_time += length
        sum_weighted_completion += weight * current_time
    return sum_weighted_completion

def solve_by_ratio(jobs_list):
    sorted_jobs = sorted(jobs_list, key=lambda x: x[0] / x[1], reverse=True)
    current_time = 0
    sum_weighted_completion = 0
    for weight, length in sorted_jobs:
        current_time += length
        sum_weighted_completion += weight * current_time
    return sum_weighted_completion

ans_diff = solve_by_difference(jobs)
ans_ratio = solve_by_ratio(jobs)
print("==================================")
print("DIFERENÇA: ", ans_diff)
print("RAZÃO: ", ans_ratio)
print("==================================")
```

---

## 5. Algoritmo de Prim para Árvore Geradora Mínima (MST)

### Descrição Teórica
Encontra uma árvore geradora de custo mínimo em um grafo não direcionado conectado. O algoritmo é iniciado em um nó arbitrário e expande a fronteira adicionando progressivamente a aresta de menor peso que se conecta a um nó não explorado. Uma fila de prioridades baseada em heap binário (`heapq`) otimiza a seleção de arestas de fronteira em cada rodada, funcionando de forma idêntica mesmo sob a presença de custos negativos.

### Código em Python (prim_mst.py)
```python
import heapq
from collections import defaultdict

filename_edges = 'edges.txt'
adj_list = defaultdict(list)
num_nodes = 0
num_edges = 0

with open(filename_edges, 'r') as f:
    lines = f.readlines()
    num_nodes, num_edges = map(int, lines[0].split())
    for line in lines[1:]:
        if line.strip():
            u, v, cost = map(int, line.split())
            adj_list[u].append((cost, v))
            adj_list[v].append((cost, u))

def run_prim(start_node=1):
    visited = set([start_node])
    heap = []
    for cost, neighbor in adj_list[start_node]:
        heapq.heappush(heap, (cost, neighbor))
    total_mst_cost = 0
    while heap and len(visited) < num_nodes:
        cost, node = heapq.heappop(heap)
        if node not in visited:
            visited.add(node)
            total_mst_cost += cost
            for neighbor_cost, neighbor in adj_list[node]:
                if neighbor not in visited:
                    heapq.heappush(heap, (neighbor_cost, neighbor))
    return total_mst_cost

ans_prim = run_prim()
print("==================================")
print("MST CUSTO TOTAL: ", ans_prim)
print("==================================")
```
