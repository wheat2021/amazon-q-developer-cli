# Agent 格式

每个 agent 的配置文件都是一个 JSON 文件。文件名（不包括 `.json` 扩展名）将成为 agent 的名称。它包含实例化和运行 agent 所需的配置。

每个 agent 配置文件可以包含以下部分：

- [`name`](#name-field) — agent 的名称（可选，如果未指定则从文件名派生）。
- [`description`](#description-field) — agent 的描述。
- [`prompt`](#prompt-field) — agent 的高级上下文。
- [`mcpServers`](#mcpservers-field) — agent 可以访问的 MCP 服务器。
- [`tools`](#tools-field) — agent 可用的工具。
- [`toolAliases`](#toolaliases-field) — 用于处理命名冲突的工具名称重映射。
- [`allowedTools`](#allowedtools-field) — 可以在不提示的情况下使用的工具。
- [`toolsSettings`](#toolssettings-field) — 特定工具的配置。
- [`resources`](#resources-field) — agent 可用的资源。
- [`hooks`](#hooks-field) — 在特定触发点运行的命令。
- [`useLegacyMcpJson`](#uselegacymcpjson-field) — 是否包含传统 MCP 配置。

## Name 字段

`name` 字段指定 agent 的名称。这用于标识和显示目的。

```json
{
  "name": "aws-expert"
}
```

## Description 字段

`description` 字段提供 agent 功能的描述。这主要用于人类可读性，帮助用户区分不同的 agent。

```json
{
  "description": "An agent specialized for AWS infrastructure tasks"
}
```

## Prompt 字段

`prompt` 字段旨在为 agent 提供高级上下文，类似于系统提示。

```json
{
  "prompt": "You are an expert AWS infrastructure specialist"
}
```

## McpServers 字段

`mcpServers` 字段指定 agent 可以访问的 Model Context Protocol (MCP) 服务器。每个服务器都用命令和可选参数定义。

```json
{
  "mcpServers": {
    "fetch": {
      "command": "fetch3.1",
      "args": []
    },
    "git": {
      "command": "git-mcp",
      "args": [],
      "env": {
        "GIT_CONFIG_GLOBAL": "/dev/null"
      },
      "timeout": 120000
    }
  }
}
```

每个 MCP 服务器配置可以包含：
- `command`（必需）：启动 MCP 服务器要执行的命令
- `args`（可选）：传递给命令的参数
- `env`（可选）：为服务器设置的环境变量
- `timeout`（可选）：每个 MCP 请求的超时时间（毫秒）（默认：120000）

## Tools 字段

`tools` 字段列出 agent 可能使用的所有工具。工具包括内置工具和来自 MCP 服务器的工具。

- 内置工具通过名称指定（例如，`fs_read`、`execute_bash`）
- MCP 服务器工具以 `@` 开头，后跟服务器名称（例如，`@git`）
- 要指定来自 MCP 服务器的特定工具，使用 `@server_name/tool_name`
- 使用 `*` 作为特殊通配符来包含所有可用工具（内置和来自 MCP 服务器的）
- 使用 `@builtin` 包含所有内置工具
- 使用 `@server_name` 包含来自特定 MCP 服务器的所有工具

```json
{
  "tools": [
    "fs_read",
    "fs_write",
    "execute_bash",
    "@git",
    "@rust-analyzer/check_code"
  ]
}
```

要包含所有可用工具，您可以简单地使用：

```json
{
  "tools": ["*"]
}
```

## ToolAliases 字段

`toolAliases` 字段是一个高级功能，允许您重新映射工具名称。这主要用于解决来自不同 MCP 服务器的工具之间的命名冲突，或为特定工具创建更直观的名称。

例如，如果 `@github-mcp` 和 `@gitlab-mcp` 服务器都提供名为 `get_issues` 的工具，您将遇到命名冲突。您可以使用 `toolAliases` 来消除歧义：

```json
{
  "toolAliases": {
    "@github-mcp/get_issues": "github_issues",
    "@gitlab-mcp/get_issues": "gitlab_issues"
  }
}
```

使用此配置，这些工具将作为 `github_issues` 和 `gitlab_issues` 提供给 agent，而不是在 `get_issues` 上发生冲突。

您还可以使用别名为常用工具创建更短或更直观的名称：

```json
{
  "toolAliases": {
    "@aws-cloud-formation/deploy_stack_with_parameters": "deploy_cf",
    "@kubernetes-tools/get_pod_logs_with_namespace": "pod_logs"
  }
}
```

键是原始工具名称（MCP 工具包括服务器前缀），值是要使用的新名称。

## AllowedTools 字段

`allowedTools` 字段指定哪些工具可以在不提示用户许可的情况下使用。这是一个安全功能，有助于防止未经授权的工具使用。

```json
{
  "allowedTools": [
    "fs_read",
    "@git/git_status",
    "@fetch"
  ]
}
```

您可以允许：
- 按名称指定的内置工具（例如，`"fs_read"`）
- 使用 `@server_name/tool_name` 的特定 MCP 工具（例如，`"@git/git_status"`）
- 使用 `@server_name` 的 MCP 服务器的所有工具（例如，`"@fetch"`）

与 `tools` 字段不同，`allowedTools` 字段不支持 `"*"` 通配符来允许所有工具。要允许特定工具，您必须单独列出它们或使用 `@server_name` 语法的服务器级通配符。

## ToolsSettings 字段

`toolsSettings` 字段为特定工具提供配置。每个工具都可以有自己独特的配置选项。

```json
{
  "toolsSettings": {
    "fs_write": {
      "allowedPaths": ["~/**"]
    },
    "@git/git_status": {
      "git_user": "$GIT_USER"
    }
  }
}
```

有关内置工具配置选项，请参阅[内置工具文档](./built-in-tools.md)。

## Resources 字段

`resources` 字段为 agent 提供对本地资源的访问。目前，仅支持文件资源，所有资源路径必须以 `file://` 开头。

```json
{
  "resources": [
    "file://AmazonQ.md",
    "file://README.md",
    "file://.amazonq/rules/**/*.md"
  ]
}
```

资源可以包括：
- 特定文件
- 多个文件的 Glob 模式
- 绝对或相对路径

## Hooks 字段

`hooks` 字段定义在特定触发点运行的命令。这些命令的输出会添加到 agent 的上下文中。

```json
{
  "hooks": {
    "agentSpawn": [
      {
        "command": "git status",
      }
    ],
    "userPromptSubmit": [
      {
        "command": "ls -la",
      }
    ]
  }
}
```

每个 hook 定义包含：
- `command`（必需）：要执行的命令

可用的 hook 触发器：
- `agentSpawn`：在 agent 初始化时触发
- `userPromptSubmit`：在用户提交消息时触发

## UseLegacyMcpJson 字段

`useLegacyMcpJson` 字段确定是否包含在传统 MCP 配置文件中定义的 MCP 服务器（全局的 `~/.aws/amazonq/mcp.json` 和工作区的 `cwd/.amazonq/mcp.json`）。

```json
{
  "useLegacyMcpJson": true
}
```

当设置为 `true` 时，agent 将可以访问全局配置中定义的所有 MCP 服务器，以及在 agent 的 `mcpServers` 字段中定义的服务器。

## 完整示例

这是一个 agent 配置文件的完整示例：

```json
{
  "name": "aws-rust-agent",
  "description": "A specialized agent for AWS and Rust development tasks",
  "mcpServers": {
    "fetch": {
      "command": "fetch3.1",
      "args": []
    },
    "git": {
      "command": "git-mcp",
      "args": []
    }
  },
  "tools": [
    "fs_read",
    "fs_write",
    "execute_bash",
    "use_aws",
    "@git",
    "@fetch/fetch_url"
  ],
  "toolAliases": {
    "@git/git_status": "status",
    "@fetch/fetch_url": "get"
  },
  "allowedTools": [
    "fs_read",
    "@git/git_status"
  ],
  "toolsSettings": {
    "fs_write": {
      "allowedPaths": ["src/**", "tests/**", "Cargo.toml"]
    },
    "use_aws": {
      "allowedServices": ["s3", "lambda"]
    }
  },
  "resources": [
    "file://README.md",
    "file://docs/**/*.md"
  ],
  "hooks": {
    "agentSpawn": [
      {
        "command": "git status"
      }
    ],
    "userPromptSubmit": [
      {
        "command": "ls -la"
      }
    ]
  },
  "useLegacyMcpJson": true
}
```
