# vault-builder

一个 Claude Code Skill，帮你 10 分钟搭好「个人经验库」工作流。

**核心理念：经验库 ≠ 百科库**
> 笔记里必须有"我"——我怎么理解、我踩过什么坑、我的用法。百科可以搜到，你的经验搜不到。

## 效果

跑完这个 skill，你会得到：

- ✅ 个性化 Obsidian 目录结构（按你的角色和领域定制）
- ✅ 5 个笔记模板（经验 / 方法论 / 知识 / 迭代 / 规划）
- ✅ Claude 认识你的角色，每次新对话自动加载
- ✅ Stop hook：每次 Claude 回复后自动 git commit，笔记不丢
- ✅（可选）GitHub 私有 repo 备份 + 多设备同步
- ✅（可选）手机飞书发消息 → 电脑 Claude 直接存知识库

## 安装

**前置：需要 Claude Code CLI**
```bash
npm install -g @anthropic-ai/claude-code
claude  # 登录
```

**安装 skill：**
```bash
# 克隆这个 repo
git clone https://github.com/<你的用户名>/vault-builder.git

# 放到 Claude Code skill 目录
mkdir -p ~/.claude/skills/vault-builder
cp -r vault-builder/* ~/.claude/skills/vault-builder/
```

**触发：**

在任意目录打开 Claude Code，说：
> "帮我搭知识库"

Claude 会问你 4 个问题，然后自动建好。

## 文件说明

```
vault-builder/
├── SKILL.md                        # 主 skill 定义（Claude Code 读取）
├── companion-vault-knowledge/
│   └── SKILL.md                    # 配套的 vault-knowledge skill（自动安装）
└── templates/
    ├── Experience.md               # 学习经验（必须有"我"）
    ├── Methodology.md              # 方法论（≥2 个案例才升级）
    ├── Knowledge.md                # 概念（少量用）
    ├── Iteration.md                # 项目迭代
    ├── Roadmap.md                  # 产品规划
    ├── Opportunity.md               # 外部机会追踪
    ├── Project-Showcase.md           # 项目展示卡片
    └── Monthly-Review.md           # 月度复盘
```

## 手机端接入（可选）

用 [cc-connect](https://github.com/chenhg5/cc-connect) 接飞书 bot，实现手机发消息 → Claude 存知识库：

```bash
npm install -g cc-connect@latest
cc-connect feishu setup --project vault
```

`~/.cc-connect/config.toml` 中**必须加这三行**，否则 Claude 每个工具调用都往飞书发一条消息：

```toml
[projects.platforms.options]
app_id = "..."
app_secret = "..."
thinking_messages = false
tool_messages = false
progress_style = "compact"
```

## 注意事项

- 这套工作流只支持 **Claude Code**（不支持 Cursor / Codex）
- vault 建议放在本地，不要放 iCloud Drive 内层（git 和 iCloud 同时同步会冲突）
- Opportunity / Project-Showcase 模板是可选的，不需要可以不装
