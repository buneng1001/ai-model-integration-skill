# 模型发现与网络连接规范

## 1. 模型状态

不要使用一个“可用”字段覆盖所有阶段。至少区分：

```text
discovered          服务商返回了模型 ID
verified            当前 Endpoint、Key、网络方式和模型通过最小连接测试
workflow_validated  在代表性业务输入和输出约束下通过业务冒烟测试
invalid_model       模型不存在或当前 Key 无权访问
stale               验证所依据的配置快照已经变化
```

模型发现成功不代表模型可以使用；连接测试成功也不代表长文档或复杂业务工作流一定能完成。

## 2. 发现模型与使用模型

前端建议提供两个独立动作：

- `发现模型`：请求服务商模型目录，保存完整模型 ID，支持滚动、搜索、复制和查看状态。
- `使用已发现模型`：将用户选中的发现模型带入当前配置快照，执行最小连接测试；测试通过后才允许作为当前模型或项目默认模型。

刷新成功后，以服务商真实返回的模型 ID替换预设候选，只做精确字符串去重。不要因为名称相似就合并模型，也不要把预设模型显示为已发现或已验证。

模型列表较长时，不能只展示前三项。下拉框或列表需要可滚动、可搜索、可显示完整模型 ID，并且不能因为列表展开而脱离对应控件。

## 3. 配置快照与失效

一次发现、测试或业务运行都绑定以下快照：

```json
{
  "provider": "deepseek",
  "endpoint": "https://api.deepseek.com",
  "model": "deepseek-flash",
  "key_source": "local-default",
  "key_fingerprint": "redacted",
  "network_mode": "auto",
  "prompt_version": "integration.v2"
}
```

provider、endpoint、model、Key 来源或指纹、network mode、Prompt/Skill 版本任一变化，都要把旧 `verified` 标记为 `stale`，并重新允许发现或测试。不得静默带出另一家服务商的模型。

## 4. 网络模式

提供以下可配置模式：

- `auto`：按有限且可观察的顺序尝试系统代理和直连；记录实际采用的路径。
- `system_proxy`：明确使用系统代理或项目配置的代理。
- `direct`：请求客户端忽略系统代理配置，尝试直连。

“直连”只控制应用 HTTP 客户端，不保证绕过 VPN、公司网关、操作系统路由或安全软件。UI 应明确说明这一边界。

网络模式改变后：

- 允许用户保留 Key、Endpoint 和模型配置；
- 使旧验证状态失效；
- 允许重新发现和重新测试；
- 记录 DNS、TLS、代理连接、直连、超时和服务商响应等诊断；
- 不得静默切换供应商、Endpoint 或模型。

## 5. 分层超时

不要用一个统一超时覆盖所有请求。至少区分：

- 连接测试网络等待；
- 首个响应等待；
- 流式响应空闲时间；
- 单次模型请求总时限；
- 整个业务工作流预算。

这些值必须可配置，并根据模型能力、输入大小、服务商限制和成本调整。错误信息要指出具体哪一层超时。

## 6. 错误提示

错误至少应区分：`dns`、`tls`、`proxy`、`connection`、`auth`、`permission`、`invalid_model`、`rate_limit`、`provider_5xx`、`response_start_timeout`、`stream_timeout`、`total_timeout` 和 `output_truncated`。

每个错误应包含用户可读原因、下一步动作、是否可重试、服务商、模型和脱敏诊断。不能只显示“网络错误”或“请求失败”。
