# 贡献指南

感谢你对 AI Skills 的关注！以下是贡献流程。

## 贡献一个新 Skill

### 1. Fork 本仓库

### 2. 创建 Skill 目录

```
skills/<your-skill-name>/
├── SKILL.md
└── scripts/        # 可选
```

### 3. 编写 SKILL.md

按照 [SKILL-FORMAT.md](SKILL-FORMAT.md) 的格式编写。

**必须包含**：
- YAML Frontmatter（name + description）
- 功能说明
- 安装步骤（带验证）
- 代码示例（可直接复制）
- 问题排查

**不能包含**：
- 个人路径、设备名、密码、Token

### 4. 测试

在至少一个 AI 工具中测试你的 Skill 能跑通：
- Cursor / Claude / MiMo / Trae 等
- 记录测试环境（操作系统、AI 工具版本）

### 5. 提交 PR

PR 描述中写清楚：
- 这个 Skill 做什么
- 测试了哪些环境
- 依赖哪些外部服务（如有）

## 改进现有 Skill

直接提 PR，说明改了什么、为什么改。

## 问题反馈

提 Issue，描述清楚：
- 哪个 Skill
- 什么问题
- 你的环境（操作系统、AI 工具）
- 错误信息

## 格式要求

详见 [SKILL-FORMAT.md](SKILL-FORMAT.md)。
