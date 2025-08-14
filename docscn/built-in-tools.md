# 内置工具

Amazon Q CLI 包含几个 agent 可以使用的内置工具。本文档描述了每个工具及其配置选项。

- [`execute_bash`](#execute_bash-tool) — 执行 shell 命令。
- [`fs_read`](#fs_read-tool) — 读取文件、目录和图像。
- [`fs_write`](#fs_write-tool) — 创建和编辑文件。
- [`report_issue`](#report_issue-tool) — 打开 GitHub 问题模板。
- [`knowledge`](#knowledge-tool) — 在知识库中存储和检索信息。
- [`thinking`](#thinking-tool) — 内部推理机制。
- [`use_aws`](#use_aws-tool) — 进行 AWS CLI API 调用。

## Execute_bash 工具

执行指定的 bash 命令。

### 配置

```json
{
  "toolsSettings": {
    "execute_bash": {
      "allowedCommands": ["git status", "git fetch"],
      "deniedCommands": ["git commit .*", "git push .*"],
      "allowReadOnly": true
    }
  }
}
```

### 配置选项

| 选项 | 类型 | 默认值 | 描述                                                                              |
|--------|------|---------|------------------------------------------------------------------------------------------|
| `allowedCommands` | 字符串数组 | `[]` | 允许在不提示的情况下执行的特定命令列表。支持正则表达式格式。注意输入的正则表达式会用 \A 和 \z 锚定 |
| `deniedCommands` | 字符串数组 | `[]` | 被拒绝的特定命令列表。支持正则表达式格式。注意输入的正则表达式会用 \A 和 \z 锚定。拒绝规则在允许规则之前评估 |
| `allowReadOnly` | 布尔值 | `true` | 是否允许在不提示的情况下执行只读命令                                    |

## Fs_read 工具

用于读取文件、目录和图像的工具。

### 配置

```json
{
  "toolsSettings": {
    "fs_read": {
      "allowedPaths": ["~/projects", "./src/**"],
      "deniedPaths": ["/some/denied/path/", "/another/denied/path/**/file.txt"]
    }
  }
}
```

### 配置选项

| 选项 | 类型 | 默认值 | 描述 |
|--------|------|---------|-------------|
| `allowedPaths` | 字符串数组 | `[]` | 可以在不提示的情况下读取的路径列表。支持 glob 模式 |
| `deniedPaths` | 字符串数组 | `[]` | 被拒绝的路径列表。支持 glob 模式。拒绝规则在允许规则之前评估 |

## Fs_write 工具

用于创建和编辑文件的工具。

### 配置

```json
{
  "toolsSettings": {
    "fs_write": {
      "allowedPaths": ["~/projects/output.txt", "./src/**"],
      "deniedPaths": ["/some/denied/path/", "/another/denied/path/**/file.txt"]
    }
  }
}
```

### 配置选项

| 选项 | 类型 | 默认值 | 描述 |
|--------|------|---------|-------------|
| `allowedPaths` | 字符串数组 | `[]` | 可以在不提示的情况下写入的路径列表。支持 glob 模式 |
| `deniedPaths` | 字符串数组 | `[]` | 被拒绝的路径列表。支持 glob 模式。拒绝规则在允许规则之前评估 |

## Report_issue 工具

打开浏览器到预填充的 GitHub 问题模板，用于报告聊天问题、错误或功能请求。

此工具没有配置选项。

## Knowledge 工具

在跨聊天会话的知识库中存储和检索信息。为文件、目录和文本内容提供语义搜索功能。

此工具没有配置选项。

## Thinking 工具

一种内部推理机制，通过将复杂任务分解为原子操作来提高复杂任务的质量。

此工具没有配置选项。

## Use_aws 工具

使用指定的服务、操作和参数进行 AWS CLI API 调用。

### 配置

```json
{
  "toolsSettings": {
    "use_aws": {
      "allowedServices": ["s3", "lambda", "ec2"],
      "deniedServices": ["eks", "rds"]
    }
  }
}
```

### 配置选项

| 选项 | 类型 | 默认值 | 描述 |
|--------|------|---------|-------------|
| `allowedServices` | 字符串数组 | `[]` | 可以在不提示的情况下访问的 AWS 服务列表 |
| `deniedServices` | 字符串数组 | `[]` | 要拒绝的 AWS 服务列表。拒绝规则在允许规则之前评估 |

## 在 Agent 配置中使用工具设置

工具设置在 agent 配置文件的 `toolsSettings` 部分中指定。每个工具的设置使用工具名称作为键来指定。

对于 MCP 服务器工具，使用格式 `@server_name/tool_name` 作为键：

```json
{
  "toolsSettings": {
    "fs_write": {
      "allowedPaths": ["~/projects"]
    },
    "@git/git_status": {
      "git_user": "$GIT_USER"
    }
  }
}
```

## 工具权限

工具可以在 agent 配置的 `allowedTools` 部分中明确允许：

```json
{
  "allowedTools": [
    "fs_read",
    "knowledge",
    "@git/git_status"
  ]
}
```

如果工具不在 `allowedTools` 列表中，当使用该工具时会提示用户获得权限。

一些工具有默认的权限行为：
- `fs_read` 和 `report_issue` 默认受信任
- `execute_bash`、`fs_write` 和 `use_aws` 默认提示权限，但可以配置为允许特定的命令/路径/服务
