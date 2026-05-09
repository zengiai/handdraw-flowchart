# handdraw-flowchart

`handdraw-flowchart` 是一个 Codex Skill，用来把流程描述转换成经过校验的 Mermaid 图，并导出手绘风格的 Excalidraw 文件和 PNG 图片。

适合用于：

- 业务流程图、决策树、状态流转图
- 系统交互时序图
- 简单领域模型或类关系图
- 需要“手绘风格”但又希望源码可维护的技术图

## 能力说明

这个 Skill 的核心流程是：

1. 根据用户描述生成 Mermaid。
2. 严格校验 Mermaid 语法和支持的图类型。
3. 将 Mermaid 转成 Excalidraw 场景。
4. 导出 `.mmd`、`.excalidraw`、`.png` 三类文件。

当前支持的 Mermaid 类型：

- `flowchart TD`
- `flowchart LR`
- `sequenceDiagram`
- `classDiagram`

其中 `flowchart` 是主要推荐路径，可获得更好的 Excalidraw 可编辑效果。

## 安装方式

### 方式一：通过 GitHub 仓库安装

把仓库克隆到 Codex skills 目录：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/<your-github-name>/handdraw-flowchart.git ~/.codex/skills/handdraw-flowchart
cd ~/.codex/skills/handdraw-flowchart
npm install
```

如果 Playwright 没有自动安装 Chromium，继续执行：

```bash
npx playwright install chromium
```

安装完成后，重启或刷新 Codex，让新的 Skill 被发现。

### 方式二：让 Codex 从 GitHub 安装

如果你的 Codex 环境支持 skill 安装器，可以直接让 Codex 执行类似指令：

```text
Install the skill from https://github.com/<your-github-name>/handdraw-flowchart
```

安装后仍建议进入 skill 目录执行一次依赖安装：

```bash
cd ~/.codex/skills/handdraw-flowchart
npm install
```

## 使用方式

在 Codex 中直接引用 Skill：

```text
Use $handdraw-flowchart to turn this process into a hand-drawn workflow diagram:

用户提交订单 -> 校验参数 -> 校验库存 -> 预扣库存 -> 创建订单 -> 发送订单消息 -> 完成
```

也可以用中文描述更复杂的业务流程：

```text
使用 $handdraw-flowchart 画一个支付回调处理流程：
收到支付渠道回调后，先做签名校验，再按支付单号做幂等校验。
如果订单已支付，直接返回成功。
如果订单未支付，更新支付状态，发送支付成功消息。
如果更新失败，需要记录异常并进入补偿任务。
```

Skill 会优先生成 `.mmd` Mermaid 文件，再调用渲染脚本输出手绘图。

## 命令行验证

仓库自带一个示例 Mermaid 文件：

```bash
npm run validate -- --input examples/order-flow.mmd
```

渲染示例：

```bash
npm run render -- --input examples/order-flow.mmd --out-dir out --name order-flow
```

成功后会生成：

```text
out/order-flow.mmd
out/order-flow.excalidraw
out/order-flow.png
```

## Mermaid 约束

为了保证渲染稳定，Skill 会限制 Mermaid 写法：

- 只允许 `flowchart`、`sequenceDiagram`、`classDiagram`。
- `.mmd` 文件中不要使用 Markdown 代码块。
- 不允许 YAML frontmatter。
- 不建议使用自定义 CSS、HTML-heavy label、外部图片。
- 不使用 Mermaid 实验性 shape 语法。
- 节点 ID 尽量使用 ASCII，中文放在节点展示文案里。

完整规则见 [references/mermaid-generation-rules.md](references/mermaid-generation-rules.md)。

## 目录结构

```text
handdraw-flowchart/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── examples/
│   └── order-flow.mmd
├── references/
│   └── mermaid-generation-rules.md
├── scripts/
│   └── render-mermaid-handdraw.mjs
├── package-lock.json
└── package.json
```

关键文件说明：

- `SKILL.md`：Codex Skill 的核心说明和触发描述。
- `agents/openai.yaml`：Codex UI 中展示的名称、简介和默认提示词。
- `scripts/render-mermaid-handdraw.mjs`：Mermaid 校验和 Excalidraw/PNG 渲染脚本。
- `references/mermaid-generation-rules.md`：生成 Mermaid 时需要遵守的规则。

## 常见问题

### 找不到 Chromium

执行：

```bash
npx playwright install chromium
```

也可以通过环境变量指定本机 Chrome：

```bash
CHROME_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" npm run render -- --input examples/order-flow.mmd --out-dir out --name order-flow
```

### Mermaid 校验失败

先单独跑校验：

```bash
npm run validate -- --input your-diagram.mmd
```

按错误信息修改 Mermaid 后再渲染。不要绕过校验，否则后续 Excalidraw 转换更容易失败。

### classDiagram 不完全可编辑

`classDiagram` 有时会走 image fallback。PNG 是可用的，但 Excalidraw 中的编辑粒度可能不如 `flowchart`。

## License

MIT.
