# Changelog

本仓库为 [Quophic/dsh-persona-memory](https://github.com/Quophic/dsh-persona-memory) 的个人 fork
（[lilyblessing/dsh-persona-memory](https://github.com/lilyblessing/dsh-persona-memory)）。
除上游功能外，自 0.1.20 起包含如下稳定性 / 安全修复。格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)。

## [0.1.20] - 2026-08-18（本 fork 独有）

聚合提交 `f28009c`「fix(admin,index,store): 3 处稳定性/安全修复」，相对上游 0.1.19：

- **路由生命周期修复（index.js）**：`webServer.register` 返回 disposer 且同 path 重复注册抛错，
  此前 disposer 被丢弃 → 插件重载/卸载会遗留孤儿路由；现接线到 `ctx.effect`，随 ctx 生命周期自动清理。
- **WebUI 写入统一走单文件锁 + 外部指纹预检（lib/memory-store.js 新增 `mutateDecoded`）**：
  「记忆管理」页的增删改改为与 `memory` 工具**共用同一把每文件锁** + sha256 指纹预检，
  不再与模型工具调用并发互覆盖，也不覆盖 Pi / 手动编辑（原为手工 read-modify-write）。
- **项目记忆编辑同样过锁 + 容量检查（lib/admin.js）**：`mutateProject` 改经项目 store（`mutateDecoded`），
  保留非法项目名校验，并新增 `projectCharLimit` 超限拒绝。
- **standing 配套调整（lib/standing.js）**。

## [0.1.19] - 上游版本（fork 起点）

上游 [Quophic/dsh-persona-memory](https://github.com/Quophic/dsh-persona-memory) 的 0.1.19（`f37e3c7`）：
`memory-store` rewrite 格式检测误判裸 `§` 导致全部记忆压成单条 — 改用真实 `\n§\n` 分隔符检测 +
修复 MEMORY.md 数据 + 回归测试。本 fork 在其之上推出 0.1.20。
