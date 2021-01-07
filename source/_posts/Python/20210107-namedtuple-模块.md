---
title: namedtuple 模块
date: 2021-01-07 10:57:47
tags:
  - Python
categories:
  - Python
---

## Namedtuple: A useful data structure for explicit coding
namedtuple 是 collections 包中的一个数据结构，可以很方便的存储从 csv, 数据库 中读取到的数据，**这种数据的特点是，数据分为多条记录，每一条记录包含若干字段。**

<!-- more -->

下面以一个班级学生的成绩表为例讲解一下，成绩表中包含 学号，姓名，成绩 三个字段
### list/dict 存储
如果对 Python 的各种数据类型接触不多，很容易会选择使用 list 或者 dict 去存储。
```python
raw_data = [
    [ 01, 'jack', '80'],
    [ 02, 'mike', '82'],
    [ 03, 'mary', '96'],
]

list_sheet = []
for record in raw_data:
    list_sheet.append(record)
# 这种方法的缺点是，每一条记录的各个字段具有什么含义并不清晰

# 列表嵌字典
dict_sheet = []
dict_record = {
    'stu_id': None,
    'name': None,
    'score': None
}
for record in raw_data:
    _id, _name, _score = record
    dict_record['stu_id'] = _id
    dict_record['name'] = _name
    dict_record['score'] = _score
    dict_sheet.append(dict_record)
   
# 字典嵌列表 
dict_sheet2 = {
    'stu_id': [],
    'name': [],
    'score': [],
}
for record in raw_data:
    _id, _name, _score = record
    dict_sheet2['stu_id'].append(_id)
    dict_sheet2['name'].append(_name)
    dict_sheet2['score'].append(_score)
# 这两种方法代码写起来比较麻烦，而且如果字典的 value 是 tuple/list 类型理解起来会有一些困难
```

### class 方法
熟练使用 C/C++ 的人，比较容易想到用 class 来存储数据
```python
def Record():
    stu_id = None
    name = None
    score = None
    def __init__(self, _id, _name, _score):
        self.stu_id = _id
        self.name = _name
        self.score = _score
    
sheet = []
for record in raw_data:
    sheet.append(Record(**record))
# 这种方法需要定义出来一个 class，代码同样显得比较麻烦
```

### namedtuple
实际上，这种场景最合适的数据结构就是 namedtuple，namedtuple 可以理解为用最简单的代码去定义一个 Record `Class`
```
from collections import namedtuple

Record = namedtuple('Record', ['stu_id', 'name', 'score'])

sheet = []
for record in raw_data:
    sheet.append(Record._make(record))
    
for r in sheet:
    print(r.stu_id, r.name, r.score)
# 相比之下，这种方法显得非常 explicit，习惯使用这种数据结构，处理 csv 数据的时候可以大大提高代码可读性。
```
