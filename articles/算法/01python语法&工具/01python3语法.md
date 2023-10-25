# 1. python3字符串str

ASCII 英文编码
GB2312中文编码
Unicode 所有编码
utf-8：变长Unicode（英文1个字节，中文3个字节）

一般文本文件存储的是bytes（节省空间），而python3字符串用的是Unicode编码
所以，为了避免乱码应坚持使用utf-8编码对str和byters互相转换

字符串编码：'hello, 中国'.encode('utf-8')
字节流解码：b'ABC'.decode('utf-8', errors='ignore')忽略错误的字节，或者"GBK"

# 2. list tuple dict set

```
# list可以修改
list1 = ['wang', 12]
list2 = []

# tuple不能修改
tuple1 = ('wang', 'li', 'zhang')
tuple2 = ()     # 创建空的tuple
tuple3 = (1,)      # 创建1个元素的tuple

# dict
dict1 = {'Michael': 95, 'Bob': 75}
dict2 = {}

# set，没有重复值
set1 = set([1, 2])
set2 = set([1, 2, 3])
set3 = set()
print(set1 & set2)     # 交集
print(set1 | set2)     # 并集
print(set1)

# 再议不可变对象
a = ['c', 'b', 'a']
a.sort()
print(a)        # 发现a变了
a = 'abc'
a.replace('a', 'A')
print(a)        # 发现a没变
# 可变：列表、集合、字典
# 不可变：元组、字符串、整型、浮点型、布尔型
```

# 3. 循环、迭代

```
# 显示下标循环
list1 = ['a', 'b', 'c']
for i, value in enumerate(list1):
    print(i, value)
# 字典循环
dict1 = {'x': 1, 'y': 2}
for key, value in dict1.items():
    print(key, value)
# 生成器
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a+b
        n = n + 1
# 迭代器
for i in fib(5):
    print(i)
```

# 4. 函数 map reduce filter sortd lambda

```
# 高级函数
# map()
L = list(map(str, [1, 2, 3, 4]))       # 变成字符串
print(L)

# reduce()
from functools import reduce
def add(x, y):
    return x + y
print(reduce(add, [1, 2, 3, 4]))

# filter()
def is_odd(n):
    return n % 2 == 0
print(list(filter(is_odd, [1, 2, 3, 4])))

# sortd()
print(sorted([1, 2, 4, -3], key=abs))
print(sorted(['bob', 'Zoo'], key=str.lower, reverse=True))

# 匿名函数
print(list(map(lambda x: x**2, [1, 2, 3, 4])))
```

# 5. 类封装

```
# 封装
class Student(object):
    def __init__(self, name, age):
        self.__name = name
        self._age = age
    def print_age(self):
        print('%s, %d' % (self.__name, self._age))
    def get_age(self):
        return self.__age
# __name表示私有变量，外部不可直接访问
# _age提示是私有变量，外部最好不要直接访问

# 类属性
class Man(object):
    count = 0
    def __init__(self, name):
        self.__name = name
        Man.count += 1
man1 = Man('zhang')
man2 = Man('wang')
print(Man.count)
```

# 文件路径os.path

导入

```
import os
```

文件路径

```text
# 当前文件路径
os.path.dirname(__file__)

# 当前文件所在目录'/project/test'
os.path.dirname('/project/test/test.py')
# 上一层目录 '/project'
os.path.dirname('/project/test')

# 目录+文件
os.path.join(file_dir, "profile_instance.json")
```

遍历文件夹

    def recursive_listdir(path):
        files = os.listdir(path)
        for file in files:
            file_path = os.path.join(path, file)
        
            if os.path.isfile(file_path):
                print(file)
        
            elif os.path.isdir(file_path):
              recursive_listdir(file_path)
# 读写json和jsonl

json和jsonl另种文件的区别：

* json与jsonl的区别在于jsonl没有list只有并行的dict之间用"\n"分割，可以一次处理一条记录。可以用作日志文件或者其他。
* json和jsonl导入的包不一样

**json**

Python处理json文本文件主要是以下四个函数：

```
json.dumps	对数据进行编码,将python中的字典 转换为 字符串
json.loads	对数据进行解码,将 字符串 转换为 python中的字典
json.dump	将dict数据写入json文件中
json.load	打开json文件，并把字符串转换为python的dict数据
```

读

```
import json
with open('./xx.json', 'r', encoding='utf-8') as f:
    json_data = json.load(f)
    # 如果是列表[]，则遍历列表
    for row in json_data:
        # 如果是k,v, 则遍历k,v
        for k, v in row.items():
            print(k, v)
```

写

```
dict1 = {'name': '张三', 'age': 18, 'sex': '男'}
with open('./info.json','a',encoding='utf8') as fp:
	# ensure_ascii=False 因为默认ASCII编码，如果有中文就会乱码
	# indent=4 自动缩进是4
    json.dump(dict1, fp, ensure_ascii=False, indent=4)

# 如果不带 ensure_ascii=False
# '{"name": "\\u5f20\\u4e09"}'
```

**jsonl**

读

```
import jsonlines
with open("xxxx.jl", "r+", encoding="utf8") as f:
    for item in jsonlines.Reader(f):
        print(item)
```

写

    import jsonline
    with jsonlines.open('../output.jsonl', mode='a') as writer:
        writer.write(dict)
# assert

如果True，就继续执行程序；如果False，则输出后面语句

```
assert a > 0, "a超出范围"
```

