# 将 Profile 迁移到 Agent

所有全局 profile（在 `~/.aws/amazonq/profiles/` 下创建）支持在初始启动时自动迁移到 `~/.aws/amazonq/cli-agents/` 下的全局 agent 目录。

如果您在 `.amazonq/mcp.json` 下定义了本地 MCP 配置，那么您可以选择将此配置添加到全局 agent，或创建新的工作区 agent。

## 创建新的工作区 Agent

工作区 agent 在当前工作目录内的 `.amazonq/cli-agents/` 下管理。

您可以使用 `q agent create --name my-agent -d .` 创建新的工作区 agent。

## 全局上下文

之前在 `~/.aws/amazonq/global_context.json` 下配置的全局上下文不再受支持。全局上下文需要手动添加到 agent 中（请参阅下面的部分）。

## MCP 服务器

agent 配置支持与之前配置相同的 MCP 格式。

有关更多详细信息，请参阅 [agent 格式文档](./agent-format.md#mcpservers-field)。

## 上下文文件

上下文文件现在是[文件 URI](https://en.wikipedia.org/wiki/File_URI_scheme)，并在 `"resources"` 字段下配置。

Profile 示例：
```json
{
    "paths": [
        "~/my-files/**/*.txt"
    ]
}
```

Agent 的相同示例：
```json
{
    "resources": [
        "file://~/my-files/**/*.txt"
    ]
}
```

## Hook

Hook 触发器已更新：
- 不再需要 Hook 名称
- `conversation_start` 现在是 `agentSpawn`
- `per_prompt` 现在是 `userPromptSubmit`

有关更多详细信息，请参阅 [agent 格式文档](./agent-format.md#hooks-field)。

Profile 示例：
```json
{
    "hooks": {
        "sleep_conv_start": {
            "trigger": "conversation_start",
            "type": "inline",
            "disabled": false,
            "timeout_ms": 30000,
            "max_output_size": 10240,
            "cache_ttl_seconds": 0,
            "command": "echo Conversation start hook"
        },
        "hello_world": {
            "trigger": "per_prompt",
            "type": "inline",
            "disabled": false,
            "timeout_ms": 30000,
            "max_output_size": 10240,
            "cache_ttl_seconds": 0,
            "command": "echo Per prompt hook"
        }
    }
}
```

Agent 的相同示例：
```json
{
    "hooks": {
        "userPromptSubmit": [
            {
                "command": "echo Per prompt hook",
                "timeout_ms": 30000,
                "max_output_size": 10240,
                "cache_ttl_seconds": 0
            }
        ],
        "agentSpawn": [
            {
                "command": "echo Conversation start hook",
                "timeout_ms": 30000,
                "max_output_size": 10240,
                "cache_ttl_seconds": 0
            }
        ]
    }
}
```
