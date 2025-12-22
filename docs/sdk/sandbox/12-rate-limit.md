# 频率限制
<subtitle>了解平台 API 及沙箱操作的访问阈值，确保您的应用平稳运行。</subtitle>

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的秘钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

为了保障服务的稳定性与公平共享，UCloud Sandbox 平台对各类型的 API 调用及资源使用设置了频率限制 (Rate Limits)。

---

## 限制维度

### 1. 生命周期与管理 API
**限制阈值：每 30 秒 20,000 次请求**

适用于沙箱的创建、关停、状态更新、列表查询以及模板管理等管理类操作。

### 2. 沙箱内操作与私有请求
**限制阈值：每 60 秒 40,000 次请求**

适用于在运行中的沙箱内执行的具体指令，例如 `run_code`、`commands.run`、文件读写等。同时也包含通过公共 URL 访问沙箱内自定义端口的流量。

### 3. 并发沙箱数量
**限制阈值：视定价方案而定**

指您的团队在同一时刻允许同时开启（处于 Running 或 Paused 状态）的沙箱总数。详情请咨询技术支持。

### 4. 新建速率
控制短时间内创建新沙箱实例的最大速度，防止瞬时峰值过载。

---

## 达到限制时的表现

当您的调用超过设定的阈值时，平台会采取以下保护措施：

*   **HTTP 协议**：直接调用 REST API 或沙箱端口时，将返回 `429 Too Many Requests`。
*   **Python SDK**：方法调用将抛出 `RateLimitException` 异常。

## 最佳实践

!> **如何应对限制？**
1.  **指数退避**：在捕获到 `RateLimitException` 时，建议采用指数退避算法（Exponential Backoff）重试。
2.  **及时销毁**：不再使用的沙箱请务必显式调用 `.kill()`，以释放并发配额。
3.  **批量合并**：尽量合并文件读写或相关命令执行，减少短时间内频繁的 API 往返。