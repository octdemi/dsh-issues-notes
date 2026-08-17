# 002 · 插件导致页面加载失败（HARNESS Failed to load plugins）

## 现象

访问 `http://127.0.0.1:3080/` 页面失败，页面报错：

```
HARNESS
Failed to load plugins
web boot: 1 entry did not activate dsh-script-library: pending (waiting for service: remote.scriptLibrary)
```

即：插件 `dsh-script-library` 的激活一直处于 pending 状态，等待服务 `remote.scriptLibrary`，导致整个 web boot 失败、页面打不开。

## 影响

- 整个 Web GUI 无法加载（白屏/错误页）
- 所有依赖该页面才能使用的功能不可用
- 通常发生在**修改插件代码或注入依赖后**重启服务时

## 原因

（本条目按需记录，不深入展开技术细节。简要背景：插件在 `apply()` 内通过 `ctx.remote.$mount` 自挂载命名空间，同时又在 Cordis 注入列表中声明了自己才能提供的服务依赖，导致激活互相等待——**注入依赖与服务创建顺序冲突**。）

## 处理方法

### 修改思路（核心）

**换一个更熟悉 DeepSeek Harness 的 agent 来协助修改**：

1. **新建一个 codex / trae 等 agent 会话**，把报错信息完整贴给它（`Failed to load plugins` + `pending (waiting for service: remote.scriptLibrary)`）
2. **同时提供必要的上下文**，让 agent 能定位问题：
   - 插件目录位置（如 `~/.dsh/profiles/node_modules/dsh-script-library/`）
   - 插件注册文件（`~/.dsh/profiles/web/cordis.patch.yml`）
   - 报错栈 / 插件代码结构
3. **要求 agent 检查注入依赖与服务创建顺序**：
   - 是否在 inject 中声明了插件自身才提供的服务（死锁）
   - 是否应该使用直接 RPC / 事件等替代方式，而不是依赖 `ctx.remote`
4. 让 agent 修改后，**重启 DSH Web 服务验证页面恢复**

### 为什么换 agent

- 担心**别的模型不了解 DeepSeek Harness** 的插件体系（Cordis 生命周期、boot 图激活机制、依赖注入规则）
- codex / trae 等 agent 对代码库理解更深，能顺着报错定位到 activation 卡住的根因
- 也方便在工作区直接编辑插件源码并复测

### 临时恢复手段

在等待修复期间，可临时**卸载插件**让页面恢复：

```bash
# 注释掉 cordis.patch.yml 中的 insert 条目
# 然后重启 DSH Web 服务
```

```yaml
# - insert:
#     - id: script-library
#       name: 'dsh-script-library'
#       config: {}
```

## 验证

1. 修改后重启 DSH Web 服务
2. 访问 `http://127.0.0.1:3080/`，页面正常加载
3. 浏览器控制台无 `Failed to load plugins` 报错
4. 插件功能正常（如侧边栏出现脚本库入口）

## 相关文件

- 插件源码：`~/.dsh/profiles/node_modules/dsh-script-library/`
- 插件注册：`~/.dsh/profiles/web/cordis.patch.yml`
- 页面入口：`http://127.0.0.1:3080/`