---
tags:
  - IT/Python
---

## 去重

**单纯列表的去重方法对比**

| 方法                         | 保顺序 | 性能         | 推荐用途                |
| -------------------------- | --- | ---------- | ------------------- |
| `list(dict.fromkeys(...))` | ✅ 是 | ✅ 高效（O(n)） | ✅ 日常推荐用法            |
| `set(...)`                 | ❌ 否 | ✅ 高效（O(n)） | 顺序无要求时最快            |
| `if not in list: append`   | ✅ 是 | ❌ 慢（O(n²)） | 小数据量临时处理可用，不推荐用于大数据 |

如果是字典元素组成的列表，把字典变成可哈希的类型——去重的依据是“值相等”，而计算唯一性需要“可哈希（hashable）”。
```
unique = []
seen = set()

for d in detection_data_dum:
    key = frozenset(d.items())  # 把 dict 转成 hashable 的 key
    if key not in seen:
        seen.add(key)
        unique.append(d)
```




## 判断值的真假：`print(True if value else False)`

![[Pasted image 20240304002808.png]]


## 关于前置if判断条件：list比set快

![[Pasted image 20240304002747.png]]

## 字符串格式化

- %的形式在python2较常用，但有点问题：类型固定
	- `"string {1} {2}".format(arg1, arg2)`
- 3.6以后引入f-String（甚至能在{}内做计算）

## 列表·字典解析：不止简单、还快

![[Pasted image 20240304002540.png]]


### `if…else` 三元表达式 vs. 列表解析

如果想用三元表达式，它的完整形式是
```python
[值1 if 条件 else 值2 for 变量 in …]
```
- 必须带 `else`，否则也会语法错误。

如果只是要过滤（不需要 `else`），就应把 `if` 放在最后：
```python
[clean_value(i) for i in merged_file_info if clean_value(i)]
```






## 字典合并

- `dict1.update(dict2)`
- Python3.5+：`{**d1, **d2, **d3}`


## defaultdict() 安全访问字典

为不存在的键提供默认值而不是报错。
对比`dict.get()`需要指定检查特定键，defaultdict可以处理更复杂的情况。

另：
- 简单情况下用字典自带的get函数：`dict.get(key, default)`。default可省略，默认返回None。
> [!NOTE] 这是当访问的 key 不存在时返回默认值，而不是值！
> 希望当值是None时返回默认值的话：`(dict.get(key) or {})`
- 复杂条件下还是用if。
	![[Pasted image 20240304002425.png]]

### 扩展：getattr()方法

用于访问对象而不是字典。
语法：`getattr(object, attribute, default)`。`default`可省略，默认返回`None`。



## yield：生成器。节省内存


## lambda函数


## 一个单独的下划线：表示不关心的临时变量

如：`for _ in range(10)`