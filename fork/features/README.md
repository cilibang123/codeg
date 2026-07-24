# 自有功能登记

## 落点规范（新文件化原则）

| 层 | 落点 | 规则 |
| ---- | ---- | ---- |
| Rust 端点 | `src-tauri/src/web/handlers/<功能>.rs` 新文件 | `router.rs` 里只加注册行（router.rs 是 seam，改动压到最小） |
| 桌面命令 | `src-tauri/src/commands/<功能>.rs` 新文件，`_core` 函数双模式共用（见上游 AGENTS.md 的条件编译约定） | 同上 |
| 前端组件 | `src/components/<功能>/` 新目录 | 挂载点改动控制在 1~2 行 |
| API/类型 | `src/lib/api.ts`、`src/lib/types.ts` | **只增不改**：追加新函数/新类型，不修改上游已有定义 |
| 外部数据源 | 必须走后端代理（CORS / 密钥 / 缓存），前端不直连第三方 | — |

新功能别忘了 i18n：`src/i18n/messages/*.json` 10 个 locale 都要加 key，自有 key 用统一前缀（见 frontend 文档）。

## 功能规划登记

| 功能 | 前端落点 | 后端落点 | 状态 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| 额度显示 | `src/components/quota/`（新） | `handlers/quota.rs`（新） | 未开始 | 参考 paseo 的 provider-usage 思路（余额条/窗口条/近限告警）；各智能体额度来源不同，先列数据源清单 |
| 降智雷达 | `src/components/radar/`（新） | `handlers/radar.rs` 代理抓取 codexradar.com | 未开始 | 先确认 codexradar 有无公开 API / 页面结构，后端定时抓+缓存，前端只读 |
| 子代理卡片重绘 | 重绘现有卡片，属前端自治区 | — | 随皮肤改造 | 见 `frontend/README.md` 第二档 |
| 上下文压缩样式重绘 | 同上 | — | 随皮肤改造 | 上游 0.21.7 已有压缩分隔线卡片，做的是样式替换 |

## 功能裁剪登记（删除/隐藏上游功能）

原则：**能隐藏就不物理删除**。物理删除的文件上游还在改，每次同步都是 modify/delete 冲突；隐藏（配置开关 / 不注册 / 不渲染）几乎零冲突。

| 裁剪 | 涉及路径 | 方式 | 上游同步风险 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| 内置专家技能（superpowers 系） | `src-tauri/experts/skills/`、`experts.toml`、i18n messages | **建议改为 toml 级禁用**（当前 main 工作树里有 67 个文件的未提交物理删除，见下方警告） | 高：上游 `chore(experts) sync superpowers` 会复活/冲突 | 上游 0.21.7 自己在加"内置技能不出现在导入选择器"的隐藏机制，方向一致，建议跟随上游机制而非删文件 |

> ⚠️ 当前 `/root/codeg` main 工作树里有一批未提交的物理删除（experts 技能 + i18n 改动，+2241/-9856）。
> 不要直接提交到 main（main 必须保持镜像）。建议：先 `git stash` 留档，等 dev 皮肤改造阶段
> 以"隐藏"方式重新实现，再丢弃 stash。

## 新功能登记模板

```
| <功能名> | src/components/<名>/ | handlers/<名>.rs | 未开始 | 数据源/依赖；挂载点；是否需要新 seam 字段 |
```
