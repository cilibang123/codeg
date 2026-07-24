# 前端自治区

## 纪律

- **自治领土**：`src/components/`、`src/app/` —— opencode 皮肤、布局、字体、组件、样式、动画、交互都在这里改，不用顾忌上游。
- **永不合并**：同步时这两个路径一律 `git checkout --ours` 保 dev 版本（SOP 见根目录 `FORK.md`）。上游前端改动对我们只有"读 release notes 参考"的价值。
- **seam 不碰**：`src/lib/transport/`、`src/lib/api.ts`、`src/lib/types.ts`（后两者允许只增不改）。这是前后端契约层，碰了就失去后端跟随能力。
- **i18n 热点**：`src/i18n/messages/*.json` 上游几乎天天动。自有 key 统一加 `x-` 前缀并按字母序插入；冲突时双方 key 都保留。10 个 locale 同步加。

## opencode 皮肤改造路线

参考：opencode 设计资产（MIT）= 60+ 主题 JSON（palette + syntax/markdown 语义色）+ Inter/JetBrains Mono 字体 + 图标 sprite；组件是 SolidJS 不可直接用，只能照着重写；其 Storybook 可当风格规范。

### 第一档：token 级换肤（1~2 周，先做）

1. **主题**：`src/app/globals.css` 新增 `[data-theme="opencode"]` 预设块。仓库已有 12 套 data-theme 预设（neutral/zinc/slate/blue…）的成熟模式，照抄结构。
   - opencode 主题 JSON → shadcn token 映射：palette 的 neutral/ink/primary/accent/success/warning/error → background/foreground/primary/accent/destructive 等；第一套手工对，之后写脚本批量转 60+ 套。
   - syntax/markdown overrides → codeg 的 Shiki/Markdown 相关变量（需对一次映射表，是本档最细的活）。
   - 待确认：主题枚举/注册位置（settings 外观页与 AppearanceProvider）。
2. **字体**：走 `font-presets.ts` 预设机制加"等宽 UI"预设；JetBrains Mono 已打包在仓库里。`--font-sans` 由 AppearanceProvider 运行时覆盖。
3. **密度/圆角/质感**：调小 `--radius`；重设 `src/components/ui/`（36 个 shadcn primitives——上游极冷，近 2 月仅 2 次提交，放心改）。
4. 验收：整体观感接近 opencode 的深色终端气质，布局不变。

### 第二档：对话区深改（1~2 个月，第一档验收后再决定）

- 范围：`chat/`（89）+ `message/`（56）+ `ai-elements/`（25）+ `layout/`（46）+ `conversations/`（24）。
- 内容：消息流/工具卡片改成无边框终端行式渲染、子代理卡片与压缩分隔线重绘、布局重排。
- 代价：这些是上游最热文件（提交数封顶），改完后**每月 release notes 里的对话区新功能都要手工重做**——这是拿功能同步换 UI 自由，动工前想清楚。
- 技巧：尽量用包裹组件和 CSS 覆盖，减少在上游原文件内部的直接编辑。

### 每月功能复查

读上游 [releases](https://github.com/xintaofei/codeg/releases) → 挑想要的前端功能 → 让 AI 在本设计系统重做 → 登记到 `fork/features/README.md`。
