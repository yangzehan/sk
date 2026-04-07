---
title: Codex Schema Index
type: index
created: 2026-04-06
---

# Codex App-Server Schema

Codex App-Server 的 TypeScript 类型定义，来源于 `codex-app-server-schema` 仓库。

## 核心概念

| 类型 | 说明 | 文档 |
|------|------|------|
| Thread | 对话线程 | [[Thread]] |
| Turn | 对话轮次 | [[Turn]] |
| ThreadItem | 轮次中的项目 | [[ThreadItem]] |
| Command Execution | 命令执行 | [[Command Execution]] |
| File System | 文件系统操作 | [[File System]] |
| MCP Integration | MCP 集成 | [[MCP Integration]] |
| Codex CLI | 命令行工具 | [[Codex CLI]] |

## 请求/响应

### Thread 操作
- `ThreadStartParams` / `ThreadStartResponse` - 启动线程
- `ThreadReadParams` / `ThreadReadResponse` - 读取线程
- `ThreadResumeParams` / `ThreadResumeResponse` - 恢复线程
- `ThreadArchiveParams` / `ThreadArchiveResponse` - 归档线程
- `ThreadForkParams` / `ThreadForkResponse` - Fork 线程

### 命令执行
- `CommandExecParams` - 执行命令请求
- `CommandExecResponse` - 执行命令响应

### 文件系统
- `FsReadFileParams` / `FsReadFileResponse`
- `FsWriteFileParams` / `FsWriteFileResponse`
- `FsCreateDirectoryParams` / `FsCreateDirectoryResponse`

### MCP
- `ListMcpServerStatusParams` / `ListMcpServerStatusResponse`

### 技能
- `SkillsListParams` / `SkillsListResponse`
- `SkillInterface` - 技能接口定义
- `SkillMetadata` - 技能元数据

### 审查
- `CommandExecutionRequestApprovalParams` / `CommandExecutionRequestApprovalResponse`
- `FileChangeRequestApprovalParams` / `FileChangeRequestApprovalResponse`

## 通知

- `ThreadStartedNotification`
- `TurnStartedNotification`
- `ItemStartedNotification`
- `CommandExecutionOutputDeltaNotification`
- `FileChangeOutputDeltaNotification`
- `ErrorNotification`

## 来源

- 源文件: `raw/codex-app-server-schema/ts/v2/`
