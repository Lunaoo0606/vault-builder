---
name: vault-builder
description: 一键搭建"个人经验库"vault 工作流。用户说"帮我搭知识库"、"我也想搞 Obsidian + Claude"、"build my vault"、"setup vault" 时使用。会问用户一组问题（vault 路径、角色、领域），然后创建目录结构 + 模板 + 配套 skill + Claude Code 设置 + git 初始化。**核心定位：经验库不是百科库**——所有产物都强制"必须有'我'"准则。
---

# vault-builder

把"经验库工作流"复制给任何想搭同款的朋友。**不是百科库**——所有沉淀都必须有"我"。

## 触发场景

- "帮我搭知识库" / "我也想搞 Obsidian + Claude" / "build my vault"
- "我朋友给我看了她的 vault，我也想要"
- "怎么用 Claude Code 管 Obsidian"

## 设计哲学（必须先讲给用户）

**经验库 ≠ 百科库**：

> 别人的方案我们抄不来，因为知识库的价值不在"有什么"，而在"**你**对它怎么理解"。
> 这个 skill 帮你搭骨架，但骨架里的肉必须你自己长——AI 不替你编。

3 条准则会写进生成的 CLAUDE.md：
1. **每条笔记必须有"我"**
2. **不编**——用户没问过的不主动补
3. **经验 = 知识 + 我的理解**

## 执行流程

### Step 1：搜集信息（用 AskUserQuestion）

问 4 个问题：

1. **vault 放哪？** 默认 `~/Documents/Obsidian Vault`（建议跟 iCloud / OneDrive 隔开）
2. **你的角色是？**（产品 / 研发 / 设计 / 数据 / 学生 / ...）一句话描述
3. **你想沉淀的领域有哪些？**（举 3-5 个，如 AI / 业务 / 学习 / 生活）
4. **要不要 GitHub 同步？**（私有 repo 备份 + 多设备）

### Step 2：建目录 + 模板

```
<VAULT>/
├── 00-Inbox/
├── 10-Knowledge/
│   ├── 经验/                 # 我学某个东西的"我的视角"
│   │   └── <用户填的领域>/
│   └── 方法论/                # 我抽象出的可复用框架
├── 20-Products/              # 用户的产品 / 项目
├── 30-Daily/
├── 50-Raw/
├── 99-Templates/
│   ├── Experience.md          # 学习经验模板（必须有"我"）
│   ├── Methodology.md         # 方法论模板（≥2 个案例才升级）
│   ├── Knowledge.md           # 概念模板（少量用）
│   ├── Iteration.md           # 项目迭代模板
│   └── Roadmap.md             # 产品规划模板
├── CLAUDE.md                  # vault 内约定 + 沉淀准则
└── README.md                  # 入口说明
```

模板源文件在本 skill 目录的 `templates/` 下，复制过去即可。

### Step 3：装 Claude Code 配套

#### 3a. 加用户级 CLAUDE.md 角色记忆

如果用户的 `~/.claude/CLAUDE.md` 不存在或没有"沉淀准则"段，追加：

```md
## 用户角色
- <用户在 Step1 填的角色>
- 个人知识库：<vault 路径>

## 沉淀准则（强要求）
1. 每条笔记必须有"我"——我的认知层级 / 思考 / 应用 / 抽象方法论
2. 不编——用户没问过 / 没学过 / 没经历过的内容不主动补全
3. 百科只留用户真问过的部分，不为完整性扩写
4. 经验 = 知识 + 我的理解：两者都要，基于真实问答
5. 抗拒"完整知识地图"——允许碎片、允许只懂一半

## 改动报告
- 每次对 vault 做创建 / 修改 / 删除，回复时显式列"本次改动清单"
```

#### 3b. 装 vault-knowledge skill

把本 skill 目录里的 `companion-vault-knowledge/SKILL.md` 拷贝到 `~/.claude/skills/vault-knowledge/SKILL.md`，并把里面的 `<VAULT>` 占位替换成用户的实际路径。

#### 3c. 配 Stop hook 自动 git commit

在 `~/.claude/settings.json` 的 hooks.Stop 加入：

```json
{
  "type": "command",
  "command": "cd '<VAULT>' && git add -A && (git diff --cached --quiet || git commit -m \"auto: $(date '+%Y-%m-%d %H:%M')\" --quiet) 2>/dev/null || true"
}
```

注意：合并到现有 hooks.Stop，不要覆盖。

### Step 4：git 初始化

```bash
cd <VAULT>
git init -b main
cat > .gitignore <<EOF
.obsidian/workspace*
.obsidian/workspace.json
.obsidian/workspaces.json
.trash/
copilot/
EOF
git add -A
git commit -m "initial vault snapshot"
```

### Step 5：可选 GitHub 同步

如果用户在 Step 1 选了 yes：

```bash
# 让用户在 github.com/new 创建 private repo（不勾 README/gitignore）
# 然后：
git remote add origin <repo-url>
git push -u origin main
```

如果用 HTTPS，提示用户准备 PAT（Personal Access Token），范围只勾 repo 即可。

### Step 6：可选 手机端接入（cc-connect + 飞书 bot）

如果用户想从手机飞书发消息触发 Claude，可接入 cc-connect：

```bash
npm install -g cc-connect@latest
cc-connect feishu setup --project vault   # 扫码创建飞书 bot
```

配置文件在 `~/.cc-connect/config.toml`，**必须加以下三行**减少消息噪音：

```toml
[projects.platforms.options]
app_id = "..."
app_secret = "..."
thinking_messages = false   # 不推送思考过程
tool_messages = false       # 不推送工具调用步骤
progress_style = "compact"  # 只用一条消息持续更新，不是每步发新消息
```

不加这三行，Claude 每跑一个工具都会往飞书发一条消息，非常嘈杂。

### Step 7：可选 Obsidian 插件

提醒用户手动装：
- **Obsidian Git**：GUI 看历史 / 回退（左侧 git 图标）
- **Templater**：模板自动填充日期 / 标题
- **Dataview**：跑表格查询

## 验证清单

跑完后检查：

- [ ] `<VAULT>/CLAUDE.md` 包含 4 个目录约定 + 沉淀准则
- [ ] `<VAULT>/99-Templates/` 有 5 个模板
- [ ] `~/.claude/CLAUDE.md` 包含角色 + 沉淀准则
- [ ] `~/.claude/skills/vault-knowledge/SKILL.md` 存在，路径已替换
- [ ] `~/.claude/settings.json` 有 Stop hook
- [ ] `<VAULT>` 内 `git log` 看到 initial commit
- [ ] （可选）GitHub repo 看到推上去的内容
- [ ] （可选）cc-connect 配置含 `thinking_messages = false` / `tool_messages = false` / `progress_style = "compact"`

## 反模式

- ❌ 不问角色 / 领域就开始建——朋友的 vault 不是你的，必须个性化
- ❌ 把模板写得太多太细——朋友看到 10 个模板会犯难，5 个核心就够
- ❌ 替朋友写第一篇笔记 demo——这违反"不编"准则
- ❌ 装 8 个 Obsidian 插件——只推荐 3 个核心，其他自己探索

## 致谢

源头启发：小红书爆款"我如何用 obsidian+AI 做知识管理"（ami.moment 1.1 万赞）的"**不要再做知识囤积者**"理念。
