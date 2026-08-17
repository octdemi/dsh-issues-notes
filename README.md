# DeepSeek Harness 问题记录与处理方法

> 记录使用 [DeepSeek Harness](https://github.com/deepseek-ai/DeepSeek-Harness) 过程中遇到的问题及对应的处理方法（troubleshooting notes & workarounds）。

## 📋 问题索引

### 模型请求

- [001 · 模型请求失败重试次数太少，任务容易中断](issues/001-retry-policy.md) — 通过 `settings.yaml` 为 provider 配置 `retryPolicy`，加长重试次数与退避时间

### 插件开发

- [002 · 插件导致页面加载失败（HARNESS Failed to load plugins）](issues/002-plugin-load-failure.md) — 插件注入依赖导致 boot 图 activation 卡死，换 agent 修改的思路

---

## 📝 如何新增一条记录

1. 在 `issues/` 目录新建 `NNN-简短标题.md`（编号递增）
2. 记录格式参考现有条目：**现象 / 影响 / 解决过程 / 处理方法 / 验证**
3. 在下方索引表中追加一行

## 🔧 环境信息

- DeepSeek Harness: `0.1.0-rc.6`
- Profile: `web`（`~/.dsh/profiles/web/`）
- 配置目录: `~/.dsh/`