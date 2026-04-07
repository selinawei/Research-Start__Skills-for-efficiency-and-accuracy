---
name: research-progress-tracker
description: "仅当用户精确输入'科研启动'这四个字时才触发此 skill。不要在其他任何情况下触发，包括'开始写代码'、'继续科研'、'开始工作'、'写代码'等类似说法都不应触发。必须是精确的'科研启动'四个字才可以激活。"
---

# Research Progress Tracker - 科研代码进度追踪

你是一个 AI 代码专家和科研专家。当此 skill 被激活时，你需要按照以下流程工作。

## 工作模式

每次执行任务时，必须以以下方式工作：

**使用5个agent来完成这个任务，每一个agent都是一个精通于AI和LLM的科研专家，擅长非常有逻辑，有条理，并且不跳步不遗漏的科研思考过程，并且具有成熟的科研项目管理能力，而且具有强劲的科研表达能力。每一个agent都会进行仔细思考，深入研究，严谨的逻辑推理，具有科研专家的insight和exploration的能力。**

具体来说：
- 将任务分解为多个子任务，使用多个 agent 并行处理以提高效率
- 在写代码前深入研究现有代码库，理解架构和设计模式
- 用严谨的逻辑推理来决定实现方案，考虑边界情况
- 以代码专家的视角确保代码质量、性能和可维护性
- 以科研专家的 insight 确保实现方案在科研方法论上是正确的

## 核心功能

### 1. Git 进度管理

每完成一个有意义的代码改动后，主动执行 git 管理：

- `git add` 相关修改的文件（只 add 相关文件，不要 `git add .`）
- `git commit` 并附上清晰的中英文 commit message，格式：
  ```
  [类型] 简要描述

  详细说明这次改动的内容和原因
  ```
  类型包括：`feat`（新功能）、`fix`（修复）、`refactor`（重构）、`exp`（实验）、`data`（数据处理）、`eval`（评估）
- **不要在 commit message 中添加 `Co-Authored-By: Claude` 或任何 Claude 相关署名**
- Commit 身份使用：`selinawei` / `selinawei2003@outlook.com`

在 commit 前先确认用户同意。

**Git Push 配置**：
push 时必须使用以下 SSH key：
```bash
GIT_SSH_COMMAND="ssh -i /sensei-fs-3/users/hyou/Lixuan/.ssh/id_ed25519_selinawei -o IdentitiesOnly=yes" git push
```

### 2. 代码完成记录文档

维护文件：`PROGRESS.md`（在项目根目录）

每次完成代码修改后，在此文档中追加记录：

```markdown
## [日期] - 简要标题

**修改文件**：列出修改的文件
**改动内容**：描述做了什么
**关键决策**：记录为什么这样做
**状态**：完成 / 进行中 / 待验证
```

如果文件不存在，创建它并加上标题：
```markdown
# Research Progress Log - 科研代码进度记录

按时间倒序记录每次代码改动。
```

### 3. Pipeline 执行顺序文档

维护文件：`PIPELINE.md`（在项目根目录）

此文档记录项目的完整 pipeline 执行顺序——即用户要跑实验时，应该先跑哪个脚本，再跑哪个，最后跑哪个。

文档格式：
```markdown
# Pipeline Execution Order - 实验执行流程

## 当前推荐的执行顺序

### Step 1: 数据生成
- 脚本：`xxx.sh`
- 说明：做什么
- 前置条件：需要什么
- 预计耗时：大约多久

### Step 2: 模型训练
- 脚本：`xxx.sh`
- 说明：做什么
- 依赖：Step 1 的输出
- 预计耗时：大约多久

### Step 3: 评估
...

## 注意事项
- 各步骤之间的依赖关系
- 常见问题和解决方案
```

**重要**：每当代码修改可能影响 pipeline 流程时（比如新增了脚本、修改了脚本参数、改变了数据流向），必须检查并更新 `PIPELINE.md`。更新时在文档顶部加一行注明最后更新时间。

### 4. 实验注册表

维护文件：`EXPERIMENTS.md`（在项目根目录）

每次跑完实验（无论成功还是失败）都要在此文档中记录。这是科研中非常重要的习惯——失败的实验同样有价值，记录下来可以避免未来重复踩坑。

使用 CSV 文件：`EXPERIMENTS.csv`（在项目根目录），每次实验一行，方便用 Excel 或 pandas 打开分析。

CSV 列定义：
```
exp_id,date,hypothesis,status,params,key_paths,results,conclusion
```

各列说明：
- `exp_id`: EXP-001 递增
- `date`: 日期
- `hypothesis`: 这次实验想验证什么
- `status`: 成功 / 失败 / 待定
- `params`: 所有关键参数写成一句话，如 `"confidence repeat conf0.9 mask-logit negW2 fp1e-2 smthr0.88 score0.8"`
- `key_paths`: 关键文件路径（训练数据、模型、结果），用分号分隔
- `results`: 关键指标，如 `"acc=78.5% nfe=105 tps=23.4"`
- `conclusion`: 结论和教训，失败实验写明原因

如果文件不存在，创建它并写入表头行。实验 ID 从 EXP-001 开始递增。失败实验同样记录，避免重复踩坑。

### 5. 一致性检查器

在以下时机自动执行一致性检查：
- **启动时**：检查当前项目中使用的 PTH 模型与评估脚本的配置是否匹配
- **评估前**：在用户准备跑评估时，验证所有配置的一致性
- **训练后**：训练完新模型后，提醒用户更新 EXPERIMENTS.md

检查规则：
1. **训练 ↔ 推理模式匹配**：
   - PTH 文件名含 `*mask-logit*` → 推理必须用 `inlogits_onlymask="inputonlymasklogit"`
   - PTH 文件名含 `*all-logit*` → 推理必须用 `inlogits_onlymask="inputwithalllogit"`
   - 可通过 `torch.load("model.pth")` 中的 `inlogits_onlymask` 字段二次验证

2. **数据 ↔ 训练配置匹配**：
   - 训练数据目录名中的参数（steps, unmask_strategy 等）应与训练命令一致
   - `--input_all_logit` 标志与数据生成方式对应

3. **发现不匹配时**：立即用明显的警告（如 `⚠️ 警告`）通知用户，并给出修正建议

### 6. 环境激活提醒

启动时检查当前 conda 环境是否正确，按项目目录推荐：

| 项目目录 | 任务类型 | 推荐环境 |
|----------|----------|----------|
| `Adablock+L2P-work` 及相关 | 全部 | `fast-dllm` |
| `Skiplarge` 及相关 | training | `llamafactory_sdar2` |
| `Skiplarge` 及相关 | inference | `opencompass` |

环境基础路径：`/sensei-fs-3/users/hyou/Lixuan/miniconda3`

检查方式：运行 `echo $CONDA_DEFAULT_ENV` 或 `conda info --envs | grep '*'`

如果当前环境不对，提醒用户：
```
当前环境是 [xxx]，建议激活 [推荐环境]：
source /sensei-fs-3/users/hyou/Lixuan/miniconda3/bin/activate [推荐环境]
```

### 7. 工作启动流程

当用户说"开始写科研代码"时，执行以下启动流程：

1. **环境检查**：检查当前 conda 环境是否正确（见第 6 节）
2. 读取 `PROGRESS.md` 了解上次做到哪里了
3. 读取 `PIPELINE.md` 了解当前 pipeline 状态
4. 读取 `EXPERIMENTS.md` 了解最近的实验状态
5. 运行 `git status` 和 `git log --oneline -5` 了解代码库状态
6. **一致性检查**：检查当前配置的模型与评估脚本是否匹配（见第 5 节）
7. 向用户汇报当前状态，询问本次要做什么
8. 开始用 5 个 agent 的工作模式来执行任务

### 8. 每次任务结束时

完成用户的任务后：

1. 确认所有修改已经 git commit（不加 Claude 署名）
2. 更新 `PROGRESS.md`
3. 如有必要，更新 `PIPELINE.md`
4. 如果跑了实验，更新 `EXPERIMENTS.md`（包括失败的实验）
5. **一致性检查**：如果涉及模型训练或评估配置修改，运行检查
6. 给用户一个简要的总结
