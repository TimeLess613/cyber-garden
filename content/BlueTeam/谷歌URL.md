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


#### 没有 Referer 报头

#GPT摘录 
HTTP 报头是否加上 Referer 通常遵循下面流程：
```
JS / HTML 触发请求
        ↓
创建 Request 对象
        ↓
计算 referrer（URL + policy）
        ↓
将 referrer 注入 request.headers
        ↓
交给网络栈（HTTP/TCP/TLS）
```

> Google 在搜索结果页 **明确指定 Referrer Policy**
> 等价效果是：Referrer-Policy: no-referrer（或 origin / strict-origin）
> 导致 HTTP 报头没有 Referer。

---

**关于“触发请求”**，有下面几个常见方式：

| 分类                 | 触发方式 / 示例                              | 是否需要用户操作 | Referer 的典型表现         | 行为特征（日志侧）              | 安全分析要点          |
| ------------------ | -------------------------------------- | -------- | --------------------- | ---------------------- | --------------- |
| **脚本显式触发**（浏览器扩展等） | `fetch()` / `XMLHttpRequest`           | 否        | 自动继承或可显式伪造            | 请求孤立；多为 API / JSON；可定时 | C2 / 扩展最常见来源    |
| **页面资源加载**         | `<img>` `<script>` `<link>` `<iframe>` | 否        | 当前文档 URL（受 policy 裁剪） | 请求呈链式；多 MIME 类型        | 若只剩单点请求 → 非页面行为 |
| **用户导航（点击）**       | `<a href>` `location.href`             | **是**    | 上一页面或被裁剪              | 紧随完整页面加载               | 无后续资源请求则不可信     |
| **表单提交**           | `<form action>`（POST）                  | **是**    | 上一页面                  | 明确业务字段；多为 POST         | 真实用户行为概率高       |
| **Service Worker** | `fetch` 事件中转                           | 否        | 可能为空或被重写              | 行为不直观；可缓存/重放           | 需结合 SW 注册信息判断   |
| **预取 / 预加载**       | `preload` `prefetch`                   | 否        | 常为空或 origin           | 无真实访问后续                | 易被误认为异常外联       |
| **浏览器内部通信**        | 更新 / Safe Browsing / Telemetry         | 否        | 通常为空                  | 固定域名 / 固定模式            | 一般可加入白名单        |
> 后3个是浏览器后台。


