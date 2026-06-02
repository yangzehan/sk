---
title: Protocol
type: concept
created: 2026-04-06
related:
  - "[[Core Primitives]]"
  - "[[Events]]"
---

# Protocol 通信协议

Codex App-Server 使用 JSON-RPC 2.0 消息进行双向通信。

## 支持的传输方式

| 传输方式 | 地址格式 | 说明 |
|---------|---------|------|
| stdio | `--listen stdio://` | 默认方式，newline-delimited JSON (JSONL) |
| websocket | `--listen ws://IP:PORT` | 实验性/不支持生产环境 |

## Websocket 安全

- 回环 websocket (`ws://127.0.0.1:PORT`) 适用于本地和 SSH 端口转发
- 非回环 websocket 默认允许未认证连接
- 支持的认证模式：
  - `capability-token`
  - `signed-bearer-token`

## 日志配置

- `RUST_LOG` 控制日志过滤/详细程度
- `LOG_FORMAT=json` 以 JSON 格式输出日志

## 背压处理

- 服务器使用有界队列处理传输入口、请求处理和出站写入
- 请求入口饱和时，新请求会被拒绝，错误码 `-32001`

## 来源

- 详见原始文档: [[openaicodex]]
