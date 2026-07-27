# prd-tech-review

Agent Skill（Claude Code + Codex）：对已完成的 PRD 做**技术审查 → 交互问答 → PRD 重生成**三阶段闭环。

## 做什么

1. **技术审查**：按 17 个维度审查 PRD + UI 原型 + UX / DFD / 状态机 / 流程图 / 时序图 / ER 图  
2. **交互问答**：用点选 UI 收集 PM 决策（双端适配）  
3. **PRD 重生成**：融合原始 PRD、审查结论与 PM 回答，输出开发可直接使用的完整 PRD  

## 点选 UI 双端适配

| 环境 | 工具 | UI |
|------|------|-----|
| **Claude Code** | `AskUserQuestion` | Plan 模式同类选择框 |
| **Codex** | `request_user_input` | TUI 底部 tab 问卷（可键盘选择） |
| 工具失败/真无 | Markdown 降级 | 文字 A/B/C |

**默认行为：直接触发点选 UI。** 审查报告交付后立刻 tool call，不征求「是否提问」、不先输出 Markdown A/B/C。

运行时：有 `request_user_input` → Codex；否则 `AskUserQuestion` → Claude。

### Codex 注意

- 单次 1–3 题（prefer 1），每题 2–3 个选项  
- `header` ≤ 12 字符；推荐项 label 后缀 ` (Recommended)`  
- **不要**手写 Other 选项（客户端自动提供）  
- 阻断题不要设 `autoResolutionMs`  
- `codex exec` 非交互模式无此工具，会降级  

可选：在 `~/.codex/config.toml` 确认未关闭实验工具：

```toml
[tools.experimental_request_user_input]
enabled = true
```

（多数版本默认已开启。）

## 安装

### Claude Code

```bash
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.claude/skills/prd-tech-review
# 或项目级
git clone git@github.com:zhiquanchi/prd-tech-review.git .claude/skills/prd-tech-review
```

### Codex

```bash
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.codex/skills/prd-tech-review
# 或项目级
git clone git@github.com:zhiquanchi/prd-tech-review.git .codex/skills/prd-tech-review
```

新开会话后，在「审阅 PRD / 检查能否开发」等描述下会自动匹配；也可点名 `prd-tech-review`。

可在 `AGENTS.md` 加一句提高命中率：

```markdown
PRD 技术审查 / 需求缺口澄清 / 审查后重生成 PRD：使用 skill prd-tech-review。
```

### 目录结构

```
prd-tech-review/
├── SKILL.md                 # Skill 主指令（含双端点选适配）
├── README.md
└── references/              # 审查清单、问答指南、重生成指南、各图框架
    ├── review-checklist.md
    ├── qa-generation-guide.md
    ├── prd-regeneration-guide.md
    └── ...
```

## 输入要求

审查需要尽量齐全的输入（缺失项会在阶段 1 标出并在阶段 2 追问）：

| 输入 | 作用 |
|------|------|
| PRD 文档 | 做什么 |
| UI 原型（Ant Design） | 怎么交互 |
| UX 流程图 | 用户路径 |
| DFD | 数据怎么流 |
| 状态机图 | 状态怎么变 |
| 业务流程图 | 逻辑怎么走 |
| 时序图 | 谁先调谁 |
| ER 图 | 数据怎么关联 |

## 触发示例

- 审阅 / 审查 PRD  
- 检查 PRD 是否可以开始开发  
- 找出需求缺口或风险  
- 审查后引导产品补充细节  
- 根据审查结果重新生成 PRD  

## License

MIT（可按需修改）
