---
title: File System
type: concept
created: 2026-04-06
related:
  - "[[ThreadItem]]"
---

# File System 文件系统

Codex 提供了文件系统操作能力。

## 文件操作

| 操作 | 请求 | 响应 |
|------|------|------|
| 读取文件 | `FsReadFileParams` | `FsReadFileResponse` |
| 写入文件 | `FsWriteFileParams` | `FsWriteFileResponse` |
| 删除文件 | `FsRemoveParams` | `FsRemoveResponse` |
| 创建目录 | `FsCreateDirectoryParams` | `FsCreateDirectoryResponse` |
| 读取目录 | `FsReadDirectoryParams` | `FsReadDirectoryResponse` |
| 复制文件 | `FsCopyParams` | `FsCopyResponse` |
| 获取元数据 | `FsGetMetadataParams` | `FsGetMetadataResponse` |

## 文件更改表示

```typescript
type FileUpdateChange = {
  path: string;
  kind: PatchChangeKind;
  beforeHash?: string;
  afterHash?: string;
  hunks?: Array<DiffHunk>;
};
```

## PatchChangeKind

```typescript
type PatchChangeKind = 
  | "fileCreated"
  | "fileUpdated"
  | "fileDeleted"
  | "fileRenamed";
```

## 通知

- `FsChangedNotification` - 文件系统变更通知
- `FileChangeOutputDeltaNotification` - 文件变更输出通知

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/FsReadFileParams.ts`
- 详见: [[openaicodex]]
