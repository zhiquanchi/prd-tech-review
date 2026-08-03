# prd-tech-review

Agent Skill 仓库（Claude Code / Codex / Grok / Qwen Code 等）：对**已完成**的 PRD 做**技术审查 → 交互问答 → PRD 重生成与验证图生成**闭环。

本仓库是**单源仓库，含三个可独立安装的 skill**：

| 目录 | name | 职责 |
|------|------|------|
| `prd-review/` | `prd-review` | 技术审查（D1–D17）+ 交互问答（点选 UI）+ 轻量模式；产出审查报告与 PM 决策记录 |
| `prd-regeneration/` | `prd-regeneration` | 融合原始 PRD + 审查报告 + PM 回答记录 → 重生成可开发 PRD + 按需验证图 |
| `connect/` | `connect` | 安装器：把 prd-review / prd-regeneration 以软链安装到本机各 Agent skills 目录（list / uninstall / --path） |

**协作闭环：** `prd-review` 完成审查与决策收集后，将交接物（原始 PRD + 审查报告 + PM 回答记录）交给 `prd-regeneration` 重生成。也可单独使用：只审不改（轻量模式）、或拿已有审查结果直接重生成。**`connect` 只负责安装，不参与审查流程。**

核心分工：**PM 做决策，AI 画图。** 图表不要求 PM 提供——AI 从 PRD 推导模型，不确定处变成选择题，PM 确认后按需生成验证图。

**不是**从 0 写 PRD / 出原型的工具；那类需求用 `prd-writer`（若已安装）。

## prd-review：技术审查 + 交互问答

1. **技术审查**：D1–D10 始终审 PRD；D11 有原型/设计稿才审；D12–D17 有图审图，无图则文字推导（**缺图不定级**）；模块完整性 ⑦⑧⑪ 两级定级
2. **交互问答**（完整模式）：点选 UI 收集决策；默认 ≤15 题；同型缺口合并
3. **交接**：输出审查报告（固定结构）+ PM 回答记录，声明调用 prd-regeneration

### 运行模式

| 模式 | 触发 | 行为 |
|------|------|------|
| **完整**（默认） | 审查 / 能否开发 / 补充 / 重生成 | 阶段 1 → 同一轮点选 → 交接重生成 |
| **轻量** | 「只审不改」「只要报告」「先别问我」 | **仅阶段 1 报告**，不自动问答 |

## prd-regeneration：重生成 + 验证图

**输入契约：** 原始 PRD（必需）+ 审查报告（必需）+ PM 回答记录（完整模式必需；轻量模式缺省用 ❓ 兜底）。

**工作流：** 融合 → 🆕/📝/❓ 修订标记 → 验收推导 → 按需 1–2 张验证图 → framework 自检 → `V{主版本号}.tech-review.{YYYYMMDD}` → 输出（完整 PRD + 图 + 产物清单 + 问题对照表）。

## 点选 UI 多端适配（prd-review 阶段 2）

| 环境 | 工具 | 单次题量 |
|------|------|----------|
| **Codex** | `request_user_input` | 1–3（prefer 1 仅极重题） |
| **Claude Code** | `AskUserQuestion` | 1–4（尽量打满） |
| **Grok / Qwen Code** | `ask_user_question` | 1–4（尽量打满） |
| 工具失败/真无 | Markdown 降级 | ≤8 |

**探测顺序：** `request_user_input` → `AskUserQuestion` / `ask_user_question` → 其它同构工具 → Markdown。

**默认行为（完整模式）：直接触发点选 UI。** 审查报告交付后立刻 tool call，不征求「是否提问」、不先输出 Markdown A/B/C。

### Codex 注意

- `header` ≤ 12 字符；推荐项 label 后缀 ` (Recommended)`
- **不要**手写 Other；阻断题不要设 `autoResolutionMs`
- 默认只在 **Plan 模式**暴露 `request_user_input`。Default 模式在 `~/.codex/config.toml`：

```toml
[features]
default_mode_request_user_input = true
```

然后**新开**会话。可用 `codex features list | rg default_mode_request` 确认。

## 安装

仓库为**单源**。多 Agent 目录请 clone 同一仓库后符号链接所需 skill，避免副本漂移。三个 skill 可只装其一：

```bash
# 1. clone 单源到任意位置
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.skills-src/prd-tech-review

# 2. 链接需要的 skill（可只链一个）；以下为 Claude Code 示例
ln -s ~/.skills-src/prd-tech-review/prd-review ~/.claude/skills/prd-review
ln -s ~/.skills-src/prd-tech-review/prd-regeneration ~/.claude/skills/prd-regeneration
ln -s ~/.skills-src/prd-tech-review/connect ~/.claude/skills/connect
# Codex：目标目录换 ~/.codex/skills/；Qwen Code：~/.qwen/skills/；Grok：~/.grok/skills/
# 项目级安装：目标目录换 .claude/skills/ 等
```

新开会话后，在「审阅 PRD / 检查能否开发 / 重生成 PRD」等描述下会自动匹配；也可点名 `prd-review` / `prd-regeneration` / `connect`。

**connect 自举：** 把 `connect/` 手动软链到任一 agent 后，其余 skill 的安装/卸载/预检可交给 `connect`（`connect` → 全量安装；`connect list` → 预检；`connect uninstall` → 卸载）。

可在 `AGENTS.md` 加一句提高命中率：

```markdown
PRD 技术审查 / 需求缺口澄清：使用 skill prd-review。
审查后重生成 PRD / 补生成验证图：使用 skill prd-regeneration。
安装/卸载本仓库 skill 到本机 agent：使用 skill connect。
从 0 写 PRD / 原型：使用 skill prd-writer（若有）。
```

### 双副本同步注意

`references/functional-module-framework.md` 在 `prd-review/` 与 `prd-regeneration/` 两个目录各有一份（各自独立安装需要）。**修改该文件时须两处同步**（可用 `cp` 覆盖或同时编辑）。

## 题量与定级（摘要）

- 默认只问阻断 + 重要；建议级不进问卷
- 硬上限 **≤ 15 题**（深挖模式 ≤ 25）
- 同型缺口合并；超出则行业默认 + ❓，禁止静默丢弃
- 模块完整性 ⑦⑧⑪ **两级定级**：正文不可推导 → 阻断；正文可推导但未成表 → 重要
- **纯结构缺失**（无 OOS/验收章节）默认 **重要**，不单独因此整单 🔴
- 点选 options **禁止**手写 Other

## 触发示例

### prd-review
- 审阅 / 审查 PRD
- 检查 PRD 是否可以开始开发
- 找出需求缺口或风险
- 只要报告、先别提问（轻量）
- 审查后引导产品补充细节

### prd-regeneration
- 根据审查结果重新生成 PRD
- 按审查结论修订 PRD
- 为审查过的 PRD 补生成验证图

### connect
- 把 prd-review / prd-regeneration 安装到本机 agent
- 列出支持安装的 agent 与状态（connect list）
- 从本机 agent 卸载（connect uninstall）
- 安装到指定项目 skills 目录（connect --path .claude/skills）

## 回归

prd-review 回归见 `prd-review/references/eval-notes.md` 与 `prd-review/references/fixtures/`。

## License

MIT（可按需修改）
