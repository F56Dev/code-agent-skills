# code-agent-skills 安装指南（AI 可执行版）

> **本指南供 AI 助手自动执行安装。** 用法：把这个文件的内容（或仓库 URL）提供给你的 AI 助手（opencode / Claude Code / 任何 harness），它会自动完成环境检测、下载、拷贝、配置与校验。
>
> 人工安装也可以照此操作。安装时间约 1 分钟。

---

## 0. 安装目标

安装 3 个 opencode skill + 3 个配套子代理：

| Skill | 目录 | 用途 |
|-------|------|------|
| vision-handoff | `skills/vision-handoff/` | 图片委派（全图详述/局部凝视/多图对比） |
| docs-review | `skills/docs-review/` | 文档产出即审（三级裁决 + 商讨回环） |
| debug-discipline | `skills/debug-discipline/` | 调试升级闸门（2 次失败委派 IIC） |

| Agent | 文件 | 用途 |
|-------|------|------|
| vision | `agents/vision.md` | 多模态读图（MiMo-V2.5 Free） |
| docs-review | `agents/docs-review.md` | 文档审查（GLM-5.2） |
| iic | `agents/iic.md` | 独立调查委员会（GLM-5.2） |

---

## 1. 环境检测（AI 执行）

1. 检测操作系统（Windows / macOS / Linux）。
2. 确认 opencode 已安装：运行 `opencode --version`。
3. 定位 opencode 全局配置目录 `CFG`：
   - macOS / Linux：`~/.config/opencode/`
   - Windows：`%USERPROFILE%\.config\opencode\`（通常是 `C:\Users\<name>\.config\opencode\`）
4. 确认 `CFG/skills/` 和 `CFG/agents/` 目录存在；不存在则创建。

---

## 2. 获取文件（AI 执行）

从仓库 `https://github.com/F56Dev/code-agent-skills` 获取以下 6 个文件：

- `skills/vision-handoff/SKILL.md`
- `skills/docs-review/SKILL.md`
- `skills/debug-discipline/SKILL.md`
- `agents/vision.md`
- `agents/docs-review.md`
- `agents/iic.md`

获取方式任选（AI 自行选择可行方式）：
- **A**：`git clone https://github.com/F56Dev/code-agent-skills.git <临时目录>`，从临时目录复制
- **B**：用 `curl`/`Invoke-WebRequest` 逐个下载 raw 文件：
  `https://raw.githubusercontent.com/F56Dev/code-agent-skills/master/skills/<name>/SKILL.md`
  `https://raw.githubusercontent.com/F56Dev/code-agent-skills/master/agents/<name>.md`

---

## 3. 放置文件（AI 执行）

| 源文件 | 目标路径 |
|--------|---------|
| `skills/vision-handoff/SKILL.md` | `CFG/skills/vision-handoff/SKILL.md` |
| `skills/docs-review/SKILL.md` | `CFG/skills/docs-review/SKILL.md` |
| `skills/debug-discipline/SKILL.md` | `CFG/skills/debug-discipline/SKILL.md` |
| `agents/vision.md` | `CFG/agents/vision.md` |
| `agents/docs-review.md` | `CFG/agents/docs-review.md` |
| `agents/iic.md` | `CFG/agents/iic.md` |

注意：
- 若 `CFG/skills/<name>/` 目录已存在旧版，**覆盖**而非追加。
- 不要修改文件内容（除非下述模型 ID 调整）。

---

## 4. 模型依赖检查（AI 执行，向用户确认）

子代理使用以下模型，**必须确认用户可用**，否则安装后无法工作：

| Agent | model 字段 | 需要 |
|-------|-----------|------|
| vision | `opencode/mimo-v2.5-free` | OpenCode Zen 免费模型（无需付费） |
| docs-review | `opencode-go/glm-5.2` | OpenCode Go 订阅 |
| iic | `opencode-go/glm-5.2` | OpenCode Go 订阅 |

**检查步骤**：
1. 向用户确认是否有 OpenCode Go 订阅（或是否有 `opencode-go/*` 模型权限）。
2. 若用户没有 Go 订阅，提供替代：
   - 将 `agents/docs-review.md` 和 `agents/iic.md` 的 `model: opencode-go/glm-5.2` 改为用户可用的异族模型（如 `opencode/glm-5.2` 走 Zen 按量，或用户自己的 provider）。
3. 提醒：`vision.md` 用的是免费模型，无需调整。

---

## 5. 校验安装（AI 执行，必须全部通过）

逐项校验，任何一项失败都要修复后重跑：

1. **文件存在**：6 个文件都在目标路径，`ls` / `Test-Path` 确认。
2. **frontmatter 合法**：每个 SKILL.md 的 YAML frontmatter 含 `name` 和 `description`；每个 agent 的 frontmatter 含 `description`、`mode: subagent`、`model`、`permission`。
3. **skill 目录名与 name 一致**：
   - `skills/vision-handoff/SKILL.md` 的 `name:` 必须是 `vision-handoff`
   - `skills/docs-review/SKILL.md` 的 `name:` 必须是 `docs-review`
   - `skills/debug-discipline/SKILL.md` 的 `name:` 必须是 `debug-discipline`
4. **模型 ID 格式合法**：agent 的 `model` 字段符合 `provider/model-id` 格式。
5. **内容非空**：每个文件大小 > 500 字节。

给出校验报告（每项 ✅/❌）。

---

## 6. 完成（AI 执行）

1. 告诉用户：**重启 opencode** 使 skill 和子代理生效。
2. 提供验证方法：
   - 重启后在任意项目输入 `/models` 或 @ 子代理，应能看到 `vision`、`docs-review`、`iic`。
   - 或在会话中尝试：粘贴一张图片（触发 vision-handoff）、说"审查某文档"（触发 docs-review）。
3. 若用户还想在**所有项目**强制这些纪律，建议把 `docs-review` 和 `debug-discipline` 的纪律要点加进用户自己的全局 AGENTS.md（`CFG/AGENTS.md`）。

---

## 卸载

删除 `CFG/skills/vision-handoff/`、`CFG/skills/docs-review/`、`CFG/skills/debug-discipline/`、`CFG/agents/vision.md`、`CFG/agents/docs-review.md`、`CFG/agents/iic.md` 即可。无其他残留。
