---
type: monthly-review
month: YYYY-MM
created: {{date}}
---

# {{month}} 月度复盘

## 本月关键词（一句话）
<!-- 这个月最关键的一件事是什么 -->

## 数字回顾

> 用 Dataview 自动抓任务进度（按你的实际目录调整路径）

```dataview
TABLE WITHOUT ID file.link AS "任务", status, due
FROM "20-Products"
WHERE due >= date("YYYY-MM-01") AND due <= date("YYYY-MM-31")
SORT status, due
```

## 完成 vs 计划

### ✅ 完成了什么
-

### ❌ 没完成 / 推迟
-

### 🔍 没完成的原因
- 客观（外部因素）：
- 主观（自己的问题）：

## 月度自检

逐条回答本月目标的完成情况：

1.
2.
3.

## 本月新沉淀的知识

```dataview
TABLE WITHOUT ID file.link AS "笔记", topic
FROM "10-Knowledge"
WHERE created >= date("YYYY-MM-01") AND created <= date("YYYY-MM-31")
SORT created DESC
```

## 下个月重点

1.
2.
3.

## 反思与调整

<!-- 本月暴露出来的问题，下月怎么调 -->

## 关联

- 上月复盘：[[YYYY-MM-月度复盘]]
