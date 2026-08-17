# 001 · 模型请求失败重试次数太少，任务容易中断

## 现象

使用过程中，请求模型偶尔失败（网络波动 / 限流 / 上游临时故障）。默认情况下 DSH 对失败请求只**重试 2 次**，间隔短（0.5s → 10s 指数退避），重试耗尽后任务直接报错停止。

但实际观察：**临时故障等一会儿就能恢复**——所以想加长重试次数、拉长重试间隔，让任务扛过短暂故障。

## 原因

DSH 的 LLM 请求重试由 `@deepseek-ai/dsh-llm-retry` 插件管理，默认策略（normal mode）：

| 配置项 | 默认值 |
|--------|--------|
| `maxRetries` | **2**（共尝试 3 次） |
| `initialDelayMs` | 500 |
| `maxDelayMs` | 10000 |
| `jitterRatio` | 0.1 |
| `retryableCodes` | `EMPTY_RESPONSE` / `RATE_LIMIT` / `SERVER` / `TIMEOUT` / `TRANSPORT` |

provider 未配置 `retryPolicy` 时使用上述默认值。

## 解决方法

在 `~/.dsh/settings.yaml` 中，给对应 provider（本机为 `llm-pi-ai.providers.eastbuy`）增加 `retryPolicy` 配置：

```yaml
llm-pi-ai:
  providers:
    eastbuy:
      displayName: ai-gateway
      apiKeyEnv: EASTBUY_API_KEY
      api: openai-completions
      baseURL: https://ai-gateway.eastbuy.com/v1
      retryPolicy:                          # ← 新增
        mode: normal
        maxRetries: 5                       # 最多重试 5 次（共尝试 6 次）
        retryableCodes:
          - EMPTY_RESPONSE
          - RATE_LIMIT
          - SERVER
          - TIMEOUT
          - TRANSPORT
        backoff:
          initialDelayMs: 2000              # 首次重试等 2s
          maxDelayMs: 30000                 # 退避上限 30s
          jitterRatio: 0.1                  # 10% 抖动，防雪崩
      models:
        - id: ...
```

### 两种模式说明

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| `normal` | 有限重试次数（`maxRetries`），可自定义退避和可重试错误码 | **推荐**：临时故障一般几次内恢复，不会无限烧 token |
| `always` | **无限重试**直到成功或取消，只可自定义退避 | 追求任务绝不因临时故障中断；但永久性错误（如 API Key 无效）也会一直重试 |

### 配置校验方法

用 DSH 自带的 `resolveRetryPolicy` 校验配置是否合法：

```bash
node --input-type=module -e "
import { resolveRetryPolicy } from '<DSH安装目录>/node_modules/@deepseek-ai/dsh-llm/lib/index.js';
const cfg = { mode: 'normal', maxRetries: 5, retryableCodes: ['EMPTY_RESPONSE','RATE_LIMIT','SERVER','TIMEOUT','TRANSPORT'], backoff: { initialDelayMs: 2000, maxDelayMs: 30000, jitterRatio: 0.1 } };
console.log(JSON.stringify(resolveRetryPolicy(cfg, 'test'), null, 2));
"
```

校验通过则输出解析后的策略对象；不合法会抛错并说明原因（如 `backoff.initialDelayMs must be positive`）。

## 验证

1. 修改 `settings.yaml` 后**重启 DSH Web 服务**（配置启动时加载）
2. 断网 / 制造临时故障测试：观察请求失败后自动重试（间隔约 2s / 4s / 8s / 16s / 30s），共尝试 6 次
3. ✅ 重试期间任务不中断，最终成功

> 注意：重试次数越多，最坏情况下报错等待越久（6 次约累计等待 60s+30s 下线）。若觉得太久可调回 `maxRetries: 3` 或 `maxDelayMs: 20000`。同时每次重试都重新计费输入 token。

## 相关文件

- 配置文件：`~/.dsh/settings.yaml`
- 源码位置：`node_modules/@deepseek-ai/dsh-llm/lib/index.js`（默认值 `DEFAULT_MAX_RETRIES = 2` 等）
- 插件文档：`node_modules/@deepseek-ai/dsh-llm-retry/README.zh.md`