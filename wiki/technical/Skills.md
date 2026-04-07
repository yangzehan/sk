---
title: Skills System
type: concept
created: 2026-04-06
related:
  - "[[Apps]]"
  - "[[Events]]"
---

# Skills 技能系统

## 调用技能

通过在文本输入中包含 `$<skill-name>` 来调用技能。

```json
{
  "method": "turn/start",
  "params": {
    "input": [
      { "type": "text", "text": "$skill-creator Add a new skill" },
      { "type": "skill", "name": "skill-creator", "path": "/path/to/SKILL.md" }
    ]
  }
}
```

## 技能列表

使用 `skills/list` 获取可用的技能列表。

```json
{
  "method": "skills/list",
  "params": {
    "cwds": ["/path/to/project"]
  }
}
```

## 技能配置

使用 `skills/config/write` 启用或禁用技能。

## 技能变化通知

当监听的本地技能文件更改时，服务器会发出 `skills/changed` 通知。

## 来源

- 详见原始文档: [[openaicodex]]
