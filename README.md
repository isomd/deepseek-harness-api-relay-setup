# DeepSeek Harness 怎么接入 API 中转站？自定义模型与协议配置教程

## 仓库附件

- [连接核对表](connection-checklist.md)
- [封面提示词](cover-prompt.md)

模板只含占位符或环境变量引用，请按下文合并到自己的配置，保留现有设置。资料核对日期为 2026-10-21；本仓库没有声称已经完成付费线路实测。

---

DeepSeek Harness 可以连接自定义模型 API。接中转站时，要把 Provider、接口协议、基础地址和模型 ID 配成一组，不能因为工具名带 DeepSeek，就把所有地址都填到内置 DeepSeek 配置里。

本文由 Orbis 项目方整理，以 [Orbis](https://orbiorbi.com) 的 OpenAI 兼容线路为例，使用 `dsh` 从终端启动 Web UI，再在模型设置里完成接入。这里没有把 Web UI 写成终端 TUI；第三方终端前端也不能直接套用同一套操作。

## 从官方入口启动

DeepSeek Harness 官方仓库的当前启动方式是先安装 Node.js，再运行：

```bash
npx @deepseek-ai/dsh web
```

首次使用时核对包名与 [官方仓库](https://github.com/deepseek-ai/deepseek-harness) 一致。命令默认启动本机 Web UI，地址通常是 `http://127.0.0.1:3080`。如果端口被调整，以终端输出为准。

在已有安装环境中，可以用 `dsh --version` 记录版本。该项目仍处于预览阶段，配置和插件可能出现不兼容变化，排查问题时应同时记录版本和启动的 profile。

## 选择自定义模型 API

在 Orbis [模型广场](https://orbiorbi.com/pricing) 选择支持 OpenAI Chat Completions 的文本模型线路，为它创建 Key。本文用 `deepseek-v4.1-flash` 说明写法，核对时它的多条公开线路提供 `openai` 协议。

到 DSH 的“设置 → 模型 → 添加模型提供商”，切换到“自定义模型 API”，按下面填写：

| 字段 | 本文示例 |
| --- | --- |
| Provider ID | `orbis-chat` |
| 显示名称 | `Orbis Chat` |
| 基础 URL | `https://api.orbiorbi.com/v1` |
| API 协议 | OpenAI Chat Completions |
| 凭据 | 这条线路的完整 API Key |
| 模型 ID | `deepseek-v4.1-flash` |

先从所选线路快速接入页确认地址；如果页面说明发生变化，以页面为准。模型展示名只用于阅读，请求要使用调用 ID。

Provider ID 用小写名称。当前官方说明中，该 ID 会被会话、默认模型和凭据引用，不是随时可改的显示名。想改 ID 应创建新 Provider，再处理旧项。

## 协议要与线路对应

DSH 的自定义模型界面列出了三种协议：

| 界面协议 | 适合什么接口 |
| --- | --- |
| OpenAI Chat Completions | 接受 Chat Completions 请求的线路 |
| OpenAI Responses | 接受 Responses 请求的线路 |
| Anthropic Messages | 接受 Claude Messages 请求的线路 |

本文只配置第一种。一个 Provider 使用一种协议；如果另有 GPT Responses 线路，应另建 Provider，配上该线路自己的 Key 和模型，不要把同一项来回改成不同协议。

“同一个 API 站提供这些协议”和“同一条线路全部支持”是两件事。Orbis 的线路由 Key 决定，不会因为你把协议选择框换成 Responses 就自动更换线路。

## 拉取不到模型，可以手动录入

在“模型目录”里使用“获取可用模型”，可以尝试读取网关模型列表。保存前核对本次表单中的地址、协议和 Key。

如果列表为空或拉取失败，先保留错误信息；已从接入页确认调用 ID 的，可以手动添加。模型列表接口与生成接口是不同的请求，列表失败不等于生成接口必然不可用。

当前官方文档还区分内置目录和自定义 Provider：内置 Provider 即使修改地址，模型列表仍可能来自安装目录。要查看网关真正提供的模型，应使用自定义 Provider。

上下文、最大输出、图片输入和推理等级，只按所选线路公开的限制填写。勾选“支持图片”不会让纯文本端点获得图片能力，客户端的能力声明也不是在线检测结果。

## 选择模型，跑一个文件整理任务

保存后，在模型选择器中选中 `Orbis Chat` 下的目标模型，新建会话。先发送“只回复 OK”，再到 Orbis 调用日志核对。

随后给一个小目录准备几份公开 Markdown 文件，把它选为工作区，并发送：

> 阅读当前目录的 Markdown 文件，为每份文件写一句摘要，并引用实际文件名。先把目录草稿发出来，不要改文件，也不要联网补充内容。

确认没有杜撰文件和内容后，再让它把结果保存为一个新文件 `INDEX.md`。检查文件确实存在，且没有覆盖原文。这样可以分别检查模型请求、文件读取和文件写入。

新会话默认值改变，不代表已有会话也切换了模型。调试时新建一个小会话，比较容易判断请求到底用了哪一项配置。

## 失败时留下可定位的信息

401 先核对凭据，404 核对基础路径和协议，模型不存在则回到线路接入页检查调用 ID。出现升级要求时，记录错误中指出的版本，不要反复改 Key。

如果对话正常而联网搜索失败，应单独检查搜索工具的服务配置。模型接口可用，不代表搜索服务也由这把 Key 提供。

准备反馈时保留版本、profile、时间、模型 ID、HTTP 状态和脱敏错误。不要公开凭据文件或含真实 Key 的配置。

## 参考资料

- [DeepSeek Harness 官方模型配置](https://deepseek-harness.github.io/deepseek-harness/guide/providers)
- [DeepSeek Harness 官方仓库与启动说明](https://github.com/deepseek-ai/deepseek-harness)
- [Orbis 快速开始](https://orbiorbi.com/docs/quickstart/)
- [Orbis：如何选择线路](https://orbiorbi.com/docs/choose-route/)

资料核对日期：2026-10-21。本文使用官方文档中的启动与配置方式，本轮没有实际提交付费生成请求，也未验证所有插件和模型组合。
