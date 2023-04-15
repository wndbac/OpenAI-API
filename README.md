# OpenAI API 代理

由于 OpenAI 及 GFW 的双重限制，国内开发者无法访问 OpenAI 的 API，现提供代理服务地址供开发者**免费**使用。

✅ 代理地址：[https://openai.wndbac.cn](https://openai.wndbac.cn)，直接替换官方的 [https://api.openai.com](https://api.openai.com)，支持官方所有v1接口。

⚠️ 本代理服务通过Cloudflare反向代理访问OpenAI的接口，只做代理中转，不会保存任何数据！

🚨 请勿使用魔法上网的方式用你的 ApiKey 去调用 api.openai.com的接口，否则大概率会被 OpenAI 封号！

## 1、🔔 获取ApiKey

注册 [OpenAI](https://platform.openai.com/account/api-keys) 账号，获取你的 [ApiKey](https://platform.openai.com/account/api-keys)，过程略。

## 2、💬 聊天接口

#### ⚠️ 不再推荐使用本接口，后面将废弃。

> POST [https://openai.wndbac.cn/pro/chat/completions](https://openai.wndbac.cn/pro/chat/completions)

请求参数

| 参数名 | 类型 | 长度 | 必须 | 备注 |
| :----- | :----- | :----- | :----- | :----- |
| apiKey | String | 64 | 是 | OpenAI 的 ApiKey |
| sessionId | String | 64 | 是 | 会话ID，关联上下文，推荐使用UUID作为sessionId |
| content | String | 1000 | 是 | 发送的内容 |

请求示例（**Content-Type = application/json**）

```JSON
{
    "apiKey": "<your_openai_api_key>",
    "sessionId": "8d1cb9b0-d535-43a8-b738-4f61b1608579",
    "content": "你是谁？"
}
```

响应示例

```JSON
{
    "code": 200,
    "message": "执行成功",
    "data": "我是一名人工智能助手，用于协助回答问题和提供建议。您可以通过与我进行对话来获取您需要的信息或帮助。"
}
```

本接口使用的是 `gpt-3.5-turbo` 模型，支持通过上下文内容进行连续对话。

本接口对官方的接口进行了封装，开发者只需为每个 `ApiKey` 下的每个会话分配一个独立的 `sessionId` 即可实现连续对话。

推荐使用 uuid 作为 `sessionId` 以保证全局唯一 ，否则对话可能会"串线"。

对话中的上下文信息有效期为30分钟，过期后再次发送消息无法关联上下文。

## 3、✅ 推荐方式

推荐大家直接根据 OpenAI 官方的接口文档开发你的程序。

你只需要将官方接口域名替换为 [https://openai.wndbac.cn](https://openai.wndbac.cn)  即可在国内网络环境下直接调用，支持SSE。

例如将：

https://**api.openai.com**/v1/chat/completions

改成

https://**openai.wndbac.cn**/v1/chat/completions

接口URL、请求参数、响应报文等全部与OpenAI官方一致，请直接参考 [OpenAI 官方文档](https://platform.openai.com/docs/introduction) 。

**测试命令**

```
# 测试聊天
curl https://api.openai-proxy.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your_openai_api_key>" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'

# 响应结果
  {
  "id": "chatcmpl-21lvNzPaxlsQJh0BEIb9DqoO0pZUY",
  "object": "chat.completion",
  "created": 1680656905,
  "model": "gpt-3.5-turbo-0301",
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 10,
    "total_tokens": 20
  },
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Hello there! How can I assist you today?"
      },
      "finish_reason": "stop",
      "index": 0
    }
  ]
}
```

```
# 测试生成图片
curl https://api.openai-proxy.com/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your_openai_api_key>" \
  -d '{
    "prompt": "A bikini girl",
    "n": 2,
    "size": "512x512"
  }'

# 响应结果
  {
  "created": 1680705608,
  "data": [
    {
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-xxxxxxx"
    },
    {
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-xxxxxxx"
    }
  ]
}
```

**如何通过OpenAI官方的接口实现连续对话？**

有很多用户问如何实现连续聊天，官方文档其实写的很清楚 [https://platform.openai.com/docs/guides/chat/introduction](https://platform.openai.com/docs/guides/chat/introduction)

> Chat models take a series of messages as input, and return a model-generated message as output.
> 
> 聊天模型将一系列消息作为输入，并返回模型生成的消息作为输出。

下面是官方Python语言实现连续聊天的代码示例，主要看这个 messges 数组：

```
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai

openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}
    ]
)
```

role为user是用户发送的内容，role为assistant是AI回答的内容。

OpenAI本身是没有记忆的，如果你不告诉他你之前说了什么以及他之前回答了什么，那么他只会根据你最近一次发送的内容进行回答。

所以，要想实现“连续对话”，每次发送消息时，你需要将你之前发送的内容(**user**)以及OpenAI之前返回的内容(**assistant**)，再结合你本次想发送的内容(**user**) 按 **时序** 组合成一个 messages[] 数组，然后再将这个数组发送给OpenAI就行了，就是这么简单。

有一点需要注意，这样虽然可以实现“连续对话”，但势必造成每次发送的消息内容会非常多，而OpenAI是按字数计费的，所以你需要根据你口袋里的钞票去衡量每次应该携带几条聊天记录比较合适。

## 4、⭐️ 第三方应用

如果你用的是第三方开发者开发的基于 OpenAI 的应用，你也可以直接把 [https://openai.wndbac.cn](https://openai.wndbac.cn) 设为其代理接口地址，这样就直接可以在国内网络环境下使用该应用。比如下列几个应用：

| 名称 | GitHub地址 | Stars |
| :----- | :----- | :----- |
| OpenAI-Translator | https://github.com/yetone/openai-translator | ![](https://img.shields.io/github/stars/yetone/openai-translator.svg) |
| ChatGPT-Next-Web | https://github.com/Yidadaa/ChatGPT-Next-Web | ![](https://img.shields.io/github/stars/Yidadaa/ChatGPT-Next-Web.svg) |
| ChatGPT-Web | https://github.com/Chanzhaoyu/chatgpt-web |  ![](https://img.shields.io/github/stars/Chanzhaoyu/chatgpt-web.svg) |

## 5、💰 查询余额

接口地址 (**GET请求**)

> GET [https://api.openai-proxy.com/pro/balance?apiKey=sk-xxxxxxxxxxxxxx](https://api.openai-proxy.com/pro/balance?apiKey=sk-xxxxxxxxxxxxxx)

请求参数

| 参数名 | 类型 | 长度 | 必须 | 备注 |
| :----- | :----- | :----- | :----- | :----- |
| apiKey | String | 64 | 是 | OpenAI 的 ApiKey |

相应示例

```
{
    "code": 200,
    "message": "执行成功",
    "data": {
        "total": 18.00,
        "balance": 17.92,
        "used": 0.08
    }
}
```

原OpenAI官方后台查询余额的接口由于被用户滥用，官方给撤销了，现有一个折中的方式去计算账户余额。逻辑是先得到OpenAI给你账户授权的总金额，然后减去最近90天你账户消耗的金额，得到的 balance 即为账户可用余额。如果你的账号已使用超过90天，此计算方式会存在误差，如果想知道准确的数据，请登录OpenAI官网查看，目前别无他法。

---

Email：towindback@qq.com

ChatGPT：https://chat.wndbac.cn

更新时间：2023-04-15
