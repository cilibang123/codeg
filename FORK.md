# Fork 组织总纲

本仓库是 [xintaofei/codeg](https://github.com/xintaofei/codeg) 的二次开发 fork。
核心策略一句话：**前端自治，后端跟随，补丁上游优先，自有功能新文件化。**

## 为什么这样组织

- 上游处于爆发期（约 13 个提交/天，其中 55% 碰前端、41% 碰后端），整体合并必然失败。
- 前端要整体换成 opencode 风格，上游前端改动对我们只有参考价值，没有合并价值。
- 后端（Rust / ACP / 解析器）是智能体兼容和安全修复的来源，必须低成本持续跟随。
- 后端 bug 修复优先提 PR 给上游：被合入的修复，分叉永久缩小一块。

## 分支拓扑

| 分支 | 角色 | 规则 |
| ---- | ---- | ---- |
| `main` | 上游镜像 | 只准 `git merge --ff-only upstream/main` 前进，**禁止提交任何自有代码**。用途：cherry-pick 基准、diff 基准、patch 分支的母体。 |
| `dev` | 工作主分支（建议设为 GitHub 默认分支） | 全部自有改动在这里：前端重写、自有功能、未上游的补丁。 |
| `fix/<名>` / `patch/<名>` | 后端补丁短分支 | 从 `main` 切出，一个分支一个修复，先提上游 PR。登记在 `fork/patches/README.md`。 |

本地布局建议（git worktree）：

- `/root/codeg` → 检出 `main`（镜像/基准）
- `/root/codeg-dev` → 检出 `dev`（日常工作）

## 三层纪律

1. **前端自治**：`src/components/`、`src/app/` 是自有领土，随便改；同步时这两个路径一律保留 dev 版本，不接收上游改动。规则细节见 `fork/frontend/README.md`。
2. **接缝与后端跟随**：`src-tauri/`、`src/lib/transport/`、`src/lib/api.ts`、`src/lib/types.ts` 保持与上游一致。自有修改只允许以 patch 短分支存在，且先提 PR。例外：`api.ts` / `types.ts` 允许**只增不改**（追加自有函数/类型）。
3. **自有功能新文件化**：新功能一律放新建文件——Rust 新 handler、前端新组件目录，挂载点改动控制在 1~2 行。登记在 `fork/features/README.md`。

## 同步 SOP（每 1~2 周一次，跟 tag 不跟 main）

```bash
# 1) main 工作树：刷新镜像
cd /root/codeg
git fetch upstream --tags
git checkout main
git merge --ff-only upstream/main     # 若失败说明 main 被污染，禁止强推，回查
git push origin main

# 2) dev 工作树：合并
cd /root/codeg-dev
git fetch origin
git merge origin/main
# 前端路径冲突一律保 dev：
git checkout --ours src/components src/app
git add src/components src/app
# seam 文件（api.ts / types.ts / transport/）手工解：双方新增都保留
# i18n messages 冲突：双方 key 都保留（自有 key 见 frontend 文档的前缀约定）
git commit
# 3) 验证后再推
pnpm install && pnpm eslint . && pnpm test && pnpm build
(cd src-tauri && cargo check --no-default-features --bin codeg-server)
git push origin dev
```

已启用 `git rerere`（每个克隆需各自 `git config rerere.enabled true`），重复冲突自动复用解决方案。

合并后必做：跑一遍对话区（发消息、子代理卡片、权限卡），seam 合并的隐性破坏靠测试抓不全。

每月做一次：读上游 [releases](https://github.com/xintaofei/codeg/releases)，挑想要的前端功能，让 AI 在自有设计系统里重做，登记到 `fork/features/README.md`。

## 冲突处理速查

| 冲突位置 | 处理方式 |
| ---- | ---- |
| `src/components/`、`src/app/` | 一律 `git checkout --ours`（保 dev）。出现冲突是正常的；没出现也正常。 |
| `src-tauri/` | 不该冲突。冲突 = 有补丁没走 patch 分支流程，回查纪律；若上游改了已打补丁的文件，把 patch 分支 rebase 到新 main 重放。 |
| `api.ts` / `types.ts` / `transport/` | 手工，机械性：上游新增与自有新增都保留。 |
| `src/i18n/messages/*.json` | 热点文件，双方 key 都保留。 |
| `src-tauri/experts/` | 功能裁剪区，上游 sync 必冲突，处理见 `fork/features/README.md` 裁剪登记。 |

## License 义务

Apache-2.0：保留根目录 `LICENSE` 及版权声明；再分发时附 NOTICE 说明修改。其余随意。

## 文档地图

| 文档 | 内容 |
| ---- | ---- |
| `fork/README.md` | `fork/` 目录定位与索引 |
| `fork/patches/README.md` | 后端补丁队列规范与登记表 |
| `fork/features/README.md` | 自有功能登记、落点规范、功能裁剪登记 |
| `fork/frontend/README.md` | 前端自治纪律、opencode 皮肤改造路线 |
