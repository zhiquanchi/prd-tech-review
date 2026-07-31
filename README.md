# prd-tech-review

Agent Skill（Claude Code / Codex / Grok）：对**已完成**的 PRD 做**技术审查 → 交互问答 → PRD 重生成与验证图生成**闭环。

核心分工：**PM 做决策，AI 画图。** 图表不要求 PM 提供——AI 从 PRD 推导模型，不确定处变成选择题，PM 确认后按需生成验证图。

**不是**从 0 写 PRD / 出原型的工具；那类需求用 `prd-writer`（若已安装）。

## 做什么

1. **技术审查**：D1–D10 始终审 PRD；D11 有原型/设计稿才审；D12–D17 有图审图，无图则文字推导（**缺图不定级**）
2. **交互问答**（完整模式）：点选 UI 收集决策；默认 ≤15 题；同型缺口合并
3. **重生成 + 验证图**：融合原始 PRD、审查结论与 PM 回答；默认优先 1–2 张关键验证图

### 运行模式

| 模式 | 触发 | 行为 |
|------|------|------|
| **完整**（默认） | 审查 / 能否开发 / 补充 / 重生成 | 阶段 1 → 同一轮点选 → 重生成 |
| **轻量** | 「只审不改」「只要报告」「先别问我」 | **仅阶段 1 报告**，不自动问答 |

## 点选 UI 多端适配

| 环境 | 工具 | 单次题量 |
|------|------|----------|
| **Codex** | `request_user_input` | 1–3（prefer 1 仅极重题） |
| **Claude Code** | `AskUserQuestion` | 1–4（尽量打满） |
| **Grok** | `ask_user_question` | 1–4（尽量打满） |
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

仓库为**单源**。多 Agent 目录请 clone 同一仓库或使用符号链接，避免副本漂移。

### Claude Code

```bash
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.claude/skills/prd-tech-review
# 或项目级
git clone git@github.com:zhiquanchi/prd-tech-review.git .claude/skills/prd-tech-review
```

### Codex

```bash
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.codex/skills/prd-tech-review
# 或项目级 .codex/skills/prd-tech-review
```

### Grok

```bash
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.grok/skills/prd-tech-review
# 或项目级
git clone git@github.com:zhiquanchi/prd-tech-review.git .grok/skills/prd-tech-review
```

新开会话后，在「审阅 PRD / 检查能否开发」等描述下会自动匹配；也可点名 `prd-tech-review`。

可在 `AGENTS.md` 加一句提高命中率：

```markdown
PRD 技术审查 / 需求缺口澄清 / 审查后重生成 PRD：使用 skill prd-tech-review。
从 0 写 PRD / 原型：使用 skill prd-writer（若有）。
```

### 可选依赖

| Skill | 作用 | 未安装时 |
|-------|------|----------|
| `prd-writer` | 从 0 写 PRD/原型 | 用户要「写需求」时勿硬套本 skill |
| `html-report` | 报告 / 重生成 HTML | 直接写 HTML 文件或 Markdown + Mermaid |
| `doc-writing-guide` | 文档写作参考 | 忽略即可 |

### 目录结构

```
prd-tech-review/
├── SKILL.md                 # 主指令（强制行为 + 加载预算 + 模式）
├── README.md
└── references/
    ├── review-checklist.md
    ├── qa-generation-guide.md
    ├── prd-regeneration-guide.md
    ├── functional-module-framework.md
    ├── ui-prototype-framework.md
    ├── eval-notes.md
    ├── fixtures/            # 回归用迷你 PRD
    └── …各图审查 framework
```

## 输入要求

| 输入 | 必需？ | 作用 |
|------|--------|------|
| PRD 文档 | **必需** | 做什么 |
| UI 原型 / 设计稿 | 可选 | 怎么交互（D11，形态分级） |
| UX / DFD / 状态机 / 流程 / 时序 / ER | 可选 | 有则审图；无则推导后生成验证图 |

## 题量与定级（摘要）

- 默认只问阻断 + 重要；建议级不进问卷  
- 硬上限 **≤ 15 题**（深挖模式 ≤ 25）  
- 同型缺口合并；超出则行业默认 + ❓，禁止静默丢弃  
- **纯结构缺失**（无 OOS/验收章节）默认 **重要**，不单独因此整单 🔴  
- 点选 options **禁止**手写 Other  

## 触发示例

- 审阅 / 审查 PRD  
- 检查 PRD 是否可以开始开发  
- 找出需求缺口或风险  
- 只要报告、先别提问（轻量）  
- 审查后引导产品补充细节  
- 根据审查结果重新生成 PRD  

## 轻量回归

见 `references/eval-notes.md` 与 `references/fixtures/`。

## License

MIT（可按需修改）
