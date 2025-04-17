1st
1つめのフェンスはk通りで塗れる。2つなら同じ色を使えるので、2つめのフェンスも同様にk*k通りで塗れる。
3つ目以降は、1つ前と違う色で塗る場合、1つ前の塗り方の数* (k-1)通りがある。
1つ前と同じ塗り方の場合、3つ連続したらだめなので、2つ前の塗り方の数 * (k-1)通りある。

```python
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n == 0　or k ==0 :
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

2nd
1を読みやすくする。
cacheして少し効率化した。とはいえrangeの中に3などのハードコードされた値があるのが気持ち的にいやだなあ。

```python
from functools import lru_cache
class Solution:
    @lru_cache(maxsize=None)
    def num_ways(self, n: int, k: int) -> int:
        if n == 0 or k ==0:
            return 0
        if n == 1:
            return k
        
        ways_to_paint = [0] * (n+1)
        ways_to_paint[1] = k
        ways_to_paint[2] = k * k

        for i in range(3, n+1):
            ways_to_paint[i] = (k-1) * (ways_to_paint[i-1] + ways_to_paint[i-2])
        return ways_to_paint[n]
```

ほかの人のコードをよむ
<https://github.com/Fuminiton/LeetCode/pull/30/files>
<https://github.com/olsen-blue/Arai60/pull/30/files>
<https://github.com/katsukii/leetcode/pull/16>
トップダウンとボトムアップというワードが出てきたので調べた。
>トップダウンDP（メモ化再帰）
問題を小さな部分問題に分解し、再帰的に解いた結果をメモ化（キャッシュ）して再利用する。
ボトムアップDP
一方、ボトムアップアプローチでは、初期状態からスタートして順次小さな部分問題の解を計算し、それを利用してより大きな問題を解いていく。
DPには上記のようなアプローチの違いで分けることができるらしい。
私のはボトムアップ的なアプローチらしい。

メモ化再帰で書いたもの

```python
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n == 0 or k == 0:
            return 0
        index_to_ways = {}
        def count_ways(index):
            if index in index_to_ways:
                return index_to_ways[index]
            if index == 1:
                return k
            if index == 2:
                return k * k
            index_to_ways[index] = (k-1) * (count_ways(index -1) + count_ways(index - 2))
            return index_to_ways[index]
        
        return count_ways(n)
```

あと、この関数が2回呼ばれたときどうなるのかついて、
同一インスタンス内の場合、self は同一なので、同じ n と k で呼び出したときにキャッシュヒットが発生し、計算が省略される。
逆に異なるインスタンスならキャシュはクリアされるという認識。（間違っていたらコメントください）

小田さんのコメントで、自分で実装できるならキャッシュなどのライブラリを使ってもよいとあるので、
こういうのは自分で実装できるようにならないといけないんだろうなというところで、自分でキャッシュを実装してみる。
まずデコレータをどう作るのかから手間取った。
<https://discord.com/channels/1084280443945353267/1233295449985650688/1247422212982440038>

```python

class Node:
    def __init__(self, key=None, value=None, next=None, prev=None):
        self.key = key
        self.value = value
        self.prev = prev
        self.next = next

class LRU_Cache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.size = 0
        self.cache = {}
        self.sentinel = Node(1, 1)
        self.sentinel.next = self.sentinel
        self.sentinel.prev = self.sentinel

    def get(self, key):
        if key not in self.cache:
            return None
        node = self.cache[key]
        self.remove(node.key)
        self.insert_front(node.key, node.value)
        return node.value

    def remove(self, key):
        node = self.cache[key]
        node.prev.next = node.next
        node.next.prev = node.prev
        del self.cache[key]
        self.size -= 1

    def insert_front(self, key, value):
        node = Node(key, value)
        next_node = self.sentinel.next
        self.sentinel.next = node
        node.prev = self.sentinel
        node.next = next_node
        next_node.prev = node
        self.size += 1
        self.cache[key] = node
        if self.size > self.capacity:
            last = self.sentinel.prev
            self.remove(last.key)


def lru_cache(maxsize=1):
    def decorating_function(func):
        nonlocal maxsize
        if maxsize <= 0:
            maxsize = 1
        return _lru_cache_wrapper(func, maxsize)
    return decorating_function


def _lru_cache_wrapper(func, maxsize):
    cache = LRU_Cache(maxsize)
    def wrap(*args, **kwargs):
        result = cache.get(args)
        if result is not None:
            return result
        result = func(*args, **kwargs)
        cache.insert_front(args, result)
        return result
    return wrap

class Solution:
    @lru_cache(maxsize=10000)
    def num_ways(self, n: int, k: int) -> int:
        if n == 0 or k ==0:
            return 0
        if n == 1:
            return k
        
        ways_to_paint = [0] * (n+1)
        ways_to_paint[1] = k
        ways_to_paint[2] = k * k

        for i in range(3, n+1):
            ways_to_paint[i] = (k-1) * (ways_to_paint[i-1] + ways_to_paint[i-2])
        return ways_to_paint[n]
```

結局1つ前と同じ色で塗るか、違う色で塗るかの違いなので、配列ではなく
変数で管理するようにしたコード

```python
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n == 0 or k ==0:
            return 0
        if n ==1:
            return k
        
        ways_to_paint_same_color = k
        ways_to_paint_different_color = k *(k-1)

        for i in range(3, n +1):
            temp = ways_to_paint_different_color 
            ways_to_paint_different_color = (ways_to_paint_same_color + ways_to_paint_different_color) * (k -1)
            ways_to_paint_same_color = temp 
        return ways_to_paint_different_color + ways_to_paint_same_color

```

3rd

ボトムアップDP

```python
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 0 or k == 0:
            return 0
        if n == 1:
            return k
        
        ways_to_paint = [0] * (n + 1)
        ways_to_paint[1] = k
        ways_to_paint[2] = k * k

        for i in range(3, n + 1):
            ways_to_paint[i] = (k-1) * (ways_to_paint[i-1] + ways_to_paint[i-2])
        return ways_to_paint[n]
```
