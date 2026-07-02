# AI Skills

模块化的 AI Skill 仓库。每个 Skill 是一份独立的指令文档，AI 工具读取后即可获得对应能力。

## 什么是 Skill

Skill 是一份给 AI 看的说明书。用户对 AI 说"我要做 X"，AI 读取对应的 Skill，就知道怎么帮用户实现。

**举例**：用户说"我想给应用加语音识别"，AI 读取 `voice-recognition` Skill，就知道要安装什么、改哪些代码、怎么排查问题。

## 已有 Skills

| Skill | 状态 | 说明 |
|-------|------|------|
| [voice-recognition](skills/voice-recognition/) | ✅ 可用 | 本地语音识别（4 种录音方案 + funasr 本地模型） |

## 怎么用

1. 找到你需要的 Skill
2. 把 `SKILL.md` 的内容给你的 AI 工具（粘贴、引用、或放到项目目录）
3. AI 读取后按指引执行

## 怎么贡献

1. Fork 本仓库
2. 按照 [SKILL-FORMAT.md](SKILL-FORMAT.md) 的格式写一个新 Skill
3. 提交 PR

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## Skill 格式

详见 [SKILL-FORMAT.md](SKILL-FORMAT.md)。

核心结构：
```
skills/<skill-name>/
├── SKILL.md              # 必须：给 AI 的指令文档
└── scripts/              # 可选：可执行脚本
```

## License

MIT
