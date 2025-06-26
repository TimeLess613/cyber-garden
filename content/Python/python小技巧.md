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



## 时间解析

**python3.9以上推荐标准库 zoneinfo**

### 时区转换

```python
from datetime import datetime
from zoneinfo import ZoneInfo

# 假设已有 UTC 时间 dt_utc
dt_utc = datetime.fromisoformat("2025-06-26T12:34:56+00:00")

# 转换到日本东京时间
dt_jst = dt_utc.astimezone(ZoneInfo("Asia/Tokyo"))
print(dt_jst.isoformat())  # 2025-06-26T21:34:56+09:00
print(dt_jst) # 会调用 `datetime` 对象的 `__str__`，默认输出类似：2025-06-26 21:34:56+09:00（用空格而不是 T 分隔日期和时间）

#### ------------------------- ####

# 转 UTC
s = "2025-06-26T21:34:56+09:00"
dt_utc = datetime.fromisoformat(s).astimezone(ZoneInfo("UTC"))

#### ------------------------- ####

# 如果你拿到的是“无时区”的（naive）`datetime` 字符串，比如 `"2025-06-26T21:34:56"`，可先指定来源时区，再转换：
# 先解析/构造一个 naive datetime，再赋给来源时区
dt_naive = datetime.fromisoformat("2025-06-26T21:34:56")
dt_with_tz = dt_naive.replace(tzinfo=ZoneInfo("Europe/London"))  # 如果本身是伦敦时间
# 再转 UTC。
dt_utc = dt_with_tz.astimezone(ZoneInfo("UTC"))
print(dt_utc.isoformat())  # 输出相应的 UTC 时间

```

> [!note] `datetime.fromisoformat` 在 3.9–3.10 中**不支持**`Z`（Zulu）的直接解析，需要预处理替换为 `+00:00`。（3.11之后就支持了）

```python
def parse_iso8601(s: str) -> datetime:
    if s.endswith("Z"):
        s = s[:-1] + "+00:00"
    return datetime.fromisoformat(s)

# 示例
dt = parse_iso8601("2025-06-26T12:34:56Z")
```

#### zoneinfo 的时区查找

```bash
# 1. Linux 命令
$ ls /usr/share/zoneinfo    # 这个好用
$ timedatectl list-timezones

# 2.从 zoneinfo 库查看
from zoneinfo import available_timezones
# available_timezones() 返回一个 set，包含所有安装的时区名称
tz_list = sorted(available_timezones())
print(tz_list[:10])  # 打印前 10 个示例
```


### 时间戳解析

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo  # Python 3.9+

ts = 1_656_000_000

# 1 直接用固定偏移——开销小
dt_utc = datetime.fromtimestamp(ts, tz=timezone.utc)
print(dt_utc.isoformat())  
# 2022-06-17T05:40:00+00:00

# 2 用 IANA 时区——考虑夏令时（DST）
dt_tokyo = datetime.fromtimestamp(ts, tz=ZoneInfo("Asia/Tokyo"))
print(dt_tokyo.isoformat())  
# 2022-06-17T14:40:00+09:00


# 3 毫秒级
ms_ts = 1_656_000_000_000  
dt = datetime.fromtimestamp(ms_ts / 1000, tz=timezone.utc)
print(dt.isoformat())  
# 2022-06-17T05:40:00+00:00
```

> [!note] 不传 `tz` 参数时，`fromtimestamp` 会根据系统本地时区（如 Asia/Tokyo）计算，但生成的是**naive**（`tzinfo=None`）的 `datetime`。