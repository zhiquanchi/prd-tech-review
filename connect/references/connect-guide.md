# connect 详细行为指南

本文件是 `connect` skill 的详细行为说明，供阶段执行时对照。核心规则以 `SKILL.md` 为准，本文件补充探测、示例输出与错误处理。

## 1. 仓库根定位

- 本 skill 被加载时，先 `readlink -f` 解析自身目录（可能经过软链），取其**上级目录**为仓库根。
- 例：`~/.claude/skills/connect -> /root/prd-tech-review/connect`，则仓库根 = `/root/prd-tech-review`。
- 仓库根下必须存在 `prd-review/` 与 `prd-regeneration/`；缺失任一 → 报错「仓库结构不完整」，不做任何写入。

## 2. Agent 探测（list 与默认安装共用）

按 `SKILL.md` 探测表逐目录检查：

```bash
for d in ~/.claude/skills ~/.codex/skills ~/.qwen/skills ~/.grok/skills ~/.agents/skills; do
  [ -d "$d" ] && echo "detected: $d"
done
```

检出规则：目录存在 → 视为该 agent 已安装、可写入。目录不存在 → 跳过（不创建目录；需要时用 `--path` 显式指定）。

**去重：** 同一路径只处理一次（如 `~/.agents/skills` 被多个 agent 引用）。

## 3. `connect list`（只读）

逐目录输出：

```text
[claude]  ~/.claude/skills          prd-review: not installed    prd-regeneration: not installed
[codex]   ~/.codex/skills           prd-review: not installed    prd-regeneration: not installed
[qwen]    ~/.qwen/skills            prd-review: not installed    prd-regeneration: not installed
[shared]  ~/.agents/skills          prd-review: installed (symlink)   prd-regeneration: installed (symlink)
[legacy]  ~/.codex/skills/prd-tech-review  旧副本（拆分前结构）→ 将被 purge
```

状态取值：`not installed` / `installed (symlink)` / `installed (copy)` / `conflict (同名非软链)` / `legacy (旧副本)`。

## 4. `connect`（默认安装，purge-then-install）

对每个**检测到**的目标目录执行：

```bash
# 1) purge：清理本仓库条目与 legacy
rm -f "$dir/prd-review" "$dir/prd-regeneration"          # 软链用 rm -f
[ -f "$dir/prd-tech-review/SKILL.md" ] && grep -q '^name: prd-tech-review' "$dir/prd-tech-review/SKILL.md" \
  && rm -rf "$dir/prd-tech-review" && echo "removed legacy prd-tech-review ($dir)"

# 2) install：绝对路径软链
ln -s "$REPO/prd-review" "$dir/prd-review"
ln -s "$REPO/prd-regeneration" "$dir/prd-regeneration"
```

规则：
- 目标已有 `prd-review` / `prd-regeneration` 且**不是软链**（真实目录）→ 不覆盖、不删除，报告 `conflict` 并跳过该条目，其余条目继续。
- `prd-tech-review` 旧副本判定：条目内 `SKILL.md` frontmatter 含 `name: prd-tech-review` → legacy，purge；条目是软链且目标无根级 `SKILL.md`（如指向仓库根——拆分后仓库根已无 SKILL.md）→ 不删除，提示用户手工移除；其余同名条目 → 不动，打印警告。
- 输出示例：

```text
installed prd-review -> /root/prd-tech-review/prd-review  (~/.claude/skills/prd-review)
installed prd-regeneration -> /root/prd-tech-review/prd-regeneration  (~/.claude/skills/prd-regeneration)
removed legacy prd-tech-review (~/.codex/skills/prd-tech-review)
skipped conflict: ~/.qwen/skills/prd-review is a real directory (not a symlink)
```

## 5. `connect --path <skills-dir>`

- 相对路径按当前 `PWD` 解析；目录不存在 → 报错并退出，不自动创建。
- **不**扫描 agent、**不**处理 legacy `prd-tech-review`、不删 `ark-*` / `byted-*` 等其他条目。
- 只对指定目录执行第 4 节的 purge（限 `prd-review` / `prd-regeneration`）+ install。

## 6. `connect uninstall` / `uninstall --path`

- **先要求用户明确确认**（打印将删除的完整路径清单，等用户同意）。
- 默认：对每个检测到目录执行 `rm -f "$dir/prd-review" "$dir/prd-regeneration"`；legacy `prd-tech-review` 一并删除（同样先确认）。
- `--path`：只清指定目录，且**不**处理 legacy。
- 永不触碰源仓库；删的是软链（rm -f 即可），若目标是真实目录则跳过并提示（避免误删用户内容）。

## 7. 错误处理

| 情况 | 行为 |
|------|------|
| 仓库根下缺 `prd-review/` 或 `prd-regeneration/` | 中止，报「仓库结构不完整」 |
| 目标目录不存在（默认安装） | 跳过该 agent |
| `--path` 目录不存在 | 报错退出，不创建 |
| 同名非软链冲突 | 跳过该条目，打印 `conflict`，不覆盖 |
| 权限不足 | 打印错误与建议（`chmod` / 以当前用户重试），已完成的条目保留 |
| uninstall 未获确认 | 中止，不删任何条目 |

## 8. 安全边界（强制）

- 只写 `prd-review` / `prd-regeneration` / 识别为 legacy 的 `prd-tech-review` 三个名字的条目。
- 删除操作仅限：目标 skills 目录下的上述条目（软链 rm -f；legacy 目录 rm -rf 前必须经 frontmatter 校验）。
- 任何 `rm -rf` 前先打印将被删除的路径，uninstall 场景还需用户确认。
