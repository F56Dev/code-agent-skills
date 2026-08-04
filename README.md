# opencode-skills

个人 opencode skill 合集。为纯文本主模型（如 DeepSeek V4 Flash）提供"看图"能力扩展——通过委派给多模态子代理，把图片内容转换成文字描述回传，主模型即可"间接看到"图片。

## 包含的 skill

| Skill | 作用 |
|-------|------|
| [`vision-handoff`](./skills/vision-handoff/) | 图片委派：全图详述 describe / 局部凝视 gaze / 多图对比 compare 三种模式 |

## 依赖

- [`agents/vision.md`](./agents/vision.md) — vision 子代理，使用多模态模型（默认 `opencode/mimo-v2.5-free`，OpenCode Zen 免费）
- 主模型需支持 `task` 工具（可委派子代理）

## 安装

### 方式一：克隆到全局配置目录

```bash
git clone https://github.com/DEV/sanen-opencode-skills.git ~/.config/opencode/skills/tmp-repo
# 或者手动把 skills/vision-handoff 和 agents/vision.md 复制到
# ~/.config/opencode/ 对应目录
```

### 方式二：直接复制文件

```bash
mkdir -p ~/.config/opencode/skills
cp -r skills/vision-handoff ~/.config/opencode/skills/
cp -r agents ~/.config/opencode/agents/
```

### 方式三：npm 安装（如果发布为 npm 包）

```bash
npm install -g opencode-skills
```

然后重启 opencode。

## 使用

安装后重启 opencode，纯文本主模型即可自动感知图片请求：

- 用户贴图片/提到图片 → 自动委派 `vision` 子代理
- 支持三种模式：
  - **describe**：全图详述
  - **gaze**：局部凝视（聚焦指定区域详述，用于读原型 UI 细节、小号文字）
  - **compare**：多图对比（E2E 截图前后对比、原型 vs 实现）

## 工作原理

```
用户粘贴图片（或给图片路径）
  → 主模型（纯文本）感知图片请求（skill 触发）
  → task 工具委派给 vision 子代理（多模态模型）
  → vision 子代理 read 图片 → 输出文字描述
  → 文字描述回传给主模型 → 主模型继续工作
```

会话中直接附加的图片（无文件路径）存储在 opencode 的 SQLite 数据库（`~/.local/share/opencode/opencode.db` 的 `part` 表），提取 base64 后转临时文件再委派。

## License

MIT
