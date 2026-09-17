# ai-model-integration-skill

用于把 DeepSeek、Kimi、智谱 GLM、硅基流动等 AI 服务商按统一方式接入项目。

Skill 显示名称：**AI 模型接入**

能力范围：

- 服务商和 Endpoint 注册
- 本地默认 API Key 与临时 API Key
- 可选的本地 Key 记忆
- 已验证模型历史记录
- 模型发现和候选模型测试
- 统一连接测试结果
- 项目适配器、调用示例和测试生成

使用前请阅读 [SKILL.md](SKILL.md)。真实 API Key 只应保存在本机配置中。

## 本地 API Key

在使用本 skill 的项目根目录创建本地 Key 文件：

```text
./.env.local
```

可以参考 `.env.example` 创建该文件。`.env.local` 已被 `.gitignore` 排除，不会提交到 GitHub。

## 调用方式

### 自动触发

当用户在项目中提出以下类型的需求时，skill 可以自动触发：

- “把 DeepSeek 接入这个项目。”
- “给这个项目增加 Kimi、GLM 或硅基流动支持。”
- “统一几个 AI 服务商的 API Key 和模型配置。”
- “检查这个 API Key 能使用哪些模型，并接入可用模型。”
- “给现有 AI 接入增加临时 Key、模型探测和连接测试。”

自动触发后，skill 会先检查项目结构和已有接入代码，再决定配置文件、适配器、测试和界面的修改位置。

### 显式调用

也可以直接使用：

```text
$ai-model-integration-skill
```

例如：

```text
$ai-model-integration-skill
请把 DeepSeek 和 Kimi 接入当前 FastAPI 项目，读取本地 .env.local，探测当前 Key 可用的模型，生成连接测试和单元测试。
```

### 不会自动触发的情况

以下需求通常不会触发本 skill：

- 只询问某个模型的产品介绍或价格。
- 只要求普通的聊天调用示例，不要求接入项目。
- 只修改与 AI 无关的页面、数据库或业务逻辑。
- 只做模型效果评测，不涉及 API 配置和项目连接。

如果用户明确要求接入 API，即使没有点名具体服务商，也应先询问或识别项目需要的服务商，再开始配置。
