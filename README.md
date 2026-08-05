# opencode-skills

个人 opencode skill 合集，含配套子代理（agents）。全部自包含——克隆即用，不依赖任何项目级 AGENTS.md 配置。

## 包含的 skill

| Skill | 作用 |
|-------|------|
| [`vision-handoff`](./skills/vision-handoff/) | 图片委派：全图详述 describe / 局部凝视 gaze / 多图对比 compare 三种模式 |
| [`docs-review`](./skills/docs-review/) | 文档产出即审：产出 → 委派 docs-review 子代理交叉核对 → 主 agent 三级裁决 → 争议回环商讨 → 交人工 |

## 依赖的子代理

| Agent | 作用 | 模型 |
|-------|------|------|
| [`agents/vision.md`](./agents/vision.md) | 多模态读图（vision-handoff 用） | `opencode/mimo-v2.5-free`（Zen 免费） |
| [`agents/docs-review.md`](./agents/docs-review.md) | 文档审查（docs-review 用，异族视角） | `opencode-go/glm-5.2`（Go 套餐，可改） |

主模型需支持 `task` 工具（可委派子代理）。

## 安装

### 方式一：克隆到全局配置目录

```bash
git clone <this-repo> ~/.config/opencode/skills/code-agent-skills
# 或手动复制 skills/ 和 agents/ 到 ~/.config/opencode/ 对应目录
```

### 方式二：直接复制文件

```bash
mkdir -p ~/.config/opencode/skills ~/.config/opencode/agents
cp -r skills/* ~/.config/opencode/skills/
cp -r agents/* ~/.config/opencode/agents/
```

安装后重启 opencode。全局配置目录的 skills/agents 对**所有项目**生效。

## 使用

### vision-handoff（看图）

- 用户贴图片/提到图片 → 自动委派 `vision` 子代理
- 三种模式：describe（全图详述）/ gaze（局部凝视，聚焦指定区域）/ compare（多图对比，结构化差异报告）

### docs-review（文档产出即审）

- **触发纪律**：任何项目产出意图/事实类文档（spec/plan/架构文档/阶段报告/BUG-LEDGER/测试报告）后必审
- **流程**：产出 → docs-review 审查（发现带 `文件:行号` 证据）→ 主 agent 三级裁决（采纳/否决-无争议/否决-有争议）→ 仅争议项进商讨回环（R2 附证据送回 → 无新证据作废 → R3 未一致升人工）→ 一致性文档交人工
- 也可在**任意项目的 AGENTS.md** 加一条"产出即审"纪律引用本 skill，强化触发

## 工作原理

```
用户粘贴图片 → 主模型感知 → task 委派 vision 子代理（多模态） → 文字描述回传

产出文档 → 主模型触发 docs-review skill → task 委派 docs-review 子代理（异族模型）
        → 三级裁决 + 回环商讨 → 一致性文档交人工
```

会话中直接附加的图片（无文件路径）存储在 opencode 的 SQLite 数据库（`~/.local/share/opencode/opencode.db` 的 `part` 表），提取 base64 后转临时文件再委派。

## License

MIT
