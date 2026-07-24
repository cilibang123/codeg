# 后端补丁队列

## 规则

1. **一个补丁一个分支**：`fix/<名>` 或 `patch/<名>`，从最新 `main`（上游镜像）切出，只含一个修复。
2. **先提 PR 给上游**（[xintaofei/codeg](https://github.com/xintaofei/codeg/pulls)），PR 链接登记到下表。上游收 PR 有先例（youxikexue、AnotiaWang、ijry、asxuen、cnYui 等均在近月被合入）。
3. **状态机**：`待修复 → 已提 PR → 上游已收（同步后删除分支与登记行）`，或 `→ 上游不收（转长期自维护，注明维护要点）`。
4. **小而孤立**：一个补丁只动必要文件。`src-tauri/src/acp/` 是上游热点区，长期补丁尽量避开；避不开就更要尽快上游化。
5. dev 需要补丁生效时：把 patch 分支合并进 dev（或在 patch 分支上开发完直接合 dev），不要绕过分支直接在 dev 上改后端。

## 登记表

| 补丁 | 分支 | 触及文件 | 状态 | 上游 PR | 备注 |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Grok 终端 ENAMETOOLONG 回退 | `fix/grok-terminal-enametoolong` | terminal_runtime、connection、回归测试 | **退役** | — | 上游 0.21.7 已自行修复（超长命令回退 shell），方案同向。分支留档不删，dev 不需要合。 |
| OpenCode 插件预检识别 | `fix/opencode-plugin-preflight-0215` | `src-tauri/src/acp/opencode_plugins.rs`（+197/-40） | **待提 PR** | 待填 | 2 commits：修正预检识别 + 兼容固定版本插件缓存。关联"opencode 跑一会失败"问题，上游大概率愿意收。提 PR 前先 rebase 到最新 main 验证。 |
| Kimi 权限设置入口缺失 | 未建 | 待查（预计 settings UI + Kimi ACP 配置写入） | **待修复** | — | 需求：codeg 里调用 Kimi Code 无处设置权限。先在上游 issue 区搜是否已有反馈。 |

## 新补丁登记模板

```
| <一句话症状> | fix/<名> | <文件清单> | 待修复 | — | 根因假设；复现路径；计划上游化的理由 |
```
