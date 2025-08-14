# 知识管理

/knowledge 命令为 Amazon Q CLI 提供持久化知识库功能，允许您存储、搜索和管理跨聊天会话持续存在的上下文信息。

> 注意：这是一个测试功能，使用前必须启用。

## 入门指南

启用知识功能

知识功能是实验性的，默认情况下被禁用。使用以下命令启用：

`q settings chat.enableKnowledge true`

## 基本用法

启用后，您可以在聊天会话中使用 `/knowledge` 命令：

`/knowledge add myproject /path/to/project`
`/knowledge show`

## 命令

#### `/knowledge show`

显示知识库中的所有条目，包含详细信息如创建日期、项目数量和持久化状态。

#### `/knowledge add <name> <path> [--include pattern] [--exclude pattern]`

将文件或目录添加到知识库。系统将递归索引目录中所有支持的文件。

`/knowledge add "project-docs" /path/to/documentation`
`/knowledge add "config-files" /path/to/config.json`

**默认模式行为**

当您不指定 `--include` 或 `--exclude` 模式时，系统使用您配置的默认模式：

- 如果没有指定模式且没有配置默认值，则索引所有支持的文件
- 当没有指定 `--include` 时应用默认包含模式
- 当没有指定 `--exclude` 时应用默认排除模式
- 显式模式总是覆盖默认值

配置默认值的示例：
```bash
q settings knowledge.defaultIncludePatterns '["**/*.rs", "**/*.py"]'
q settings knowledge.defaultExcludePatterns '["target/**", "__pycache__/**"]'

# 这将使用默认模式
/knowledge add "my-project" /path/to/project

# 这将用显式模式覆盖默认值
/knowledge add "docs-only" /path/to/project --include "**/*.md"
```

**新功能：模式过滤**

您现在可以使用包含和排除模式控制哪些文件被索引：

`/knowledge add "rust-code" /path/to/project --include "*.rs" --exclude "target/**"`
`/knowledge add "docs" /path/to/project --include "**/*.md" --include "**/*.txt" --exclude "node_modules/**"`

模式示例：
- `*.rs` - 递归地所有目录中的所有 Rust 文件（等同于 `**/*.rs`）
- `**/*.py` - 递归地所有 Python 文件
- `target/**` - target 目录中的所有内容
- `node_modules/**` - node_modules 目录中的所有内容

支持的文件类型（扩展）：

- 文本文件：.txt、.log、.rtf、.tex、.rst
- Markdown：.md、.markdown、.mdx（现在支持！）
- JSON：.json（现在作为文本处理以获得更好的可搜索性）
- 配置：.ini、.conf、.cfg、.properties、.env
- 数据文件：.csv、.tsv
- Web 格式：.svg（基于文本）
- 代码文件：.rs、.py、.js、.jsx、.ts、.tsx、.java、.c、.cpp、.h、.hpp、.go、.rb、.php、.swift、.kt、.kts、.cs、.sh、.bash、.zsh、.html、.htm、.xml、.css、.scss、.sass、.less、.sql、.yaml、.yml、.toml
- 特殊文件：Dockerfile、Makefile、LICENSE、CHANGELOG、README（无扩展名的文件）

> 重要：不支持的文件会被索引但不提取文本内容。

#### `/knowledge remove <identifier>`

从知识库中删除条目。您可以按名称、路径或上下文 ID 删除。

`/knowledge remove "project-docs"` # 按名称删除
`/knowledge remove /path/to/old/project` # 按路径删除

#### `/knowledge update <path>`

使用指定路径的新内容更新现有的知识库条目。更新期间保留原始的包含/排除模式。

`/knowledge update /path/to/updated/project`

#### `/knowledge clear`

从知识库中删除所有条目。此操作需要确认且无法撤销。

您将被提示确认：

> ⚠️ 这将删除所有知识库条目。您确定吗？(y/N):

#### `/knowledge status`

查看后台索引操作的状态，包括进度和队列信息。

#### `/knowledge cancel [operation_id]`

取消后台操作。您可以通过 ID 取消特定操作，或者如果没有提供 ID 则取消所有操作。

`/knowledge cancel abc12345 # 取消特定操作`
`/knowledge cancel all # 取消所有操作`

## 配置

配置知识库行为：

`q settings knowledge.maxFiles 10000` # 每个知识库的最大文件数
`q settings knowledge.chunkSize 1024` # 处理的文本块大小
`q settings knowledge.chunkOverlap 256` # 块之间的重叠
`q settings knowledge.defaultIncludePatterns '["**/*.rs", "**/*.md"]'` # 默认包含模式
`q settings knowledge.defaultExcludePatterns '["target/**", "node_modules/**"]'` # 默认排除模式

## 工作原理

#### 索引过程

当您向知识库添加内容时：

1. **模式过滤**：根据包含/排除模式过滤文件（如果指定）
2. **文件发现**：系统递归扫描目录以查找支持的文件类型
3. **内容提取**：从每个支持的文件中提取文本内容
4. **分块**：大文件被分割成更小的可搜索块
5. **后台处理**：索引在后台异步进行
6. **语义嵌入**：内容被处理以获得语义搜索能力

#### 搜索能力

知识库使用语义搜索，这意味着：

- 您可以使用自然语言查询进行搜索
- 结果按相关性排序，而不仅仅是关键词匹配
- 即使确切的词不匹配，也能找到相关概念

#### 持久化

- 持久化上下文：在聊天会话和 CLI 重启之间保持
- 上下文持久化根据使用模式自动确定
- 包含/排除模式与每个上下文一起存储，并在更新期间重用

#### 最佳实践

组织您的知识库

- 添加上下文时使用描述性名称："api-documentation" 而不是 "docs"
- 在添加文件之前将相关文件分组到目录中
- 使用包含/排除模式专注于相关文件
- 定期审查和更新过时的上下文

#### 有效搜索

- 使用自然语言查询："如何使用知识工具处理身份验证错误"
- 对您要查找的内容要具体："数据库连接配置"
- 如果初始搜索没有返回预期结果，请尝试不同的措辞
- 提示 Q 使用工具，如"使用您的知识库查找数据库连接配置"或"使用您的知识工具，您能找到如何更换笔记本电脑吗"

#### 管理大型项目

- 尽可能添加项目目录而不是单个文件
- 使用包含/排除模式避免索引构建工件：`--exclude "target/**" --exclude "node_modules/**"`
- 使用 /knowledge status 监控大目录的索引进度
- 考虑将非常大的项目分解为逻辑子目录

#### 模式过滤最佳实践

- **要具体**：使用精确的模式避免过度包含
- **排除构建工件**：始终排除像 `target/**`、`node_modules/**`、`.git/**` 这样的目录
- **包含相关扩展名**：专注于您实际需要搜索的文件类型
- **测试模式**：在大型索引操作之前验证模式匹配预期文件

## 限制

#### 文件类型支持

- 索引期间忽略二进制文件
- 非常大的文件可能被分块，可能会分割相关内容
- 某些专门的文件格式可能无法最佳地提取内容

#### 性能考虑

- 大目录可能需要大量时间来索引
- 后台操作受并发处理限制的限制
- 搜索性能可能因知识库大小和嵌入引擎而异
- 模式过滤在文件遍历期间发生，提高大目录的性能

#### 存储和持久化

- 没有明确的存储大小限制，但适用实际限制
- 没有自动清理旧的或未使用的上下文
- 清除操作是不可逆的，没有备份功能

## 故障排除

#### 文件未被索引

如果您的文件没有出现在搜索结果中：

1. **检查模式**：确保您的包含模式匹配您想要的文件
2. **验证排除模式**：确保排除模式没有过滤掉所需的文件
3. **检查文件类型**：确保您的文件具有支持的扩展名
4. **监控状态**：使用 /knowledge status 检查索引是否仍在进行中
5. **验证路径**：确保您添加的路径实际存在且可访问
6. **检查错误**：在 CLI 输出中查找错误消息

#### 搜索未找到预期结果

如果搜索没有返回预期结果：

1. **等待索引**：使用 /knowledge status 确保索引完成
2. **尝试不同查询**：使用各种措辞和关键词
3. **验证内容**：使用 /knowledge show 确认您的内容已添加
4. **检查文件类型**：不支持的文件类型不会有可搜索的内容

#### 性能问题

如果操作缓慢：

1. **检查队列状态**：使用 /knowledge status 查看操作队列
2. **如需要则取消**：使用 /knowledge cancel 停止有问题的操作
3. **添加较小的块**：考虑添加子目录而不是整个大型项目
4. **使用更好的模式**：使用排除模式排除不必要的文件
5. **调整设置**：考虑降低 maxFiles 或 chunkSize 以获得更好的性能

#### 模式问题

如果模式没有按预期工作：

1. **测试模式**：首先使用简单模式，然后增加复杂性
2. **检查语法**：确保 glob 模式使用正确的语法（`**` 用于递归，`*` 用于单级）
3. **验证路径**：确保模式匹配项目中的实际文件路径
4. **使用绝对模式**：考虑在模式中使用完整路径以获得精确性
