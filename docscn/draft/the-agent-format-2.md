# Agent 格式

每个 agent 的配置文件称为其_清单_。它以 JSON 格式编写。它包含实例化和运行 agent 所需的元数据和配置。

每个清单文件由以下部分组成：

- [`name`](#the-name-field) --- agent 的名称。
- [`version`](#the-version-field) --- agent 的版本。
- [`description`](#the-description-field) --- agent 的描述。
- [`model`](#the-model-field) --- agent 使用的模型。
- [`inputSchema`](#the-input-schema-field) --- agent 的输入模式。
- [`mcpServers`](#the-mcp-servers-field) --- agent 可以访问的 MCP 服务器。
- [`tools`](#the-tools-field) --- agent 可用的工具。
- [`allowedTools`](#the-allowed-tools-field) --- 可以在不提示的情况下使用的工具。
- [`toolsSettings`](#the-tools-settings-field) --- 特定工具的配置。
- [`resources`](#the-resources-field) --- 声明性资源和上下文。

### `name` 字段

`name` 字段指定 agent 的名称。这用于标识和显示目的。

名称必须仅使用[字母数字]字符或 `-`，且不能为空。

- 只允许 ASCII 字符。
- 不要使用保留名称。
- 不要使用特殊的 Windows 名称，如 "nul"。
- 使用最多 64 个字符的长度。

### `version` 字段

`version` 字段根据 [SemVer] 规范格式化：

版本必须有三个数字部分，
主版本、次版本和补丁版本。

可以在破折号后添加预发布部分，如 `1.0.0-alpha`。
预发布部分可以用句点分隔以区分单独的
组件。数字组件将使用数字比较，而
其他所有内容将按字典顺序比较。
例如，`1.0.0-alpha.11` 高于 `1.0.0-alpha.4`。

### `description` 字段

`description` 字段提供 agent 功能的描述，供人类和机器阅读。重要的是描述要简洁地定义 agent 行为，因为这些描述在用作工具时会占用 LLM 上下文。

### `model` 字段

`model` 字段指定 agent 应使用的语言模型。

模型标识符可以在 [AWS Bedrock 文档](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)中找到。

目前，支持的两个模型是

1. `anthropic.claude-3-7-sonnet-20250219-v1:0`
2. `anthropic.claude-sonnet-4-20250514-v1:0`

### `inputSchema` 字段

`inputSchema` 字段定义当 agent 用作工具时的输入参数。此模式采用 JSON 模式的形式。因为 agent 可以作为 MCP 工具执行，这直接与 MCP 工具本身的输入模式对齐。输入参数可以使用以下语法在 agent 中模板化：

```
${my-input-parameter}
```

请注意，这与环境变量使用的语法相同，这样，环境变量充当用户定义的输入，而输入参数服务于机器输入。

```json
{
  "inputSchema": {
    "type": "object",
    "properties": {
      "model-to-use": {
        "type": "string",
        "description": "The model to use for this agent invocation."
      }
    },
    "required": ["model-to-use"]
  }
}
```

然后您可能会这样做：

```json
{
  "name": "my-agent",
  "model": ${model-to-use},
}
```

### `mcpServers` 字段

`mcpServers` 字段指定 agent 可以访问的 MCP 服务器。MCP 服务器可以是本地的或远程的。

**本地服务器**用命令和传输配置定义：

```json
{
  "type": "object",
  "properties": {
    "command": {
      "type": "string",
      "description": "The command to execute to start the MCP server"
    },
    "args": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Arguments to pass to the command"
    },
    "transport": {
      "type": "string",
      "enum": ["stdio", "streamable-http"],
      "description": "The transport protocol to use"
    },
    "env": {
      "type": "object",
      "description": "Environment variables to set for the server"
    }
  },
  "required": ["command", "transport"]
}
```

**远程服务器**用 URL 定义：

```json
{
  "type": "object",
  "properties": {
    "url": {
      "type": "string",
      "format": "uri",
      "description": "The URL of the remote MCP server"
    },
    "headers": {
      "type": "object",
      "description": "HTTP headers to include in requests"
    },
    "auth": {
      "type": "object",
      "description": "Authentication configuration"
    }
  },
  "required": ["url"]
}
```

**完整示例：**

```json
{
  "mcpServers": {
    "fetch": {
      "command": "fetch",
      "args": [],
      "transport": "stdio"
    },
    "git": {
      "command": "git-mcp",
      "transport": "streamable-http",
      "env": {
        "GIT_CONFIG_GLOBAL": "/dev/null"
      }
    },
    "github-mcp": {
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    }
  }
}
```

### `tools` 字段

`tools` 字段列出 agent 可能使用的所有工具。来自 MCP 服务器的工具以 `@` 为前缀，而 agent 本身以 `#` 为前缀。

原生工具可以选择性地以 `@native` 为前缀。

**注意：所有原生工具的列表可以在[这里](./tools.md)找到。**

```json
{
  "tools": [
    "fs-read",
    "@native/execute-bash",
    "@git",
    "@my-enterprise-mcp/read-internal-website",
    "#my-workspace-agent"
  ]
}
```

### `allowedTools` 字段

`allowedTools` 字段指定哪些工具可以在不提示用户许可的情况下使用。允许的工具可以是工具的超集，因为此权限在工具执行时检查。

```json
{
  "allowedTools": [
    "fs-read",
    "@git/git-status",
    "@my-enterprise-mcp",
    "#my-workspace-agent"
  ]
}
```

### `toolsSettings` 字段

`toolsSettings` 字段为特定工具提供配置。每个工具都有独特的配置，只能通过检查工具的文档来了解。有关原生工具配置，请参阅[文档的这一部分](./tools.md)。

```json
{
  "toolsSettings": {
    "fs-write": {
      "allowedPaths": [".", "/var/www/**"]
    },
    "@my-enterprise-mcp.my-tool": {
      "some-configuration-value": true
    }
  }
}
```

### `resources` 字段

`resources` 字段定义为 agent 提供上下文的声明性资源。目前，这些资源由各个客户端实现唯一处理。

资源可以是简单的字符串以启用轻量级提示，但否则与 MCP 资源对齐。有关 MCP 资源的更多信息，请参阅 [MCP 文档](https://modelcontextprotocol.io/docs/concepts/resources)。

```json
{
  "resources": [
    "file://my-excellent-prompt.md",
    "file://${workspace}",
    "file://my-mcp-resource.json@builder-mcp"
  ]
}
```

## 完整示例

这是一个 agent 清单的完整示例：

```json
{
  "name": "rust-developer-agent",
  "version": "1.2.0",
  "description": "A specialized agent for Rust development tasks",
  "model": "anthropic.claude-sonnet-4-20250514-v1:0",
  "mcpServers": {
    "fetch": { "command": "fetch", "args": [], "transport": "stdio" },
    "git": { "command": "git-mcp", "transport": "streamable-http" },
    "rust-analyzer": { "command": "rust-analyzer-mcp", "transport": "stdio" }
  },
  "tools": [
    "fs-read",
    "fs-write",
    "execute-bash",
    "@git",
    "@rust-analyzer/check-code",
    "@rust-analyzer/format-code"
  ],
  "allowedTools": ["fs-read", "@git.git-status", "@rust-analyzer/check-code"],
  "toolsSettings": {
    "fs-write": {
      "allowedPaths": ["src/**", "tests/**", "Cargo.toml"]
    }
  },
  "resources": [
    "file://rust-style-guide.md",
    "file://${workspace}/README.md",
    "file://project-context.md@workspace-mcp"
  ],
  "inputSchema": {
    "type": "object",
    "properties": {
      "task": {
        "type": "string",
        "description": "The development task to perform"
      },
      "context": {
        "type": "object",
        "description": "Additional context for the task"
      }
    },
    "required": ["task"]
  }
}
```
