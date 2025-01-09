1st

```python
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n == 0:
            return 0
        if n == 1:
            return k

        way_of_paint = [0] * (n + 1)
        way_of_paint[1] = k
        way_of_paint[2] = k * k

        for i in range(3, n + 1):
            way_of_paint[i] = (k - 1) * (way_of_paint[i - 1] + way_of_paint[i - 2])

        return way_of_paint[n]

```
