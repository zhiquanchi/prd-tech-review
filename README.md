# prd-tech-review

Claude Code Skill：对已完成的 PRD 做**技术审查 → 交互问答 → PRD 重生成**三阶段闭环。

## 做什么

1. **技术审查**：按 17 个维度审查 PRD + UI 原型 + UX / DFD / 状态机 / 流程图 / 时序图 / ER 图  
2. **交互问答**：用 Claude Code 内置 **`AskUserQuestion`** 弹出可点选选择题（与 Plan 模式同类选择框）  
3. **PRD 重生成**：融合原始 PRD、审查结论与 PM 回答，输出开发可直接使用的完整 PRD  

## 安装

### Claude Code

将本仓库克隆到 skills 目录（目录名可自定义，但需含 `SKILL.md`）：

```bash
# 用户级
git clone git@github.com:zhiquanchi/prd-tech-review.git ~/.claude/skills/prd-tech-review

# 或项目级
git clone git@github.com:zhiquanchi/prd-tech-review.git .claude/skills/prd-tech-review
```

重启 Claude Code 或新开会话后，在相关需求下会自动匹配；也可在对话中说明「用 prd-tech-review 审查这份 PRD」。

### 目录结构

```
prd-tech-review/
├── SKILL.md                 # Skill 主指令
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

## 阶段 2 与选择框

阶段 2 **强制**调用 `AskUserQuestion`，在 TUI/IDE 中出现可点选 UI，而不是纯 Markdown 的 A/B/C。

若运行环境没有该工具，Skill 会降级为 Markdown 选择题并提示用户。

## 触发示例

- 审阅 / 审查 PRD  
- 检查 PRD 是否可以开始开发  
- 找出需求缺口或风险  
- 审查后引导产品补充细节  
- 根据审查结果重新生成 PRD  

## License

MIT（可按需修改）
