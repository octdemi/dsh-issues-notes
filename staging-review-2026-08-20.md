# 暂存区 11 条候选说明（2026-08-20 补录）

> 这 11 条全部来自**同一次排查任务**：调查 DeepSeek API 429 报错 `bad_response_status_code` 的来源与含义（会话 `session-6eeb63f7`，约 8 月 17 日）。它们不是 11 个独立脚本，而是**一条调研链的连续步骤**，每条只是当时查资料用的临时代码。

## 逐条说明

| # | 暂存 ID | 实际做了什么 | 可复用性 |
|---|---------|-------------|---------|
| 1 | code-const-r-await-tools-web-search-query-deepseek-api-429-b | 用 `tools.web_search` 搜"deepseek API 429 bad_response_status_code 报错" | 🔴 一次性调研 |
| 2 | code-const-r-await-tools-web-search-query-openrouter-bad-res | 搜 "OpenRouter bad_response_status_code 429 DeepSeek error" | 🔴 一次性调研 |
| 3 | code-const-r-await-tools-web-search-query-deepseek-api-429-r | 搜 "deepseek api 429 rate limit 官方 说明 限流 额度" | 🔴 一次性调研 |
| 4 | code-roocode-issue-nconst-execsync-await-import-no | 用 `curl` 抓取官方限流文档 + RooCode issue 页面 HTML | 🟡 可复用：`execSync` + curl 抓网页去掉 HTML 标签的小工具 |
| 5 | code-const-execsync-await-import-node-child-process-nfunctio | 抓取并解析官方"限速与隔离"文档正文（并发限速说明） | 🟡 同上，网页正文提取技巧 |
| 6 | code-github-bad-response-status-code-nconst-execs | 在 GitHub 上搜 `bad_response_status_code` 源码来源（未登录被拒） | 🔴 一次性调研（而且失败了） |
| 7 | code-const-r-await-tools-web-search-query-bad-response-statu | 搜该错误串在 chatgpt/claude 等 API 的含义 | 🔴 一次性调研 |
| 8 | code-dmxapi-theneuralbase-rate-limit-errors-nconst-e | 抓 dmxapi 报错详情页 + theneuralbase 限流说明 | 🔴 一次性调研 |
| 9 | code-const-r-await-tools-web-search-query-openai-error-bad-r | 搜 `openai_error` + `bad_response_status_code` + deepseek 429 | 🔴 一次性调研 |
| 10 | code-const-r-await-tools-web-search-query-openai-error-error | 搜该错误格式的来历与 DeepSeek 429 修复方案 | 🔴 一次性调研 |
| 11 | code-const-r-await-tools-web-search-query-openrouter-error-b | 验证 OpenRouter 错误格式与 DeepSeek 原生 429 格式差异 | 🔴 一次性调研 |

## 结论与建议

- **全部删除**：这 11 条都是当时查资料的一次性 `web_search` / `curl` 胶水代码，没有沉淀价值，未来不会再复用
- 唯一的半成品价值是 **#4 / #5 的"curl 抓网页 + 去标签转正文"小技巧**，但实现很粗糙（正则删标签），不值得留
- 建议：侧边栏面板里全部点"删除"，暂存区恢复为空

## 本次调研的最终结论（保留价值，不是脚本）

> `bad_response_status_code` 是 **OpenRouter 网关**对上游返回非 2xx 时包装的错误码格式；经过 eastbuy 代理转发给 DSH 时，API 返回 429（限流）。DeepSeek 官方自身的错误码格式是 `{"error":{"message":..., "type":"..."}}`，不含 `bad_response_status_code` 这个字段。即：429 是**上游限流**，错误字段格式是**网关层**的包装。实际应对 = 等一会儿/降频（与本次配置的 retryPolicy 呼应）。