# Meta Skills / 元能力层

能力的能力——教 AI 如何设计和构建其他系统。

## 实用 Skill

| Skill | 说明 |
|-------|------|
| [ai-builder](ai-builder/SKILL.md) | AI 构建器，帮用户挑选组合 Skill |
| [contribute](contribute/SKILL.md) | 向 CreateYouAI 仓库贡献新 Skill |
| [mcp-adapter](mcp-adapter/SKILL.md) | MCP 工具适配器 |

## 架构指南（guides/）

当用户确定 Skill 组合后，用于细化设计。

| 指南 | 文件 | 用途 |
|------|------|------|
| 工具系统 | [tool-system-builder](ai-builder/guides/tool-system-builder.md) | 设计工具注册、执行、权限 |
| 多 Agent 协调 | [multi-agent-coordinator](ai-builder/guides/multi-agent-coordinator.md) | 设计多 Agent 协作 |
| 记忆系统 | [memory-system-design](ai-builder/guides/memory-system-design.md) | 设计分层记忆存储 |
| 查询引擎 | [query-engine-design](ai-builder/guides/query-engine-design.md) | 设计 LLM 调用循环 |
| Skill 系统 | [skill-system-design](ai-builder/guides/skill-system-design.md) | 设计 Skill 加载执行 |
| 状态管理 | [state-management-design](ai-builder/guides/state-management-design.md) | 设计应用状态存储 |
| 权限系统 | [permission-system-design](ai-builder/guides/permission-system-design.md) | 设计权限检查和规则 |
| 命令系统 | [command-system-design](ai-builder/guides/command-system-design.md) | 设计命令注册和解析 |
| 服务层 | [service-layer-design](ai-builder/guides/service-layer-design.md) | 设计外部服务调用 |
| IDE 集成 | [ide-integration-design](ai-builder/guides/ide-integration-design.md) | 设计 IDE 桥接扩展 |

## 架构指南说明

架构指南是**设计模式**，不是直接可用的 Skill：
- 用于用户确定 AI 构建方案后的细化设计阶段
- 需要根据用户项目适配
- TypeScript 版为主版本（生产验证），Python 版标记为社区贡献待验证
