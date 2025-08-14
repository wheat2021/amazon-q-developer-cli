# 默认 Agent 行为

当没有配置特定 agent 或找不到指定的 agent 时，Q CLI 遵循回退层次结构来确定使用哪个 agent。

## Agent 选择优先级

Q CLI 按以下优先级顺序选择 agent：

### 1. 命令行指定的 Agent
通过启动 Q CLI 时的 `--agent` 标志指定的 agent：

```bash
q chat --agent my-custom-agent
```

如果此 agent 存在，将使用它。如果不存在，Q CLI 将显示错误并回退到下一个选项。

### 2. 用户定义的默认 Agent
通过设置系统配置的默认 agent：

```bash
q settings chat.defaultAgent my-preferred-agent
```

此设置存储在您的 Q CLI 配置中，将在所有会话中使用，除非被 `--agent` 标志覆盖。

如果找不到配置的默认 agent，Q CLI 将显示错误并回退到内置默认值。

### 3. 内置默认 Agent
如果没有指定或找到 agent，Q CLI 使用具有以下配置的内置默认 agent：

```json
{
  "name": "default",
  "description": "Default agent",
  "tools": ["*"],
  "allowedTools": ["fs_read"],
  "resources": [
    "file://AmazonQ.md",
    "file://README.md", 
    "file://.amazonq/rules/**/*.md"
  ],
  "useLegacyMcpJson": true
}
```

## 内置默认 Agent 详细信息

内置默认 agent 提供：

### 可用工具
- **所有工具**：使用 `"*"` 通配符包含所有内置工具和 MCP 服务器工具

### 受信任的工具
- **仅 fs_read**：只有 `fs_read` 工具是预先批准的，不会提示权限
- 所有其他工具在执行前需要用户确认

### 默认资源
agent 自动在其上下文中包含这些文件（如果存在）：
- `AmazonQ.md` - Amazon Q 文档或笔记
- `README.md` - 项目自述文件
- `.amazonq/rules/**/*.md` - `.amazonq/rules/` 目录和子目录中的任何 markdown 文件

### 传统 MCP 支持
- **已启用**：默认 agent 包含来自传统全局配置文件（`~/.aws/amazonq/mcp.json`）的 MCP 服务器

## 错误消息

当发生 agent 回退时，您将看到信息性消息：

### Agent 未找到
```
Error: no agent with name my-agent found. Falling back to user specified default
```

### 用户默认未找到
```
Error: user defined default my-default not found. Falling back to in-memory default
```

## 自定义默认行为

### 设置用户默认 Agent
为避免使用内置默认值，配置您的首选 agent：

```bash
q settings chat.defaultAgent my-preferred-agent
```

### 为特定会话覆盖
使用 `--agent` 标志为特定会话指定 agent：

```bash
q chat --agent specialized-agent
```

### 创建自定义默认值
您可以通过在以下位置放置名为 `q_cli_default` 的 agent 文件来创建自己的"默认" agent：
- `.aws/amazonq/cli-agents/`（本地）
- `~/.aws/amazonq/cli-agents/`（全局）

这将覆盖内置默认 agent 配置。

## 最佳实践

1. **设置用户默认值**：配置与您典型工作流匹配的默认 agent
2. **使用描述性名称**：选择清楚表明其用途的 agent 名称
3. **测试 agent 可用性**：确保您的默认 agent 存在且可访问
4. **记录团队 agent**：如果与团队共享 agent，记录哪些 agent 应用于不同任务

## 示例工作流

```bash
# 设置首选默认 agent
q settings chat.defaultAgent development-helper

# 使用默认 agent（development-helper）
q chat

# 为特定任务覆盖
q chat --agent aws-specialist

# 如果 development-helper 不存在，回退到内置默认值
# 并显示警告消息
```
