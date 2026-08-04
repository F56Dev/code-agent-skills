---
name: vision-handoff
description: >-
  当任务涉及图片（截图/图片/图/png/jpg/jpeg/gif/webp/svg/pdf 文件、UI 界面截图、物理题图、
  电路图、图表、渲染输出）且你无法读取图片时使用。委托给 vision 多模态子代理读取图片并获取文字描述。
  支持三种模式：全图详述 describe、局部凝视 gaze（聚焦指定区域）、多图对比 compare（结构化差异报告）。
  触发词：图片、截图、看图、图、diagram、screenshot、image、png、jpg、svg 文件、对比、差异、局部、区域。
---

# Vision Handoff — 图片委派

你是纯文本模型（如 DeepSeek V4 Flash），**无法读取图片**。当任务需要理解一张图片时，不要自己尝试 `read` 图片文件（你看到了也解读不了），而是委派给 `vision` 多模态子代理。

## 三种模式

按需求选用，在 task prompt 里明确指定用哪种：

### 1. 全图详述（describe）
需要了解一张图整体内容时用。prompt 给**绝对路径** + "需要了解什么"。

### 2. 局部凝视（gaze）
需要聚焦图里某个具体元素/区域时用（读 UI 原型关注某个控件、检查渲染细节、转写小号文字、定位 bug 的具体位置）。prompt 中**必须明确指定区域**，例如：
```
只聚焦 <区域>，例如："底部 15%" / "左上角" / "x≈300, y≈500 附近" / "右上角的按钮" / "左侧工具栏第三个图标"
```

### 3. 多图对比（compare）
需要对比两张以上图片差异时用（E2E 截图前后对比、原型 vs 实现、before/after、一次改动多处 UI 后确认效果）。prompt 中给**所有绝对路径** + 要求**结构化差异报告**。

## 何时触发

满足以下任一条件，说明你需要看一张图：

- 用户提到/附加/引用了图片：截图、图片、看图、张图、screenshot、image、diagram
- 出现图片文件路径：`.png` `.jpg` `.jpeg` `.gif` `.webp` `.svg` `.pdf`
- 任务涉及检查视觉产物：UI 布局、渲染 bug、SVG 导出效果、截图比对、物理题图、电路图
- 用户说"你看一下这张图 / 帮我看看这个截图 / 照着这张图做"
- 用户要求对比图片差异（before/after、原型/实现、E2E 截图）
- 用户要求聚焦图片的某个具体区域/元素详述

## 委派步骤

1. **不要自己 read 图片**——你无法看到它。
2. 调用 `task` 工具，参数如下：
   - `subagent_type: "vision"`
   - `description`: 简短语（如 "compare screenshots" / "读图" / "gaze region"）
   - `prompt`: 包含所有图片的**绝对文件路径** + 模式（describe/gaze/compare）+ 区域或对比要求。
3. `vision` 子代理返回文字描述（或结构化差异报告）。
4. 把这段描述当作"你已经看到了图片"来使用：基于它回答问题、分析、修改代码。

## 示例

### 全图详述
```
task(
  subagent_type: "vision",
  description: "describe svg screenshot",
  prompt: "请读取图片 C:\\Users\\admin\\Pictures\\screenshot.png，这是一张疑似渲染错误的截图，请描述：1) 画面整体内容 2) 所有可见文字 3) 异常的地方（重影、错位、颜色不对）"
)
```

### 局部凝视
```
task(
  subagent_type: "vision",
  description: "gaze at toolbar",
  prompt: "请读取图片 C:\\Users\\admin\\Pictures\\ui.png，执行局部凝视：只聚焦图片顶部左侧的工具栏区域（左上 1/4），忽略其他。列出每个工具按钮的图标形状、文字、位置。"
)
```

### 多图对比
```
task(
  subagent_type: "vision",
  description: "compare screenshots",
  prompt: "请对比两张截图（E2E 测试前后）：1) C:\\...\\before.png 2) C:\\...\\after.png。输出结构化差异报告：每张图各自内容概览 + 逐条差异列表（位置+内容）+ 关键变化总结。"
)
```

## 注意

- 如果图片路径是相对的，先解析为绝对路径再传给子代理。
- 一次可以传多个图片路径，让子代理逐个描述或对比。
- 需要"先对比、再聚焦某个差异区域详述"时，可以分两次 task 调用。
- 如果用户粘贴的是 base64 或拖拽的图片（无文件路径），先从 opencode 数据库提取附件再传路径给子代理：
  1. 附件存在 `C:\Users\admin\.local\share\opencode\opencode.db` 的 `part` 表 `data` 列（JSON）
  2. 查 `SELECT data FROM part ORDER BY time_created DESC LIMIT 300`，过滤 JSON 里 `type==="file"` 且 `mime` 以 `image/` 开头的记录
  3. 用正则 `^data:image/(\w+);base64,(.+)$` 解出 base64，写入临时文件（如 `C:\Users\admin\AppData\Local\Temp\opencode\attachment-N.png`）
  4. 把临时文件路径传给子代理
- 委派是单向的：子代理只负责读图转文字，其他工作（改代码、查文档）由你完成。
