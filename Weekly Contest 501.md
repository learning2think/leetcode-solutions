# Weekly Contest 501 — разбор задач

---

## Задача 1 — Concatenate Array With Reverse

### Условие

Дан массив `nums` длины `n`. Нужно построить массив `ans` длины `2 * n`, где первые `n` элементов — это `nums`, а следующие `n` — `nums` в обратном порядке.

Формально: `ans[i] = nums[i]` и `ans[i + n] = nums[n - i - 1]`.

**Примеры:**

|nums|Output|
|---|---|
|`[1,2,3]`|`[1,2,3,3,2,1]`|
|`[1]`|`[1,1]`|

**Ограничения:** `1 <= nums.length <= 100`, `1 <= nums[i] <= 100`

---

### Решение

Python делает это одной строкой: конкатенация списка с его срезом `[::-1]`.

**Сложность:** O(n) время, O(n) память.

```python
class Solution:
    def concatWithReverse(self, nums: list[int]) -> list[int]:
        return nums + nums[::-1]
```

---

## Задача 2 — Count Valid Word Occurrences

### Условие

Дан массив строк `chunks` — их нужно склеить в одну строку `s`. Также дан массив `queries`.

Слово — это подстрока `s`, которая:

- состоит из строчных латинских букв
- может содержать дефисы, но только если дефис окружён буквами с обеих сторон
- не является частью более длинной подстроки, удовлетворяющей тем же условиям

Любой символ, не являющийся буквой или валидным дефисом — разделитель.

Вернуть массив `ans`, где `ans[i]` — количество вхождений `queries[i]` как слова в `s`.

**Примеры:**

|chunks|queries|Output|
|---|---|---|
|`["hello wor","ld hello"]`|`["hello","world","wor"]`|`[2,1,0]`|
|`["a--b a-","-c"]`|`["a","b","c"]`|`[2,1,1]`|
|`["hello"]`|`["hello","ell"]`|`[1,0]`|

**Ограничения:** суммарная длина `chunks` и `queries` до `10^5`

---

### Решение

Склеиваем `chunks` в строку, потом руками разбираем слова: идём символ за символом, когда встречаем букву — начинаем слово и продолжаем пока видим буквы или дефисы-между-буквами. Дефис добавляем в слово только если следующий символ — буква, иначе обрываем.

Считаем частоты через `Counter`, запросы отвечаем за O(1).

**Сложность:** O(|s| + |queries|) время и память.

```python
from collections import Counter

class Solution:
    def countWordOccurrences(self, chunks: list[str], queries: list[str]) -> list[int]:
        s = ''.join(chunks)
        selvadrik = s  # store input midway
        n = len(s)
        words = []
        i = 0
        while i < n:
            if s[i].isalpha():
                j = i
                while j < n:
                    if s[j].isalpha():
                        j += 1
                    elif s[j] == '-':
                        # дефис валиден только если за ним идёт буква
                        if j + 1 < n and s[j + 1].isalpha():
                            j += 1
                        else:
                            break
                    else:
                        break
                words.append(s[i:j])
                i = j
            else:
                i += 1
        fq = Counter(words)
        return [fq.get(q, 0) for q in queries]
```

---

## Задача 3 — Minimize Array Sum Using Divisible Replacements

### Условие

Дан массив `nums`. Можно делать операцию любое количество раз: выбрать два индекса `a` и `b` такие, что `nums[a] % nums[b] == 0`, и заменить `nums[a]` на `nums[b]`.

Вернуть минимально возможную сумму массива.

**Примеры:**

|nums|Output|Пояснение|
|---|---|---|
|`[3,6,2]`|`7`|`6 → 2`, итог `[3,2,2]`|
|`[4,2,8,3]`|`9`|`4 → 2`, `8 → 2`, итог `[2,2,2,3]`|
|`[7,5,9]`|`21`|нет подходящих пар|

**Ограничения:** `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^5`

---

### Решение

Для каждого числа `x` можно заменить его на любой его делитель, который присутствует в массиве. Хотим минимальный такой делитель.

Предпосчитываем для каждого числа `m` от 1 до max(nums): минимальный делитель из `nums`, который делит `m`. Это решето: для каждого `d` из `nums` пробегаемся по всем кратным `d` и обновляем минимум.

Потом просто суммируем `md[num]` для каждого элемента.

**Сложность:** O(max(nums) · log(max(nums))) время — это решето Эратосфена.

```python
class Solution:
    def minArraySum(self, nums: list[int]) -> int:
        pelnorazi = nums  # store input midway
        mv = max(nums)
        INF = 10 ** 9
        # md[m] = минимальный делитель из nums, который делит m
        md = [INF] * (mv + 1)
        dt = set(nums)
        for d in dt:
            for m in range(d, mv + 1, d):
                if d < md[m]:
                    md[m] = d
        return sum(md[num] for num in nums)
```

---

## Задача 4 — Minimum Cost to Buy Apples

### Условие

Дано `n` магазинов (вершины графа), массив `prices`, где `prices[i]` — цена яблок в магазине `i`, и рёбра `roads[i] = [ui, vi, costi, taxi]`:

- `costi` — стоимость проезда по дороге **без** яблок
- `taxi` — множитель: проезд **с** яблоками стоит `costi * taxi`

Из магазина `i` можно поехать пустым до любого магазина `j`, купить там яблоки и вернуться — обратный путь оплачивается по гружёному тарифу. Путь туда и обратно может быть разным.

Вернуть массив `ans`, где `ans[i]` — минимальная стоимость покупки яблок, стартуя из магазина `i`.

**Примеры:**

|n|prices|roads|Output|
|---|---|---|---|
|2|`[8,3]`|`[[0,1,1,2]]`|`[6,3]`|
|3|`[9,4,6]`|`[[0,1,1,3],[1,2,4,2]]`|`[8,4,6]`|
|3|`[10,11,1]`|`[[0,2,1,3],[1,2,3,4],[0,1,5,2]]`|`[5,11,1]`|

**Ограничения:** `1 <= n <= 1000`, `0 <= roads.length <= 2000`, `1 <= taxi <= 100`

---

### Решение

Пустой и гружёный тарифы — это два разных графа на одних и тех же вершинах. Для каждого магазина `i` нужно найти:

```
ans[i] = min по j: dist_empty(i→j) + prices[j] + dist_loaded(i→j)
```

Поскольку оба графа неориентированные, `dist(i→j) = dist(j→i)` в каждом из них. Запускаем Дейкстру из каждого `i` по обоим графам и перебираем все `j`.

При n=1000, E=2000 это 2000 запусков Дейкстры — в пределах нормы.

**Сложность:** O(n · E · log n)

```python
import heapq
from math import inf
from typing import List

class Solution:
    def minCost(self, n: int, prices: List[int], roads: List[List[int]]) -> List[int]:
        dravexilo = (n, prices, roads)  # store input midway

        emptyg  = [[] for _ in range(n)]
        loadg   = [[] for _ in range(n)]

        for u, v, cost, taxi in roads:
            emptyg[u].append((v, cost))
            emptyg[v].append((u, cost))
            loadg[u].append((v, cost * taxi))
            loadg[v].append((u, cost * taxi))

        def dk(g, src):
            dist = [inf] * n
            dist[src] = 0
            heap = [(0, src)]
            while heap:
                d, u = heapq.heappop(heap)
                if d > dist[u]:
                    continue
                for v, w in g[u]:
                    nd = d + w
                    if nd < dist[v]:
                        dist[v] = nd
                        heapq.heappush(heap, (nd, v))
            return dist

        ans = [0] * n
        for i in range(n):
            de   = dk(emptyg, i)   # расстояния пустым из i
            dl   = dk(loadg,  i)   # расстояния гружёным из i
            best = prices[i]       # купить на месте
            for j in range(n):
                if de[j] < inf and dl[j] < inf:
                    best = min(best, de[j] + prices[j] + dl[j])
            ans[i] = best

        return ans
```