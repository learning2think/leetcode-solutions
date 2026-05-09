# Biweekly Contest — разбор задач

---

## Задача 1 — Score Validator

### Условие

Дан массив строк `events`. Изначально `score = 0`, `counter = 0`. Каждый элемент — одно из следующих событий:

- `"0"`, `"1"`, `"2"`, `"3"`, `"4"`, `"6"` — прибавить это значение к `score`
- `"W"` — увеличить `counter` на 1 (score не меняется)
- `"WD"` — прибавить 1 к `score`
- `"NB"` — прибавить 1 к `score`

Обработка идёт слева направо. Остановиться когда все элементы обработаны **или** `counter` достиг 10.

Вернуть `[score, counter]`.

**Примеры:**

|events|Output|
|---|---|
|`["1","4","W","6","WD"]`|`[12, 1]`|
|`["WD","NB","0","4","4"]`|`[10, 0]`|
|`["W","W","W","W","W","W","W","W","W","W","W"]`|`[0, 10]`|

**Ограничения:** `1 <= events.length <= 1000`

---

### Решение

Чистая симуляция. Единственная нетривиальная деталь: `break` нужно делать **сразу после** инкремента `counter` — как только он стал 10, следующие события уже не обрабатываются.

**Сложность:** O(n) время, O(1) память.

```python
class Solution:
    def scoreValidator(self, events: list[str]) -> list[int]:
        sc = 0
        c = 0
        for event in events:
            if event in {"0", "1", "2", "3", "4", "6"}:
                sc += int(event)
            elif event == "W":
                c += 1
                if c == 10:
                    break          # сразу останавливаемся
            elif event == "WD":
                sc += 1
            elif event == "NB":
                sc += 1
        return [sc, c]
```

---

## Задача 2 — Minimum Flips to Make String Coherent

### Условие

Дана бинарная строка `s`. Строка называется **coherent**, если она не содержит `"011"` или `"110"` как подпоследовательности.

За одну операцию можно перевернуть любой символ (`'0'` → `'1'` или `'1'` → `'0'`).

Вернуть минимальное количество флипов, чтобы сделать строку coherent.

**Примеры:**

|s|Output|Пояснение|
|---|---|---|
|`"1010"`|`1`|Flip s[0] → `"0010"`|
|`"0110"`|`1`|Flip s[1] → `"0010"`|
|`"1000"`|`0`|Уже coherent|

**Ограничения:** `1 <= s.length <= 10^5`

---

### Решение

Сначала надо понять, как выглядит coherent строка. `"011"` требует нуль слева и две единицы правее него. `"110"` — две единицы и ноль правее. Получается, любой блок из 2+ единиц рядом с нулём — уже нарушение. Безопасных форм ровно три:

1. Все `'0'` — стоит `count_ones` флипов
2. Все `'1'` — стоит `count_zeros` флипов
3. Ровно одна `'1'` где угодно — стоит `count_ones - 1` флипов (оставляем одну там, где она уже есть)

Берём минимум.

**Сложность:** O(n) время, O(1) память.

```python
class Solution:
    def minFlips(self, s: str) -> int:
        n = len(s)
        total_ones = s.count('1')

        # вариант 1: все нули
        ans = total_ones
        # вариант 2: все единицы
        ans = min(ans, n - total_ones)
        # вариант 3: ровно одна единица
        if total_ones == 0:
            ans = min(ans, 1)
        else:
            ans = min(ans, total_ones - 1)
        if n >= 2:
            left = 1 if s[0] == '1' else 0
            right = 1 if s[-1] == '1' else 0
            flips_ends = total_ones + 2 - 2 * (left + right)
            ans = min(ans, flips_ends)

        return ans
```

---

## Задача 3 — Minimum Generations

### Условие

Дан массив точек `points`, где `points[i] = [xi, yi, zi]` — точка в 3D пространстве, и целевая точка `target`.

**Поколение 0** — исходный список точек. Для каждого `k >= 1` поколение `k` формируется так:

- берём все пары различных точек из поколений 0 до k-1
- для каждой пары `a = [x1,y1,z1]` и `b = [x2,y2,z2]` вычисляем `c = [⌊(x1+x2)/2⌋, ⌊(y1+y2)/2⌋, ⌊(z1+z2)/2⌋]`

Вернуть наименьшее `k`, при котором `target` появляется в поколениях 0..k. Если невозможно — вернуть `-1`.

**Примеры:**

|points|target|Output|
|---|---|---|
|`[[0,0,0],[6,6,6]]`|`[3,3,3]`|`1`|
|`[[0,0,0],[5,5,5]]`|`[1,1,1]`|`2`|
|`[[0,0,0],[2,2,2],[3,3,3]]`|`[2,2,2]`|`0`|
|`[[1,2,3]]`|`[5,5,5]`|`-1`|

**Ограничения:** `1 <= points.length <= 20`, координаты `0..6`

---

### Решение

Координаты от 0 до 6, значит всего `7³ = 343` возможных точки во всём пространстве. Можно просто симулировать.

Держим множество `visited` — все точки, накопленные с поколения 0. На каждом шаге перебираем все пары из `visited` и смотрим, какие midpoint'ы получились новыми. Если новых нет — target недостижим. Важно: пары берутся из **всех** накопленных точек, не только из предыдущего поколения — именно так работает `visited`.

**Сложность:** O(343³) в худшем случае — на практике несколько миллисекунд.

```python
from typing import List

class Solution:
    def minGenerations(self, points: List[List[int]], target: List[int]) -> int:
        morvilexa = points  # store input midway

        visited = {tuple(p) for p in points}
        tgt = tuple(target)

        if tgt in visited:
            return 0

        gen = 0
        while True:
            pts = list(visited)
            n = len(pts)
            new_points = set()

            for i in range(n):
                for j in range(i + 1, n):
                    a = pts[i]
                    b = pts[j]
                    mid = (
                        (a[0] + b[0]) // 2,
                        (a[1] + b[1]) // 2,
                        (a[2] + b[2]) // 2,
                    )
                    if mid not in visited:
                        new_points.add(mid)

            if not new_points:
                return -1   # новых точек нет — target недостижим

            gen += 1
            if tgt in new_points:
                return gen

            visited.update(new_points)
```

---

## Задача 4 — Minimum Threshold

### Условие

Дан неориентированный взвешенный граф из `n` вершин (0..n-1), рёбра `edges[i] = [ui, vi, wi]`, а также `source`, `target`, `k`.

Порог `threshold` определяет:

- **лёгкое** ребро: вес `<= threshold`
- **тяжёлое** ребро: вес `> threshold`

Путь от `source` до `target` валиден, если содержит не более `k` тяжёлых рёбер.

Вернуть минимальный `threshold`, при котором существует хотя бы один валидный путь. Если путь невозможен — вернуть `-1`.

**Примеры:**

|n|edges|source|target|k|Output|
|---|---|---|---|---|---|
|6|`[[0,1,5],[1,2,3],[3,4,4],[4,5,1],[1,4,2]]`|0|3|1|`4`|
|6|`[[0,1,3],[1,2,4],[3,4,5],[4,5,6]]`|0|4|1|`-1`|
|4|`[[0,1,2],[1,2,2],[2,3,2],[3,0,2]]`|0|0|0|`0`|

**Ограничения:** `1 <= n <= 10^3`, `0 <= edges.length <= 10^3`, `1 <= wi <= 10^9`

---

### Решение

Чем больше threshold — тем меньше тяжёлых рёбер — тем проще уложиться в лимит `k`. Это монотонность, а монотонность означает бинарный поиск. Бинарим не по всему диапазону `[0, 10⁹]`, а только по реальным весам рёбер (плюс `0` как кандидат) — между двумя соседними весами картина "лёгкое/тяжёлое" не меняется.

Для каждого кандидата нужно проверить: можно ли дойти от `source` до `target`, использовав не более `k` тяжёлых рёбер? Это кратчайший путь, где лёгкие рёбра стоят 0, тяжёлые — 1. Классический 0-1 BFS: лёгкое ребро — в начало deque, тяжёлое — в конец.

Если `source == target` — ответ сразу `0`, путь нулевой длины.

**Сложность:** O(E · log E · (V + E))

```python
from collections import deque

class Solution:
    def minimumThreshold(self, n: int, edges: list[list[int]],
                         source: int, target: int, k: int) -> int:

        tarnicuvo = (n, edges, source, target, k)  # store input midway

        if source == target:
            return 0

        graph = [[] for _ in range(n)]
        for u, v, w in edges:
            graph[u].append((v, w))
            graph[v].append((u, w))

        # 0 добавляем явно — threshold может быть меньше минимального веса
        candidates = sorted(set([0] + [w for _, _, w in edges]))

        def min_heavy(threshold: int) -> int:
            """0-1 BFS: минимальное число тяжёлых рёбер до target"""
            dist = [float('inf')] * n
            dist[source] = 0
            dq = deque([source])

            while dq:
                u = dq.popleft()
                for v, w in graph[u]:
                    heavy = 0 if w <= threshold else 1
                    new_dist = dist[u] + heavy
                    if new_dist < dist[v]:
                        dist[v] = new_dist
                        if heavy == 0:
                            dq.appendleft(v)  # лёгкое — в начало
                        else:
                            dq.append(v)      # тяжёлое — в конец

            return dist[target]

        lo, hi = 0, len(candidates) - 1
        ans = -1

        while lo <= hi:
            mid = (lo + hi) // 2
            threshold = candidates[mid]

            if min_heavy(threshold) <= k:
                ans = threshold   # работает — пробуем меньше
                hi = mid - 1
            else:
                lo = mid + 1      # не работает — нужен больше

        return ans
```