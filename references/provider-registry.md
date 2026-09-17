# AI 服务商注册表

这些是接入时的初始候选值，不是永久不变的接口承诺。实现前应以对应服务商的官方 API 文档核对 Base URL、模型 ID、认证方式和模型列表能力。

| 服务商 | Base URL | Chat API | 默认 Key 环境变量 | 备注 |
|---|---|---|---|---|
| 硅基流动 | `https://api.siliconflow.cn/v1` | `/chat/completions` | `SILICONFLOW_API_KEY` | 可调用平台模型目录中的多家模型 |
| DeepSeek | `https://api.deepseek.com` | `/chat/completions` | `DEEPSEEK_API_KEY` | OpenAI 兼容接口 |
| Kimi | `https://api.moonshot.cn/v1` | `/chat/completions` | `KIMI_API_KEY` | OpenAI 兼容接口 |
| 智谱 GLM | `https://open.bigmodel.cn/api/paas/v4` | `/chat/completions` | `GLM_API_KEY` | 接入前核对官方文档和账户权限 |

## 配置模板

```env
SILICONFLOW_API_KEY=
DEEPSEEK_API_KEY=
KIMI_API_KEY=
GLM_API_KEY=

# 可选的 Endpoint 覆盖项
SILICONFLOW_ENDPOINT=
DEEPSEEK_ENDPOINT=
KIMI_ENDPOINT=
GLM_ENDPOINT=
```

不要把真实 Key 填入模板、示例、测试、截图或 Git。不同服务商的 Key、Endpoint 和模型目录必须分开处理。

## 模型发现的统一结果

模型发现或候选模型测试应归一化为以下结构：

```json
{
  "provider": "siliconflow",
  "models": [
    {
      "id": "Qwen/Qwen3-8B",
      "status": "verified",
      "reason": null
    },
    {
      "id": "some-model",
      "status": "permission_denied",
      "reason": "当前 API Key 无权访问该模型"
    }
  ]
}
```

允许的状态包括：`verified`、`discovered`、`permission_denied`、`invalid_model`、`network_error`、`stale`。
