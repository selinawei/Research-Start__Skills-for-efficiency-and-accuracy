# Claude Research Skills 🧪

一套面向 **AI / LLM 科研工作流** 的 [Claude Code](https://claude.com/claude-code) Skills。把代码进度管理、算法审查、任务执行这三件"每天都要做但最容易偷懒"的事，交给 Claude 按固定流程帮你跑完。

每个 skill 都强制使用 **5 个并行 agent**（算法流程 / 边界异常 / 数值正确性 / 性能 / 对照设计）来完成工作，保证"不跳步、不遗漏"。

---

## 📦 Skills 一览

| Skill | 精确触发词 | 作用 |
|---|---|---|
| [`research-progress-tracker`](./research-progress-tracker/SKILL.md) | `科研启动` | 启动科研工作流：读取 `PROGRESS.md` / `PIPELINE.md` / `EXPERIMENTS.csv`，检查 conda 环境，汇报当前状态；任务结束后自动 git commit、更新进度/pipeline/实验记录 |
| [`research-algorithm-checker`](./research-algorithm-checker/SKILL.md) | `科研启动-检查` | 逐段阅读、逐段质疑的算法审查：对照论文/伪代码检查实现正确性、边界条件、数值计算、性能瓶颈；发现的 issue 按 todo/doing/discard 分类写入 `ISSUES.md`，被 discard 的思路以后不再重复提 |
| [`research-task-executor`](./research-task-executor/SKILL.md) | `科研启动-执行` | 高质量执行一个具体的科研编码任务：从原始需求出发，先调研后动手，避免过度发散 |

> ⚠️ **触发词是精确匹配的**。例如 `research-progress-tracker` 只在用户输入**完全就是** `科研启动` 这四个字时才会激活——"开始写代码"、"继续科研"这类近义说法都不会触发。这是故意的，防止误启动整个重流程。

---

## 🔧 安装

Claude Code 会自动加载 `~/.claude/skills/` 下的所有 skill，所以最简单的装法就是把这个 repo clone 过去：

```bash
# 方式一：整个 repo 作为 skills 目录（如果你还没有自己的 skills）
git clone https://github.com/<your-username>/claude-research-skills.git ~/.claude/skills

# 方式二：clone 到别处，再把三个 skill 目录 symlink 过去（推荐，不影响已有 skills）
git clone https://github.com/<your-username>/claude-research-skills.git ~/claude-research-skills
ln -s ~/claude-research-skills/research-progress-tracker   ~/.claude/skills/research-progress-tracker
ln -s ~/claude-research-skills/research-algorithm-checker  ~/.claude/skills/research-algorithm-checker
ln -s ~/claude-research-skills/research-task-executor      ~/.claude/skills/research-task-executor
```

装完后在 Claude Code 里输入 `/` 或直接输入触发词，就能看到这些 skill 被激活。

---

## 🚀 使用示例

```
你：科研启动
Claude：[检查 conda 环境 → 读 PROGRESS.md / PIPELINE.md / EXPERIMENTS.csv → git status → 一致性检查 → 汇报]
        当前状态：... 本次要做什么？

你：科研启动-检查
Claude：要检查哪些文件？有没有对照的论文/伪代码？重点关注算法正确性还是性能？
你：检查 generate.py 里的 confidence-based unmask，对照 Fast-dLLM 论文 Sec.3
Claude：[5 agent 并行逐段审查 → 汇总问题 → 逐条询问 todo/doing/discard → 更新 ISSUES.md]

你：科研启动-执行
Claude：[确认需求 → 调研代码库 → 写实现 → 自检 → git commit]
```

---

## 📁 这些 skill 会在项目根目录维护的文件

| 文件 | 内容 | 由谁维护 |
|---|---|---|
| `PROGRESS.md` | 按时间倒序的代码改动记录 | progress-tracker / algorithm-checker |
| `PIPELINE.md` | 实验执行顺序（先跑哪个脚本、再跑哪个） | progress-tracker |
| `EXPERIMENTS.csv` | 所有实验（包括失败的）的结构化记录 | progress-tracker |
| `ISSUES.md` | 算法审查发现的问题，分 todo/doing/done/discarded | algorithm-checker |

> 失败实验同样要记录——这是 skill 强制执行的科研好习惯。

---

## ⚙️ 个人配置项

这些 skill 里有一些**我自己的路径和身份信息**是硬编码的，clone 下来需要改成你自己的：

- **Git commit 身份**：`selinawei` / `selinawei2003@outlook.com`
- **Git push SSH key**：`/sensei-fs-3/users/hyou/Lixuan/.ssh/id_ed25519_selinawei`
- **Conda 环境推荐表**（按项目目录 → 环境名）
- **Conda 基础路径**：`/sensei-fs-3/users/hyou/Lixuan/miniconda3`

在各自的 `SKILL.md` 里搜这些字符串替换即可。

---

## 📄 License

MIT — 随便改随便用。
