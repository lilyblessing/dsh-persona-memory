# Changelog

本仓库为 [Quophic/dsh-persona-memory](https://github.com/Quophic/dsh-persona-memory) 的个人 fork
（[lilyblessing/dsh-persona-memory](https://github.com/lilyblessing/dsh-persona-memory)）。
除上游功能外，自 0.1.20 起包含如下稳定性 / 安全修复。格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)。

## [0.1.21] - 2026-09-12（本 fork 独有）

- **管理页配置回显修复（lib/admin.js）**：`/api/persona-memory/status` 返回的 `config`
  此前只把 6 个派生键（dir / vectorEnabled / embeddingProvider / embeddingModel /
  embeddingRemoteHost / embeddingCacheDir）盖在 `CFG_DEFAULTS` 上，**已解析的 cfg 从未合并**。
  后果一（显示错值）：设置页「插件配置卡」对**除这 6 键外的每个字段**都显示默认值——客户端
  `client/client.js:187-188` 是 `if (!(f.key in s.config)) continue; const v = s.config[f.key];`。
  实机症状：页面显示 `memoryCharLimit: 5000` 而运行时实际按 8000 执行
  （由 `stores[].usagePct` 反推 + `memory` 工具末行 `Usage: 51% (4089 chars / limit 8000)` 双重确证）。
  后果二（**更严重**）：配置卡保存时把整个表单回贴（`client/client.js:255` `api('/configSave', cfgForm)`），
  因此错误的回显会被**写回 profile patch**，等于把 5000 固化成真值。
  后果三：`vectorIndexDir`、`embeddingBaseUrl` 既不在 `CFG_DEFAULTS` 也不在派生块中，
  **自始至终没有渲染过**。
  修复：`currentConfig` 从 `makeAdminRoutes` 闭包提升为**模块级导出纯函数** `currentConfig(cfg)`，
  改为 **`CFG_SCHEMA` 驱动**：对除 `embeddingApiKey` 外的每个 schema 键取
  `cfg[key] !== undefined ? cfg[key] : CFG_DEFAULTS[key]`，仍为 undefined 则回落 `''`。
  优先级为 **已解析 cfg → 默认值 → 空串**，并统一了旧派生块与默认值不一致的语义
  （旧版会把 `vectorEnabled: undefined` 显示成 `true`、`embeddingProvider: undefined` 显示成 `'local'`）。
  `embeddingApiKey` 显式跳过，密钥永不发给浏览器。
- **回归测试（test/smoke.mjs）**：新增 10 条断言，覆盖「非派生键字段正确回显」
  （含 `autoBackupMin` 这一原 bug 的核心反例）、未设置键回落默认、
  「除密钥外每个 schema 字段都必须出现在回显里」（与客户端 `!(f.key in s.config)` 约束对齐，
  并专门锁定 `vectorIndexDir`/`embeddingBaseUrl` 两个曾经不可见的键）、
  `vectorEnabled: false` 不再被派生块反转、以及 `embeddingApiKey` 不泄露。

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
