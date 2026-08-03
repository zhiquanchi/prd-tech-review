---
name: connect
version: 1.0.0
description: "connect：把本仓库内嵌的 AI skills（prd-review、prd-regeneration）以符号链接安装到本机检测到的所有 AI Agent（Claude Code、Codex、Qwen Code、Grok 等），支持安装、列出已支持 agent、卸载、--path 指定目录。当用户需要将 prd-tech-review 能力同步到本地 agent 时使用。"
---

# connect：把本仓库 skills 安装到本地 Agent

**前置：** 先解析本 skill 目录的真实路径（`readlink -f` 解析软链），**仓库根 = 本 skill 目录的上级目录**（`connect/` 与 `prd-review/`、`prd-regeneration/` 平级）。

把仓库内嵌的 `prd-review` / `prd-regeneration` skill 以**符号链接**安装到本机 AI Agent 的 skills 目录。**纯本地文件操作，不需要认证。**

## 调用形态（子命令只有这些）

| 调用 | 说明 |
|------|------|
| `connect` | 默认行为：安装到所有检测到的 agent 的 skills 目录 |
| `connect --path <skills-dir>` | 安装到指定的本地 skills 目录（项目级 / 自定义路径），不扫描 agent、不改其他目录 |
| `connect list` | 只读：列出支持的 agent、目录检测状态与已安装条目 |
| `connect uninstall` | 破坏性：从所有检测到的 agent 卸载（删软链，不碰源仓库） |
| `connect uninstall --path <skills-dir>` | 删除指定目录下的 `prd-review` / `prd-regeneration` 软链，不扫描/修改其他目录 |

> ⚠️ **没有** `connect install` / `connect setup` / `connect sync` 等子命令；安装就是默认行为，不要凭直觉补 install。`--path` 是 flag，不是子命令。

## 路由判断

- 用户想把本仓库 skills 装进本地 agent → 跑 `connect`，建议先 `connect list` 预检
- 用户想把 skills 装进某个 repo/project 的本地 skills 目录 → 跑 `connect --path <skills-dir>`（如 `connect --path .claude/skills`）；`--path` 接收具体 skills 目录，不是项目根目录
- 用户只想知道支持哪些 agent → **只**跑 `connect list`，**不要**顺手装
- 用户想清理已装 skills → `connect uninstall`，**先要求用户明确确认**（破坏性操作，删目标目录下 `prd-review` / `prd-regeneration` 条目）

## 反触发（应该路由到别处）

- 审查 PRD / 重生成 PRD → 走 `prd-review` / `prd-regeneration`，**与 connect 无关**
- 从 0 写 PRD / 出交互原型 → 走 `prd-writer`（若存在）
- 用户要改本仓库 skill 内容 → 直接编辑仓库，`connect` 是安装器不是编辑器

## 关键事实（写在 SKILL.md 内，避免 Agent 幻觉）

- 安装走 **purge-then-install**：对每个目标 skills 目录，先删除其中的 `prd-review` / `prd-regeneration` 条目（软链 `rm -f`，目录 `rm -rf`），再创建指向仓库对应目录的**符号链接**（绝对路径）。
- **legacy 旧副本清理**：目标目录下存在 `prd-tech-review` 条目（拆分前的单 skill 结构）→ 读取其 `SKILL.md` frontmatter，`name: prd-tech-review` 则判定为 legacy，**删除并打印提示**；无法识别为旧副本的同名目录/文件 → **不删除**，打印警告请用户手工处理。
- **符号链接**：`ln -s <仓库根>/prd-review <目标>/prd-review`；目标已有同名**非软链**内容（如真实目录）→ 不覆盖，报告冲突并跳过该条目。
- `--path <skills-dir>` 是隔离安装：相对路径按当前 `PWD` 解析，绝对路径原样使用；只清理并写入该目录下的 `prd-review` / `prd-regeneration`，**不**扫描 agent、不处理 legacy `prd-tech-review`、不删其他条目。
- 多个 agent 共享同一个 skills 目录（或目录路径重复出现）→ 按路径**自动去重**，只 purge / 装一次。
- **多路径扫描**：Claude Code 扫 `~/.claude/skills`；Codex 扫 `~/.codex/skills`（另有共享 `~/.agents/skills`）；Qwen Code 扫 `~/.qwen/skills`；Grok 扫 `~/.grok/skills`；共享 `~/.agents/skills` 存在时也装入一份（跨 agent 汇总目录，与 antd 模式一致）。agent 检出 = 对应目录存在。
- `list` 只扫描本地文件系统，不写入、不联网、不需要认证。
- 支持的 agent 目录硬编码在下方探测表；新增 agent 需更新本 SKILL.md 与 `references/connect-guide.md`。
- `uninstall` 只删软链与识别出的 legacy 条目，**永不删除源仓库文件**；破坏性仅限于目标 skills 目录内的本仓库条目。

## Agent 探测表

| Agent | 目录（存在即视为已安装该 agent） | 备注 |
|-------|------|------|
| Claude Code | `~/.claude/skills` | 项目级 `.claude/skills` 用 `--path` 处理 |
| Codex | `~/.codex/skills` | 另扫共享 `~/.agents/skills`（若存在） |
| Qwen Code | `~/.qwen/skills` | |
| Grok | `~/.grok/skills` | |
| 共享目录 | `~/.agents/skills` | 跨 agent 汇总，与各 agent 目录并存时按路径去重 |

详细行为、示例输出、错误处理见 [`references/connect-guide.md`](references/connect-guide.md)。
