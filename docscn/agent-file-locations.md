# Agent 文件位置

Agent 配置文件可以放置在两个不同的位置，允许工作区特定和用户范围的 agent 配置。

## 本地 Agent（工作区特定）

本地 agent 存储在当前工作目录下：

```
.amazonq/cli-agents/
```

这些 agent 特定于当前工作区或项目，只有在从该目录或其子目录运行 Q CLI 时才可用。

**示例结构：**
```
my-project/
├── .amazonq/
│       └── cli-agents/
│           ├── dev-agent.json
│           └── aws-specialist.json
└── src/
    └── main.py
```

## 全局 Agent（用户范围）

全局 agent 存储在您的主目录下：

```
~/.aws/amazonq/cli-agents/
```

注意：对于全局可用的 agent，`amazonq` 目录在 `.aws` 文件夹中。

这些 agent 在使用 Q CLI 时可从任何目录访问。

**示例结构：**
```
~/.aws/amazonq/cli-agents/
├── general-assistant.json
├── code-reviewer.json
└── documentation-writer.json
```

## Agent 优先级

当 Q CLI 查找 agent 时，它遵循以下优先级顺序：

1. **本地优先**：检查当前工作目录中的 `.aws/amazonq/cli-agents/`
2. **全局回退**：如果本地未找到，检查主目录中的 `~/.aws/amazonq/cli-agents/`

## 命名冲突

如果本地和全局目录都包含同名的 agent，**本地 agent 优先**。当发生这种情况时，Q CLI 将显示警告消息：

```
WARNING: Agent conflict for my-agent. Using workspace version.
```

同名的全局 agent 将被忽略，优先使用本地版本。

## 最佳实践

### 使用本地 Agent 用于：
- 项目特定配置
- 需要访问特定项目文件或工具的 agent
- 具有独特要求的开发环境
- 通过版本控制与团队成员共享 agent 配置

### 使用全局 Agent 用于：
- 跨多个项目使用的通用 agent
- 个人生产力 agent
- 不需要项目特定上下文的 agent
- 常用的开发工具和工作流

## 示例用法

为当前项目创建本地 agent：

```bash
mkdir -p .amazonq/cli-agents
cat > .amazonq/cli-agents/project-helper.json << 'EOF'
{
  "description": "Helper agent for this specific project",
  "tools": ["fs_read", "fs_write", "execute_bash"],
  "resources": [
    "file://README.md",
    "file://docs/**/*.md"
  ]
}
EOF
```

创建在任何地方都可用的全局 agent：

```bash
mkdir -p ~/.aws/amazonq/cli-agents
cat > ~/.aws/amazonq/cli-agents/general-helper.json << 'EOF'
{
  "description": "General purpose assistant",
  "tools": ["*"],
  "allowedTools": ["fs_read"]
}
EOF
```

## 目录创建

如果全局 agent 目录（`~/.aws/amazonq/cli-agents/`）不存在，Q CLI 将自动创建它。但是，如果您想使用本地 agent，需要在工作区中手动创建本地 agent 目录（`.amazonq/cli-agents/`）。
