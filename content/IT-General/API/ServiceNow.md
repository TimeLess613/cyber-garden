---
tags:
  - IT/API
---
- ticket 工单界面的 description，换行符要用 `<be>`。`\n` 和 `\\n` 等都无效。
- 当传入 list 时，各个元素中的特殊值会被编码（如 `<>`，似乎就是用 Unicode）。而直接传入字符串的话则不会。


comment/worknote 默认用文本，如果希望 HTML 格式，则用 `[code] ... [/code]`包围（甚至可以省略 `[/code]`）


关于样式，在 ServiceNow SIR 的外部网页中推荐方式——用内联样式写入标签（外部网页不解析 style 块）：

|方式|是否推荐|理由|
|---|---|---|
|`<style>...</style>`|❌ 不推荐|很可能被过滤，不显示效果|
|`<div style="...">`|✅ 推荐|可被正确渲染、适合 API comment|
|`<table class="header">`|⚠️ 需要 `<style>` 定义才能生效，危险||
|`<table style="...">`|✅ 推荐|最安全有效方式，保留排版和样式|