---
tags:
  - IT/蓝队
---
在使用 Google 搜索时，**搜索结果页面的 URL**通常遵循以下格式：

### ✅ **基本格式**

```
<https://www.google.com/search?q=><搜索关键词>&[其他参数]

```

**关键参数：**

- `q=` → 搜索关键词（例如 `q=smartphone`）
- `hl=` → 界面语言（例如 `hl=en`）
- `source=` → 来源（如 `source=hp` 表示首页搜索）
- `ei=` → 编码的请求 ID（用于跟踪）
- `ved=` → 点击来源或搜索行为编码
- `oq=` → 原始查询（original query）
- `aqs=` → 自动完成建议相关参数
- `uact=` → 用户行为标识（如点击类型）

### ✅ **常见情况：Google 跳转 URL**

当你点击搜索结果时，浏览器通常先访问一个 Google 跳转链接，然后再跳转到目标网站。这个跳转 URL 一般格式是：

```
<https://www.google.com/url?q=><目标URL>&sa=<参数>&ved=<参数>&usg=<跟踪ID>
```

- `q=` → 目标网站的真实 URL
- `sa=` → 来源标识（search action）
- `ved=` → 编码的点击来源信息（广告、自然搜索等）
- `usg=` → Google 生成的跟踪签名，用于防止篡改

#### ✅ 常见 `sa=` 类型

|参数值|含义（推测）|
|---|---|
|`sa=t`|点击了**普通搜索结果**（text result）|
|`sa=i`|点击的是**图像搜索结果**（image result）|
|`sa=N`|点击的是**新闻结果**（News tab）|
|`sa=X`|点击的是其它**特殊栏目**（如地图、购物等）|
|`sa=U`|**URL 被引用**或处理来源为用户点击链接|