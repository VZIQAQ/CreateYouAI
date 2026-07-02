---
name: AI 构建器
description: >
  帮用户从 Skill 仓库中挑选合适的 Skill，组合成自己的 AI。
  当用户想要构建 AI、搭建智能体、创建自动化工作流、
  或描述"我想做一个能做 X 的 AI"时触发。
  关键词：构建 AI、搭建智能体、做 AI、自动化、工作流、组合 skill。
---

# AI 构建器

用户描述想要什么 AI，你帮他从仓库挑选 Skill 并组合。

## 执行流程

### Step 1：了解用户需求

问用户以下问题（如果用户已经说清楚了就跳过）：

1. **你想做什么？**（一句话描述目标）
2. **什么平台？**（桌面应用 / 网页 / Python 脚本 / 不确定）
3. **有什么技术偏好？**（有 Python 环境 / 纯 Node.js / 都行）

### Step 2：扫描可用 Skill

列出仓库中所有 Skill：

```bash
find skills/ -name "SKILL.md" -type f
```

读取每个 SKILL.md 的 frontmatter，获取 name、description、tags。

### Step 3：推荐 Skill 组合

根据用户需求，从仓库中匹配 Skill。

**匹配逻辑**：
- 读每个 SKILL.md 的 description，判断是否与用户需求相关
- 读 tags 中的 category 和 platform，过滤平台匹配的
- 优先推荐 dependencies 少的（轻量级）

**输出格式**：

```markdown
根据你的需求，推荐以下 Skill 组合：

| Skill | 用途 | 为什么选它 |
|-------|------|-----------|
| voice-recognition | 语音输入 | 你说需要语音交互 |
| file-manager | 文件操作 | 你说需要读写文件 |

**不需要的 Skill**：
- code-review（你说不需要代码审查）

**下一步**：
1. 先安装第一个 Skill 的依赖
2. 再配置第二个 Skill
3. 我帮你逐步集成
```

### Step 4：生成构建方案

帮用户生成一个构建计划：

```markdown
# 你的 AI 构建计划

## 目标
<用户描述>

## 需要的 Skill
1. <skill-1> — <用途>
2. <skill-2> — <用途>

## 安装顺序
1. 先装 <skill-1>（基础能力）
2. 再装 <skill-2>（扩展能力）

## 预估时间
- <skill-1>：<X 分钟>
- <skill-2>：<X 分钟>
- 总计：<X 分钟>
```

### Step 5：逐步执行

按计划逐个安装 Skill，每个 Skill 安装后验证。

## 示例对话

**用户**：我想做一个能语音控制电脑的 AI

**AI**：
1. 扫描仓库，找到 voice-recognition（语音输入）
2. 推荐：voice-recognition + file-manager（语音控制文件操作）
3. 生成构建计划
4. 逐步安装

## 注意事项

- 如果仓库里没有用户需要的 Skill，告诉用户可以自己写一个（参考 contribute-skill）
- 不要推荐太多 Skill（3-5 个为宜），太多会复杂
- 优先推荐经过验证的 Skill（标记为 ✅ Ready 的）
