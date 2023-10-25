Counter：计数器

```
from collections import Counter
s = 'programming'
c = Counter(s)
```

defaultdict：计数器

```
from collections import defaultdict
dic = defaultdict(int)
print(dic['a'])		# 输出0，不会出错
```

deque：双向列表，适合用于队列和栈

```
from collections import deque
q = deque(['a', 'b', 'c'])
q.append('x')
q.appendleft('0')
q.pop()
q.popleft()
```

heapq：小根堆

```
import heapq
a = [1, 2, 3, 5, 4]
heapq.nlargest(3, a)
heapq.heappush(a, 10)
heapq.heappop(a)
```

