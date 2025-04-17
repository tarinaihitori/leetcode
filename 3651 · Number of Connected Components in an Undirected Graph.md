1st

再帰的な DFS。
スタックオーバーフロー（RecusionError）になるおそれがないかだけは確認しておく必要がある。
今回再帰の深さは 10,000 に設定されていた。
グラフが線形で n が 10000 より大きい場合、RecursionError が発生するが、今回はそのような値が来ないと仮定する。
時間計算量：O(V + E)
空間計算量：O(V + E)

```python
from typing import List
from collections import defaultdict

class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        adjacency_list = defaultdict(list)
        for node1, node2 in edges:
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)

        visited_nodes = set()

        def visit_connected_nodes(node_index):
            if node_index in visited_nodes:
                return
            visited_nodes.add(node_index)
            for neighbor in adjacency_list[node_index]:
                visit_connected_nodes(neighbor)

        count = 0
        for node_index in range(n):
            if node_index in visited_nodes:
                continue
            count += 1
            visit_connected_nodes(node_index)
        return count

```

2nd
1st を読みやすくする。基本的には変数名の変更。

```python
from typing import List
from collections import defaultdict

class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        def visit_connected_nodes(node_index:int):
            if node_index in visited_node_indexes:
                return
            visited_node_indexes.add(node_index)
            for neighbor_index in adjacency_list[node_index]:
                visit_connected_nodes(neighbor_index)

        adjacency_list = defaultdict(list)
        for node1_index, node2_index in edges:
            adjacency_list[node1_index].append(node2_index)
            adjacency_list[node2_index].append(node1_index)

        visited_node_indexes = set()
        connected_component_count = 0

        for node_index in range(n):
            if node_index in visited_node_indexes:
                continue
            connected_component_count += 1
            visit_connected_nodes(node_index)
        return connected_component_count
```

stack を用いた DFS
これだと、再帰を使用しないため、RecusionError を考慮しなくてよくなる。
時間計算量：O(V + E)
空間計算量：O(V + E)

```python
from typing import List
from collections import defaultdict


class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        def visit_connected_nodes(node_index):
            stack = []
            stack.append(node_index)
            while stack:
                current_node_index = stack.pop()
                if current_node_index in visited_node_indexes:
                    continue
                visited_node_indexes.add(current_node_index)
                for neighbor in adjacency_list[current_node_index]:
                    stack.append(neighbor)

        adjacency_list = defaultdict(list)
        for node1, node2 in edges:
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)

        visited_node_indexes = set()
        connected_component_count = 0

        for node_index in range(n):
            if node_index in visited_node_indexes:
                continue
            connected_component_count += 1
            visit_connected_nodes(node_index)
        return connected_component_count
```

BFS でも実装
時間計算量：O(V + E)
空間計算量：O(V + E)

```python
from typing import List
from collections import defaultdict, deque


class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        def visit_connected_nodes(node_index):
            queue = deque([node_index])
            while queue:
                current_node_index = queue.popleft()
                if current_node_index in visited_node_indexes:
                    continue
                visited_node_indexes.add(current_node_index)
                for neighbor in adjacency_list[current_node_index]:
                    queue.append(neighbor)

        adjacency_list = defaultdict(list)
        for node1, node2 in edges:
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)

        visited_node_indexes = set()
        connected_component_count = 0

        for node_index in range(n):
            if node_index in visited_node_indexes:
                continue
            connected_component_count += 1
            visit_connected_nodes(node_index)
        return connected_component_count

```

unionfind でも実装。
時間計算量：O(V + E)
空間計算量：O(V)

```python
from typing import List


class UnionFind:
    def __init__(self, size):
        self.parent = [i for i in range(size)]

    def find(self, node):
        if self.parent[node] != node:
            self.parent[node] = self.find(self.parent[node])
        return self.parent[node]

    def union(self, node1, node2):
        parent1 = self.find(node1)
        parent2 = self.find(node2)
        if parent1 != parent2:
            self.parent[parent1] = parent2


class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        union_find = UnionFind(n)

        for node1, node2 in edges:
            union_find.union(node1, node2)

        connected_components = set()

        for i in range(n):
            connected_components.add(union_find.find(i))
        return len(connected_components)
```

3rd
iterative な DFS で書いた。
スタックオーバーフローが起きる可能性のある入力が来た場合に対応できるようにするのと、
個人的には Union-Find を書き慣れていないのが理由。
UnionFind はソフトウェアエンジニアの常識の範囲内だと思うので、書ける&読めるようにはしておく。

```python
from typing import List
from collections import defaultdict


class Solution:
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        def visit_connected_nodes(node_index):
            stack = []
            stack.append(node_index)
            while stack:
                current_node_index = stack.pop()
                if current_node_index in visited_node_indexes:
                    continue
                visited_node_indexes.add(current_node_index)
                for neighbor in adjacency_list[current_node_index]:
                    stack.append(neighbor)

        adjacency_list = defaultdict(list)
        for node1, node2 in edges:
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)

        visited_node_indexes = set()
        connected_component_count = 0

        for node_index in range(n):
            if node_index in visited_node_indexes:
                continue
            connected_component_count += 1
            visit_connected_nodes(node_index)
        return connected_component_count
```
