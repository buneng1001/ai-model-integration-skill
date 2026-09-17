---
name: ai-model-integration-skill
description: >-
  Connect AI model providers to a project with consistent provider configuration, local API key handling,
  remembered model history, model discovery, connection tests, and reusable adapter code. Use when a user asks
  to integrate an AI/LLM provider into an application or wants the same integration pattern reused in another project.
---

# AI 模型接入

将多个 AI 服务商接入当前项目，并保持配置、密钥、模型发现、连接测试和调用适配器的一致结构。

## 工作边界

- 先检查项目语言、框架、依赖管理和现有 AI 接入代码，再决定落点。
- 服务商、Endpoint、模型 ID 和 API Key 必须隔离；不能把一家服务商的 Key 发给另一家。
- 默认 Key 从项目约定的本地配置读取；禁止写入源码、日志、报告、测试快照或提交记录。
- 临时 Key 默认只保存在当前进程或页面内存。用户明确勾选“记住临时 API Key”时，才写入本地 `.env.local` 或项目已采用的本地凭据文件，并确保该文件被 `.gitignore` 忽略。
- 真正发起连接测试前，说明会产生外部请求，用户已明确要求测试时才执行。

## 标准流程

1. 读取 [provider-registry.md](references/provider-registry.md)，根据用户要求选择服务商；对 Endpoint 和模型目录有时效性的内容，优先查官方文档。
2. 检查项目是否已经有配置层、HTTP 客户端、适配器、设置页面和测试；优先扩展已有 seam，不重写无关业务。
3. 建立或复用本地配置文件，至少为每个服务商提供独立的 API Key 环境变量和可选 Endpoint 覆盖项。
4. 合并 Key 来源，优先级固定为：当前临时 Key > 已记忆的本地 Key > 项目默认环境变量。
5. 优先读取“已验证模型记录”。它记录服务商、模型、验证时间、结果和配置来源；只要配置和 Key 未变化，就优先使用这些模型，不要求用户重复选择。
6. 只有在没有历史验证记录、用户主动刷新、Endpoint 变化或历史模型连接失败时，才执行模型发现流程：
   - 服务商支持模型列表接口时，读取结构化模型列表；
   - 不支持或列表不可用时，使用注册表中的候选模型逐个做低成本连接测试；
   - 将结果保存为结构化记录，而不是把服务商返回的原始文本直接放入下拉框。
7. UI 中将已验证模型、发现但未验证模型、连接失败模型和手动输入模型分开显示。手动模型应通过“其他模型 / 手动输入模型”进入，并在首次使用前测试。
8. 使用统一的连接测试请求验证 API Key、Endpoint 和模型，返回结构化状态：`ok`、`provider`、`model`、`key_source`、`checked_at`、`error_kind`、`retryable`、`message`。
9. 按项目语言生成最小适配器、配置示例、调用示例和测试；保留 Mock 或离线 seam，避免外部模型成为项目关键路径，除非用户明确要求。
10. 完成后检查 `.gitignore`、Git diff 和敏感信息扫描，再报告修改文件、已验证模型和仍需人工确认的服务商。

## 错误返回要求

所有失败路径都必须返回明确、可行动的错误提示。不能返回空字符串、通用的“请求失败”、只记录日志后继续、静默切换服务商或静默更换模型。

每个错误至少包含以下字段：

```json
{
  "ok": false,
  "error_kind": "authentication",
  "provider": "deepseek",
  "model": "deepseek-chat",
  "stage": "connection_test",
  "retryable": false,
  "message": "DeepSeek API Key 无效或当前账户无权访问该模型",
  "action": "检查 DEEPSEEK_API_KEY、账户余额和模型权限后重试",
  "details": "HTTP 401"
}
```

规则如下：

- `message` 面向用户，必须说明“哪里失败”和“可能原因”。
- `action` 面向用户，必须说明下一步怎么处理；没有可行操作时写明“请联系服务商或项目维护者”。
- `details` 只放脱敏后的状态码、错误类型或请求阶段，不能放 API Key、Authorization 头或完整敏感响应。
- 必须标记错误发生阶段：`configuration`、`key_resolution`、`model_discovery`、`connection_test`、`model_request`、`persistence` 或 `code_generation`。
- 能确定服务商和模型时必须返回它们；尚未确定时使用 `null`，不能猜测。
- 可重试错误返回 `retryable: true`，认证、权限、模型不存在、参数错误和配置错误返回 `retryable: false`。
- 连接测试失败时不能显示“当前可用”，也不能把失败模型加入“已验证模型”。
- 模型发现部分失败时，要分别列出成功、失败和未验证模型，不能用一个总的“探测失败”掩盖具体结果。
- 本地配置文件不存在、变量为空、格式错误、无写权限或保存失败，都必须显示具体文件路径、变量名或操作，并给出修复动作。
- 清除临时 Key 或记忆 Key 失败时，必须明确说明“内存是否已清除”和“本地文件是否仍保留”。
- 任何降级到 Mock、默认模型或备用服务商，都必须先说明降级原因并获得用户明确允许；不能静默降级。

## 模型记录规则

推荐使用项目内的非敏感 JSON 文件或数据库记录模型元数据，例如：

```json
{
  "provider": "deepseek",
  "model": "deepseek-chat",
  "endpoint": "https://api.deepseek.com",
  "status": "verified",
  "key_source": "local-default",
  "checked_at": "2026-09-17T00:00:00Z"
}
```

记录中不得出现 API Key 原值、Authorization 头或完整请求内容。Key 变化、Endpoint 变化或连续认证失败时，应将历史记录标记为 `stale`，再重新探测。

## 交互约定

推荐文案和控件：

- `AI 模型接入`
- `服务商`
- `API Key 来源：本地默认 / 临时 Key`
- `记住这个临时 API Key`
- `已验证模型`
- `探测并保存模型`
- `其他模型 / 手动输入`
- `测试 AI 连接`
- `使用本地默认配置测试`
- `清除临时 API Key`

“探测并保存模型”必须展示结构化结果和可操作动作：用户可以勾选模型加入已验证列表、查看失败原因、重试单个模型，或把模型复制到手动输入框；不能只展示一段不可操作的文本。

## 安全和失败处理

- 401/403：停止重试，提示 Key、账户权限或模型权限。
- 400：提示模型 ID 或请求参数问题。
- 429、408、5xx、明确的临时网络超时：执行项目约定的有限重试，并展示可重试状态。
- 其他网络错误：保留可诊断错误，但不得打印 Key。
- DNS、TLS、代理、连接超时和响应解析失败必须区分提示，不能全部归类为“网络错误”。
- 服务商返回非 JSON、缺少 `choices`、缺少模型 ID 或响应结构不符合适配器约定时，提示具体的响应结构问题。
- 不能把模型列表中的“存在”描述成“当前 Key 可用”；只有连接测试成功才标记为 `verified`。
- 用户要求清除临时 Key 时，清除内存状态；若之前选择了记忆，同时删除对应本地配置项，并明确说明删除范围。
