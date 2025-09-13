# cursor-api

* 当前版本已稳定，若发现响应出现缺字漏字，与本程序无关。
* 若发现首字慢，与本程序无关。
* 若发现响应出现乱码，也与本程序无关。
* 属于官方的问题，请不要像作者反馈。
* 本程序拥有堪比客户端原本的速度，甚至可能更快。
* 本程序的性能是非常厉害的。
* 根据本项目开源协议，Fork的项目不能以作者的名义进行任何形式的宣传、推广或声明。
* 更新的时间跨度达5月有余了，求赞助，项目不收费，不定制。
* 推荐自部署，[官方网站](https://cc.wisdgod.com) 仅用于作者测试，不保证稳定性。

## 获取key

1. 访问 [www.cursor.com](https://www.cursor.com) 并完成注册登录
2. 在浏览器中打开开发者工具（F12）
3. 在 Application-Cookies 中查找名为 `WorkosCursorSessionToken` 的条目，并复制其第三个字段。请注意，%3A%3A 是 :: 的 URL 编码形式，cookie 的值使用冒号 (:) 进行分隔。

## 配置说明

### 环境变量

* `PORT`: 服务器端口号（默认：3000）
* `AUTH_TOKEN`: 认证令牌（必须，用于API认证）
* `ROUTE_PREFIX`: 路由前缀（可选）

更多请查看 `/env-example`

### Token文件格式（已弃用）

`.tokens` 文件：每行为token和checksum的对应关系：

```
# 这里的#表示这行在下次读取要删除
token1,checksum1
token2,checksum2
```

该文件可以被自动管理，但用户仅可在确认自己拥有修改能力时修改，一般仅有以下情况需要手动修改：

* 需要删除某个 token
* 需要使用已有 checksum 来对应某一个 token

### 模型列表

写死了，后续也不会会支持自定义模型列表，因为本身就支持动态更新，详见[更新模型列表说明](#更新模型列表说明)

```
claude-4-sonnet
claude-4-sonnet-thinking
claude-4-opus-thinking
claude-4-opus
default
claude-3.5-sonnet
o3
gemini-2.5-pro-preview-05-06
gemini-2.5-flash-preview-04-17
gpt-4.1
claude-3.7-sonnet
claude-3.7-sonnet-thinking
cursor-small
claude-3.5-haiku
gemini-2.5-pro-exp-03-25
gpt-4o
o4-mini
deepseek-r1
deepseek-v3.1
grok-3-beta
grok-3-mini
```

支持图像(default始终支持)：
```
claude-4-sonnet
claude-4-sonnet-thinking
claude-4-opus-thinking
claude-4-opus
claude-3.5-sonnet
o3
gemini-2.5-pro-preview-05-06
gemini-2.5-flash-preview-04-17
gpt-4.1
claude-3.7-sonnet
claude-3.7-sonnet-thinking
claude-3.5-haiku
gemini-2.5-pro-exp-03-25
gpt-4o
o4-mini
```

支持思考：
```
claude-4-sonnet-thinking
claude-4-opus-thinking
o3
gemini-2.5-pro-preview-05-06
gemini-2.5-flash-preview-04-17
claude-3.7-sonnet-thinking
gemini-2.5-pro-exp-03-25
o4-mini
deepseek-r1
```

支持Max与非Max：
```
claude-4-sonnet
claude-4-sonnet-thinking
claude-3.5-sonnet
gemini-2.5-pro-preview-05-06
gpt-4.1
claude-3.7-sonnet
claude-3.7-sonnet-thinking
gemini-2.5-pro-exp-03-25
o4-mini
grok-3-beta
```

Max Only：
```
claude-4-opus-thinking
claude-4-opus
o3
```

非Max Only：
```
default
gemini-2.5-flash-preview-04-17
cursor-small
claude-3.5-haiku
gpt-4o
deepseek-r1
deepseek-v3.1
grok-3-mini
```

## 接口说明

### 基础对话

* 接口地址: `/v1/chat/completions`
* 请求方法: POST
* 认证方式: Bearer Token
  1. 使用环境变量 `AUTH_TOKEN` 进行认证
  2. ~~使用 `.token` 文件中的令牌列表进行轮询认证~~ 在v0.1.3的rc版本更新中移除`.token`文件
  3. ~~自v0.1.3-rc.3起支持直接使用 token,checksum 进行认证，但未提供配置关闭~~ v0.3.0起不再支持
  4. 使用 `/build-key` 构建的动态密钥认证
  5. 使用 `/config` 设置的共享Token进行认证 (关联：环境变量`SHARED_TOKEN`)
  6. 日志中的缓存 token key 的两种表示方式认证 (`/build-key` 同时也会给出这两种格式作为动态密钥的别名，该数字key本质为一个192位的整数)

#### 请求格式

```json
{
  "model": string,
  "messages": [
    {
      "role": "system" | "user" | "assistant", // 也可以是 "developer" | "human" | "ai"
      "content": string | [
        {
          "type": "text" | "image_url",
          "text": string,
          "image_url": {
            "url": string
          }
        }
      ]
    }
  ],
  "stream": boolean,
  "stream_options": {
    "include_usage": boolean
  }
}
```

#### 响应格式

如果 `stream` 为 `false`:

```json
{
  "id": string,
  "object": "chat.completion",
  "created": number,
  "model": string,
  "choices": [
    {
      "index": number,
      "message": {
        "role": "assistant",
        "content": string
      },
      "finish_reason": "stop" | "length"
    }
  ],
  "usage": {
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0
  }
}
```

如果 `stream` 为 `true`:

```
data: {"id":string,"object":"chat.completion.chunk","created":number,"model":string,"choices":[{"index":number,"delta":{"role":"assistant","content":string},"finish_reason":null}]}

data: {"id":string,"object":"chat.completion.chunk","created":number,"model":string,"choices":[{"index":number,"delta":{"content":string},"finish_reason":null}]}

data: {"id":string,"object":"chat.completion.chunk","created":number,"model":string,"choices":[{"index":number,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

### 获取模型列表

* 接口地址: `/v1/models`
* 请求方法: GET
* 认证方式: Bearer Token

#### 查询参数

可选的 JSON 请求体用于作为请求模型列表的参数：

```json
{
  "is_nightly": boolean,                    // 是否包含 nightly 版本模型，默认 false
  "include_long_context_models": boolean,   // 是否包含长上下文模型，默认 false  
  "exclude_max_named_models": boolean,      // 是否排除 max 命名的模型，默认 false
  "additional_model_names": [string]        // 额外包含的模型名称列表，默认空数组
}
```

**注意**: 认证可选，查询参数可选且认证时生效，未提供时使用默认值。

#### 响应格式

```json
{
  "object": "list",
  "data": [
    {
      "id": string,
      "display_name": string,
      "created": number,
      "created_at": string,
      "object": "model",
      "type": "model", 
      "owned_by": string,
      "supports_thinking": boolean,
      "supports_images": boolean,
      "supports_max_mode": boolean,
      "supports_non_max_mode": boolean
    }
  ]
}
```

#### 更新模型列表说明

每次携带Token时都会拉取最新的模型列表，与上次更新需距离至少30分钟。`additional_model_names` 可以用添加额外模型。

### Token管理接口

#### 简易Token信息管理页面

* 接口地址: `/tokens`
* 请求方法: GET
* 响应格式: HTML页面
* 功能: 调用下面的各种相关API的示例页面

#### 获取Token信息

* 接口地址: `/tokens/get`
* 请求方法: POST
* 认证方式: Bearer Token
* 响应格式:

```json
{
  "status": "success",
  "tokens": [
    [
      number,
      string,
      {
        "bundle": {
          "primary_token": string,
          "secondary_token": string, // 可选
          "checksum": {
            "first": string,
            "second": string,
          },
          "client_key": string, // 可选，非空时显示
          "config_version": string, // 可选
          "session_id": string, // 可选
          "proxy": string, // 可选
          "timezone": string, // 可选
          "gcpp_host": object, // 可选
          "user": { // 可选
            "email": string,
            "name": string,
            "updated_at": string,
            "picture": string, // 可选
            "is_on_new_pricing": boolean
          }
        },
        "status": "enabled" | "disabled",
        "stripe": { // 可选
          "membership_type": "free" | "free_trial" | "pro" | "pro_plus" | "ultra" | "enterprise",
          "payment_id": string, // 可选
          "days_remaining_on_trial": number,
          "subscription_status": "trialing" | "active" | "incomplete" | "incomplete_expired" | "past_due" | "canceled" | "unpaid" | "paused", // 可选
          "verified_student": boolean, // 可选
          "is_on_student_plan": boolean // 可选
        }
      }
    ]
  ],
  "tokens_count": number
}
```

#### 设置Token信息

* 接口地址: `/tokens/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
[
  [
    string,
    {
      "bundle": {
        "primary_token": string,
        "secondary_token": string, // 可选
        "checksum": {
          "first": string,
          "second": string,
        },
        "client_key": string, // 可选
        "config_version": string, // 可选
        "session_id": string, // 可选
        "proxy": string, // 可选
        "timezone": string, // 可选
        "gcpp_host": object, // 可选
        "user": { // 可选
          "email": string,
          "name": string,
          "updated_at": string,
          "picture": string, // 可选
          "is_on_new_pricing": boolean
        }
      },
      "status": "enabled" | "disabled",
      "stripe": { // 可选
        "membership_type": "free" | "free_trial" | "pro" | "pro_plus" | "ultra" | "enterprise",
        "payment_id": string, // 可选
        "days_remaining_on_trial": number,
        "subscription_status": "trialing" | "active" | "incomplete" | "incomplete_expired" | "past_due" | "canceled" | "unpaid" | "paused", // 可选
        "verified_student": boolean // 可选
      }
    }
  ]
]
```

* 响应格式:

```json
{
  "status": "success",
  "tokens_count": number,
  "message": "Token files have been updated and reloaded"
}
```

#### 添加Token

* 接口地址: `/tokens/add`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "tokens": [
    {
      "alias": string, // 可选，无则自动生成
      "token": string,
      "checksum": string, // 可选，无则自动生成
      "client_key": string, // 可选，无则自动生成
      "session_id": string, // 可选
      "config_version": string, // 可选
      "proxy": string, // 可选
      "timezone": string, // 可选
      "gcpp_host": string // 可选
    }
  ],
  "status": "enabled" | "disabled"
}
```

* 响应格式:

```json
{
  "status": "success",
  "tokens_count": number,
  "message": string  // "New tokens have been added and reloaded" 或 "No new tokens were added"
}
```

#### 删除Token

* 接口地址: `/tokens/del`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "aliases": [string], // 要删除的token列表
  "include_failed_tokens": boolean // 默认为false
}
```

* 响应格式:

```json
{
  "status": "success",
  "failed_tokens": [string] // 可选，根据include_failed_tokens返回，表示未找到的token列表
}
```

* expectation说明:
  - simple: 只返回基本状态
  - updated_tokens: 返回更新后的token列表
  - failed_tokens: 返回未找到的token列表
  - detailed: 返回完整信息（包括updated_tokens和failed_tokens）

#### 设置Tokens标签（已弃用）

* 接口地址: `/tokens/tags/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "tokens": [string],
  "tags": {
    string: null | string // 键可以为 timezone: 时区标识符 或 proxy: 代理名称
  }
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": string  // "标签更新成功"
}
```

#### 更新令牌Profile

* 接口地址: `/tokens/profile/update`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
[
  string // aliases
]
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已更新{}个令牌配置, {}个令牌更新失败"
}
```

#### 更新令牌CV

* 接口地址: `/tokens/config-version/update`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
[
  string // aliases
]
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已更新{}个令牌配置版本, {}个令牌更新失败"
}
```

#### 刷新令牌

* 接口地址: `/tokens/refresh`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
[
  string // aliases
]
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已刷新{}个令牌, {}个令牌刷新失败"
}
```

#### 设置令牌状态

* 接口地址: `/tokens/status/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "aliases": [string],
  "status": "enabled" | "disabled"
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已设置{}个令牌状态, {}个令牌设置失败"
}
```

#### 设置令牌别名

* 接口地址: `/tokens/alias/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "{old_alias}": "{new_alias}"
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已设置{}个令牌别名, {}个令牌设置失败"
}
```

#### 设置Tokens代理

* 接口地址: `/tokens/proxy/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "aliases": [string],
  "proxy": string  // 可选，代理地址，null表示清除代理
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已设置{}个令牌代理, {}个令牌设置失败"
}
```

#### 设置Tokens时区

* 接口地址: `/tokens/timezone/set`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "aliases": [string],
  "timezone": string  // 可选，时区标识符（如"Asia/Shanghai"），null表示清除时区
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": "已设置{}个令牌时区, {}个令牌设置失败"
}
```

#### 构建API Key

* 接口地址: `/build-key`
* 请求方法: POST
* 认证方式: Bearer Token (当SHARE_AUTH_TOKEN启用时需要)
* 请求格式:

```json
{
  "token": string,               // 格式: JWT
  "checksum": {
    "first": string,             // 格式: 长度为64的Hex编码字符串
    "second": string,            // 格式: 长度为64的Hex编码字符串
  },
  "client_key": string,          // 格式: 长度为64的Hex编码字符串
  "config_version": string,      // 格式: UUID
  "session_id": string,          // 格式: UUID
  "secret": string,              // 可选，没什么用
  "proxy_name": string,          // 可选，指定代理
  "timezone": string,            // 可选，指定时区
  "gcpp_host": string,           // 可选，代码补全区域
  "disable_vision": boolean,       // 可选，禁用图片处理能力
  "enable_slow_pool": boolean,     // 可选，启用慢速池
  "include_web_references": boolean,
  "usage_check_models": {          // 可选，使用量检查模型配置
    "type": "default" | "disabled" | "all" | "custom",
    "model_ids": string  // 当type为custom时生效，以逗号分隔的模型ID列表
  }
}
```

* 响应格式:

```json
{
  "keys": [string]    // 成功时返回生成的key
}
```

或出错时:

```json
{
  "error": string  // 错误信息
}
```

说明：

1. 此接口用于生成携带动态配置的API Key，是对直接传token与checksum模式的升级版本，在0.3起，直接传token与checksum的方式已经不再适用

2. 生成的key格式为: `sk-{encoded_config}`，其中sk-为默认前缀(可配置)

3. usage_check_models配置说明:
   - default: 使用默认模型列表(同下 `usage_check_models` 字段的默认值)
   - disabled: 禁用使用量检查
   - all: 检查所有可用模型
   - custom: 使用自定义模型列表(需在model_ids中指定)

4. 在当前版本，keys数组长度总为3，后2个基于缓存，仅第1个使用过才行：
   1. 完整key，旧版本也存在
   2. 数字key的base64编码版本
   3. 数字key的明文版本

5. 数字key是一个128位无符号整数与一个64位无符号整数组成的，比通常使用的uuid更难破解。

#### 获取Config Version

* 接口地址: `/config-version`
* 请求方法: POST
* 认证方式: Bearer Token (当SHARE_AUTH_TOKEN启用时需要)
* 请求格式:

```json
{
  "token": string,               // 格式: JWT
  "checksum": {
    "first": string,             // 格式: 长度为64的Hex编码字符串
    "second": string,            // 格式: 长度为64的Hex编码字符串
  },
  "client_key": string,          // 格式: 长度为64的Hex编码字符串
  "session_id": string,          // 格式: UUID
  "proxy_name": string,          // 可选，指定代理
  "timezone": string,            // 可选，指定时区
  "gcpp_host": string            // 可选，代码补全区域
}
```

* 响应格式:

```json
{
  "config_version": string    // 成功时返回生成的UUID
}
```

或出错时:

```json
{
  "error": string  // 错误信息
}
```

### 代理管理接口

#### 简易代理信息管理页面

* 接口地址: `/proxies`
* 请求方法: GET
* 响应格式: HTML页面
* 功能: 调用下面的各种相关API的示例页面

#### 获取代理配置信息

* 接口地址: `/proxies/get`
* 请求方法: POST
* 响应格式:

```json
{
  "status": "success",
  "proxies": {
    "proxies": {
      "proxy_name": "non" | "sys" | "http://proxy-url",
    },
    "general": string // 当前使用的通用代理名称
  },
  "proxies_count": number,
  "general_proxy": string,
  "message": string // 可选
}
```

#### 设置代理配置

* 接口地址: `/proxies/set`
* 请求方法: POST
* 请求格式:

```json
{
  "proxies": {
    "{proxy_name}": "non" | "sys" | "http://proxy-url"
  },
  "general": string  // 要设置的通用代理名称
}
```

* 响应格式:

```json
{
  "status": "success",
  "proxies_count": number,
  "message": "代理配置已更新"
}
```

#### 添加代理

* 接口地址: `/proxies/add`
* 请求方法: POST
* 请求格式:

```json
{
  "proxies": {
    "{proxy_name}": "non" | "sys" | "http://proxy-url"
  }
}
```

* 响应格式:

```json
{
  "status": "success",
  "proxies_count": number,
  "message": string  // "已添加 X 个新代理" 或 "没有添加新代理"
}
```

#### 删除代理

* 接口地址: `/proxies/del`
* 请求方法: POST
* 请求格式:

```json
{
  "names": [string],  // 要删除的代理名称列表
  "expectation": "simple" | "updated_proxies" | "failed_names" | "detailed"  // 默认为simple
}
```

* 响应格式:

```json
{
  "status": "success",
  "updated_proxies": {  // 可选，根据expectation返回
    "proxies": {
      "proxy_name": "non" | "sys" | "http://proxy-url"
    },
    "general": string
  },
  "failed_names": [string]  // 可选，根据expectation返回，表示未找到的代理名称列表
}
```

#### 设置通用代理

* 接口地址: `/proxies/set-general`
* 请求方法: POST
* 请求格式:

```json
{
  "name": string  // 要设置为通用代理的代理名称
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": "通用代理已设置"
}
```

#### 代理类型说明

* `non`: 表示不使用代理
* `sys`: 表示使用系统代理
* 其他: 表示具体的代理URL地址（如 `http://proxy-url`）

#### 注意事项

1. 代理名称必须是唯一的，添加重复名称的代理会被忽略
2. 设置通用代理时，指定的代理名称必须存在于当前的代理配置中
3. 删除代理时的 expectation 参数说明：
   - simple: 只返回基本状态
   - updated_proxies: 返回更新后的代理配置
   - failed_names: 返回未找到的代理名称列表
   - detailed: 返回完整信息（包括updated_proxies和failed_names）

### 错误格式

所有接口在发生错误时会返回统一的错误格式：

```json
{
  "status": "error",
  "code": number,   // 可选
  "error": string,  // 可选，错误详细信息
  "message": string // 错误提示信息
}
```

### 配置管理接口

#### 配置页面

* 接口地址: `/config`
* 请求方法: GET
* 响应格式: HTML页面
* 功能: 提供配置管理界面,可以修改页面内容和系统配置

#### 更新配置

* 接口地址: `/config`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "action": "get" | "update" | "reset",
  "path": string,
  "content": {
    "type": "default" | "not_found" | "redirect" | "plain_text" | "html" | "css" | "js",
    "value": string  // type=redirect时为URL, type=plain_text/html/css/js时为对应内容
  },
  "vision_ability": "none" | "base64" | "all", // "disabled" | "base64-only" | "base64-http"
  "enable_slow_pool": boolean,
  "enable_long_context": boolean,
  "usage_check_models": {
    "type": "none" | "default" | "all" | "list",
    "content": string
  },
  "enable_dynamic_key": boolean,
  "share_token": string,
  "calibrate_token": string,
  "include_web_references": boolean
}
```

* 响应格式:

```json
{
  "status": "success",
  "message": string,
  "data": {
    "content": {
      "type": "default" | "not_found" | "redirect" | "plain_text" | "html" | "css" | "js",
      "value": string
    },
    "vision_ability": "none" | "base64" | "all",
    "enable_slow_pool": boolean,
    "enable_long_context": boolean,
    "usage_check_models": {
      "type": "none" | "default" | "all" | "list",
      "content": string
    },
    "enable_dynamic_key": boolean,
    "share_token": string,
    "calibrate_token": string,
    "include_web_references": boolean
  }
}
```

注意：`usage_check_models` 字段的默认值为非"cursor-small"、"deepseek-v3.1"、"grok-3-mini"的所有模型。

这些模型将默认进行使用量检查。您可以通过配置接口修改此设置。

### 日志管理接口

#### 获取日志接口

* 接口地址: `/logs`
* 请求方法: GET
* 响应格式: 根据配置返回不同的内容类型(默认、文本或HTML)

#### 获取日志数据

* 接口地址: `/logs/get`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
{
  "query": {
    // 分页与排序控制
    "limit": number,             // 可选，返回记录数量限制
    "offset": number,            // 可选，起始位置偏移量
    "reverse": boolean,          // 可选，反向排序，默认false（从旧到新），true时从新到旧

    // 时间范围过滤
    "from_date": string,         // 可选，开始日期时间，RFC3339格式
    "to_date": string,           // 可选，结束日期时间，RFC3339格式

    // 用户标识过滤
    "user_id": string,           // 可选，按用户ID精确匹配
    "email": string,             // 可选，按用户邮箱过滤（支持部分匹配）
    "membership_type": string,   // 可选，按会员类型过滤 ("free"/"free_trial"/"pro"/"enterprise")

    // 核心业务过滤
    "status": string,            // 可选，按状态过滤 ("pending"/"success"/"failure")
    "model": string,             // 可选，按模型名称过滤（支持部分匹配）
    "include_models": [string],  // 可选，包含特定模型
    "exclude_models": [string],  // 可选，排除特定模型

    // 请求特征过滤
    "stream": boolean,           // 可选，是否为流式请求
    "has_chain": boolean,        // 可选，是否包含对话链
    "has_usage": boolean,        // 可选，是否有usage信息

    // 错误相关过滤
    "has_error": boolean,        // 可选，是否包含错误
    "error": string,             // 可选，按错误过滤（支持部分匹配）

    // 性能指标过滤
    "min_total_time": number,    // 可选，最小总耗时（秒）
    "max_total_time": number,    // 可选，最大总耗时（秒）
    "min_tokens": number,        // 可选，最小token数（input + output）
    "max_tokens": number         // 可选，最大token数
  }
}
```

* 响应格式:

```json
{
  "total": number,
  "logs": [
    {
      "id": number,
      "timestamp": string,
      "model": string,
      "token_info": {
        "key": string,
        "stripe": { // 可选
          "membership_type": "free" | "free_trial" | "pro" | "pro_plus" | "ultra" | "enterprise",
          "payment_id": string, // 可选
          "days_remaining_on_trial": number,
          "subscription_status": "trialing" | "active" | "incomplete" | "incomplete_expired" | "past_due" | "canceled" | "unpaid" | "paused", // 可选
          "verified_student": boolean // 可选
        }
      },
      "chain": {
        "prompt": [ // array or string
          {
            "role": string,
            "content": string
          }
        ],
        "delays": [
          string,
          [
            number, // chars count
            number // time
          ]
        ],
        "usage": { // optional
          "input": number,
          "output": number,
        }
      },
      "timing": {
        "total": number
      },
      "stream": boolean,
      "status": string,
      "error": string
    }
  ],
  "timestamp": string,
  "status": "success"
}
```

* 说明：
  - 所有查询参数都是可选的
  - 管理员可以查看所有日志，普通用户只能查看与其token相关的日志
  - 如果提供了无效的状态或会员类型，将返回空结果
  - 日期时间格式需遵循 RFC3339 标准，如："2024-03-20T15:30:00+08:00"
  - 邮箱和模型名称支持部分匹配

#### 获取日志令牌

* 接口地址: `/logs/tokens/get`
* 请求方法: POST
* 认证方式: Bearer Token
* 请求格式:

```json
[
  string
]
```

* 响应格式:

```json
{
  "status": "success",
  "tokens": {
    "{key}": {
      "primary_token": string,
      "secondary_token": string, // 可选
      "checksum": {
        "first": string,
        "second": string,
      },
      "client_key": string, // 可选，非空时显示
      "config_version": string, // 可选
      "session_id": string, // 可选
      "proxy": string, // 可选
      "timezone": string, // 可选
      "gcpp_host": object, // 可选
      "user": { // 可选
        "email": string,
        "name": string,
        "updated_at": string,
        "picture": string, // 可选
        "is_on_new_pricing": boolean
      }
    }
  },
  "total": number,
  "timestamp": string
}
```

### 静态资源接口

#### 获取共享样式

* 接口地址: `/static/shared-styles.css`
* 请求方法: GET
* 响应格式: CSS文件
* 功能: 获取共享样式表

#### 获取共享脚本

* 接口地址: `/static/shared.js`
* 请求方法: GET
* 响应格式: JavaScript文件
* 功能: 获取共享JavaScript代码

#### 获取其他资源

* 接口地址: `/static/{path}`
* 请求方法: GET
* 请求参数:
  - `path`: 静态文件的相对路径

* 响应格式:
  - **成功响应 (200 OK)**:
    - Headers:
      - `Content-Type`: 根据文件扩展名自动设置（见下方MIME类型映射表）
      - `Content-Length`: 文件大小
    - Body: 文件的二进制内容

  - **文件不存在或大小超过4GiB (404 Not Found)**:
    - Headers:
      - `Content-Type`: `text/plain; charset=utf-8`
    - Body: 错误信息

* 支持的MIME类型映射:
  - 文本类型: html, htm, txt, css, js, mjs, csv, xml, md, markdown
  - 图像类型: jpg, jpeg, png, gif, webp, svg, bmp, ico, tiff, tif, avif
  - 音频类型: mp3, mp4a, wav, ogg, oga, weba, aac, flac, m4a
  - 视频类型: mp4, mpeg, mpg, webm, ogv, avi, mov, qt, flv
  - 文档类型: pdf, doc, docx, xls, xlsx, ppt, pptx
  - 压缩文件: zip, rar, 7z, gz, gzip, tar
  - 字体类型: ttf, otf, woff, woff2
  - 其他类型: 默认为 `application/octet-stream`

* 功能: 获取从环境变量DATA_DIR指定的目录下的子目录static下的文件。

#### 环境变量示例

* 接口地址: `/env-example`
* 请求方法: GET
* 响应格式: 文本文件
* 功能: 获取环境变量配置示例

### 健康检查接口

* **接口地址**: `/health` 或 `/`(重定向)
* **请求方法**: GET
* **认证方式**: 无需
* **响应格式**: 根据配置返回不同的内容类型(默认JSON、文本或HTML)

#### 响应结构

```json
{
  "status": "success",
  "service": {
    "name": "cursor-api",
    "version": "1.0.0",
    "is_debug": false,
    "build": {
      "version": 1,
      "timestamp": "2024-01-15T10:30:00Z",
      "is_debug": false,
      "is_prerelease": false
    }
  },
  "runtime": {
    "started_at": "2024-01-15T10:00:00+08:00",
    "uptime_seconds": 1800,
    "requests": {
      "total": 1250,
      "active": 3,
      "errors": 12
    }
  },
  "system": {
    "memory": {
      "used_bytes": 134217728,
      "used_percentage": 12.5,
      "available_bytes": 1073741824
    },
    "cpu": {
      "usage_percentage": 15.2,
      "load_average": [0.8, 1.2, 1.5]
    }
  },
  "capabilities": {
    "models": ["gpt-4", "claude-3"],
    "endpoints": ["/chat", "/completions", "/embeddings"],
    "features": ["streaming", "function_calling", "vision"]
  }
}
```

#### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `status` | string | 服务状态: "success", "warning", "error" |
| `service.name` | string | 服务名称 |
| `service.version` | string | 服务版本 |
| `service.is_debug` | boolean | 是否为调试模式 |
| `service.build.version` | number | 构建版本号(仅preview功能启用时) |
| `service.build.timestamp` | string | 构建时间戳 |
| `service.build.is_prerelease` | boolean | 是否为预发布版本 |
| `runtime.started_at` | string | 服务启动时间 |
| `runtime.uptime_seconds` | number | 运行时长(秒) |
| `runtime.requests.total` | number | 总请求数 |
| `runtime.requests.active` | number | 当前活跃请求数 |
| `runtime.requests.errors` | number | 错误请求数 |
| `system.memory.used_bytes` | number | 已使用内存(字节) |
| `system.memory.used_percentage` | number | 内存使用率(%) |
| `system.memory.available_bytes` | number | 可用内存(字节,可选) |
| `system.cpu.usage_percentage` | number | CPU使用率(%) |
| `system.cpu.load_average` | array | 系统负载[1分钟,5分钟,15分钟] |
| `capabilities.models` | array | 支持的模型列表 |
| `capabilities.endpoints` | array | 可用的API端点 |
| `capabilities.features` | array | 支持的功能特性 |

### 其他接口

#### 随机生成一个uuid

* 接口地址: `/gen-uuid`
* 请求方法: GET
* 响应格式:

```plaintext
string
```

#### 随机生成一个hash

* 接口地址: `/gen-hash`
* 请求方法: GET
* 响应格式:

```plaintext
string
```

#### 随机生成一个checksum

* 接口地址: `/gen-checksum`
* 请求方法: GET
* 响应格式:

```plaintext
string
```

#### 随机生成一个token

* 接口地址: `/gen-token`
* 请求方法: GET
* 响应格式:

```plaintext
string
```

#### 获取当前的tsheader

* 接口地址: `/get-tsheader`
* 请求方法: GET
* 响应格式:

```plaintext
string
```

#### 获取用户信息（已弃用）

* 接口地址: `/userinfo`
* 请求方法: POST
* 认证方式: 请求体中包含token
* 请求格式:

```json
{
  "token": string
}
```

* 响应格式:

```json
{
  "usage": {
    "premium": {
      "num_requests": number,
      "total_requests": number,
      "num_tokens": number,
      "max_requests": number,
      "max_tokens": number
    },
    "standard": {
      "num_requests": number,
      "total_requests": number,
      "num_tokens": number,
      "max_requests": number,
      "max_tokens": number
    },
    "start_of_month": string
  },
  "user": {
    "email": string,
    "name": string,
    "id": string,
    "updated_at": string
  },
  "stripe": {
    "membership_type": "free" | "free_trial" | "pro" | "enterprise",
    "payment_id": string,
    "days_remaining_on_trial": number
  }
}
```

如果发生错误，响应格式为:

```json
{
  "error": string
}
```

#### 获取更新令牌（已弃用）

* 接口地址: `/token-upgrade`
* 请求方法: POST
* 认证方式: 请求体中包含token
* 请求格式:

```json
{
  "token": string
}
```

* 响应格式:

```json
{
  "status": "success" | "failure" | "error",
  "message": string,
  "result": string // optional
}
```

#### 基础校准（已弃用）

* 接口地址: `/basic-calibration`
* 请求方法: POST
* 认证方式: 请求体中包含token
* 请求格式:

```json
{
  "token": string
}
```

* 响应格式:

```json
{
  "status": "success" | "error",
  "message": string,
  "user_id": string,
  "create_at": string
}
```

注意: `user_id` 和 `create_at` 字段在校验失败时可能不存在。

## Copilot++ 接口文档

1. 相关接口都需要 `x-client-key`, 格式请见 `/gen-hash`（32字节）。
2. Cookie `FilesyncCookie` 是16字节，工作区不变即不变。
3. 关于形如 `AWSALBAPP-0` 的 Cookie 具有7天有效期，可能变化，详情请查阅 Amazon 相关文档。
4. `FilesyncCookie` 和 `AWSALBAPP` 总是被 `/file/upload` 或 `/file/sync`。
5. 以下所有接口都使用 POST 方法，且都需要认证。

### 获取代码补全服务的配置信息

* 接口地址: `/cpp/config`

#### 请求格式

```json
{
  "is_nightly": boolean,  // 可选，是否使用nightly版本
  "model": string,        // 模型名称
  "supports_cpt": boolean // 可选，是否支持CPT
}
```

### 响应格式

```json
{
  "above_radius": number,                                        // 可选，上方扫描半径
  "below_radius": number,                                        // 可选，下方扫描半径
  "merge_behavior": {                                            // 可选，合并行为
    "type": string,
    "limit": number,                                             // 可选，限制
    "radius": number                                             // 可选，半径
  },
  "is_on": boolean,                                              // 可选，是否开启
  "is_ghost_text": boolean,                                      // 可选，是否使用幽灵文本
  "should_let_user_enable_cpp_even_if_not_pro": boolean,         // 可选，非专业用户是否可以启用
  "heuristics": [                                                // 启用的启发式规则列表
    "lots_of_added_text",
    "duplicating_line_after_suggestion",
    "duplicating_multiple_lines_after_suggestion",
    "reverting_user_change",
    "output_extends_beyond_range_and_is_repeated",
    "suggesting_recently_rejected_edit"
  ],
  "exclude_recently_viewed_files_patterns": [string],            // 最近查看文件排除模式
  "enable_rvf_tracking": boolean,                                // 是否启用RVF跟踪
  "global_debounce_duration_millis": number,                     // 全局去抖动时间(毫秒)
  "client_debounce_duration_millis": number,                     // 客户端去抖动时间(毫秒)
  "cpp_url": string,                                             // CPP服务URL
  "use_whitespace_diff_history": boolean,                        // 是否使用空白差异历史
  "import_prediction_config": {                                  // 导入预测配置
    "is_disabled_by_backend": boolean,                           // 是否被后端禁用
    "should_turn_on_automatically": boolean,                     // 是否自动开启
    "python_enabled": boolean                                    // Python是否启用
  },
  "enable_filesync_debounce_skipping": boolean,                  // 是否启用文件同步去抖动跳过
  "check_filesync_hash_percent": number,                         // 文件同步哈希检查百分比
  "geo_cpp_backend_url": string,                                 // 地理位置CPP后端URL
  "recently_rejected_edit_thresholds": {                         // 可选，最近拒绝编辑阈值
    "hard_reject_threshold": number,                             // 硬拒绝阈值
    "soft_reject_threshold": number                              // 软拒绝阈值
  },
  "is_fused_cursor_prediction_model": boolean,                   // 是否使用融合光标预测模型
  "include_unchanged_lines": boolean,                            // 是否包含未更改行
  "should_fetch_rvf_text": boolean,                              // 是否获取RVF文本
  "max_number_of_cleared_suggestions_since_last_accept": number, // 可选，上次接受后清除建议的最大数量
  "suggestion_hint_config": {                                    // 可选，建议提示配置
    "important_lsp_extensions": [string],                        // 重要的LSP扩展
    "enabled_for_path_extensions": [string]                      // 启用的路径扩展
  }
}
```

### 获取可用的代码补全模型列表

* 接口地址: `/cpp/models`

#### 请求格式

无

### 响应格式

```json
{
  "models": [string],     // 可用模型列表
  "default_model": string // 可选，默认模型
}
```

### 上传文件

* 接口地址: `/file/upload`

#### 请求格式

```json
{
  "uuid": string,                    // 文件标识符
  "relative_workspace_path": string, // 文件相对于工作区的路径
  "contents": string,                // 文件内容
  "model_version": number,           // 模型版本
  "sha256_hash": string              // 可选，文件的SHA256哈希值
}
```

### 响应格式

```json
{
  "error": string // 错误类型：unspecified, non_existant, hash_mismatch
}
```

### 同步文件变更

* 接口地址: `/file/sync`

#### 请求格式

```json
{
  "uuid": string,                                // 文件标识符
  "relative_workspace_path": string,             // 文件相对于工作区的路径
  "model_version": number,                       // 模型版本
  "filesync_updates": [                          // 文件同步更新数组
    {
      "model_version": number,                   // 模型版本
      "relative_workspace_path": string,         // 文件相对于工作区的路径
      "updates": [                               // 单个更新请求数组
        {
          "start_position": number,              // 更新开始位置
          "end_position": number,                // 更新结束位置
          "change_length": number,               // 变更长度
          "replaced_string": string,             // 替换的字符串
          "range": {                             // 简单范围
            "start_line_number": number,         // 开始行号
            "start_column": number,              // 开始列
            "end_line_number_inclusive": number, // 结束行号（包含）
            "end_column": number                 // 结束列
          }
        }
      ],
      "expected_file_length": number             // 预期文件长度
    }
  ],
  "sha256_hash": string                          // 文件的SHA256哈希值
}
```

### 响应格式

```json
{
  "error": string // 错误类型：unspecified, non_existant, hash_mismatch
}
```

### 流式代码补全

* 接口地址: `/cpp/stream`

#### 请求格式

```json
{
  "current_file": {                                               // 当前文件信息
    "relative_workspace_path": string,                            // 文件相对于工作区的路径
    "contents": string,                                           // 文件内容
    "rely_on_filesync": boolean,                                  // 是否依赖文件同步
    "sha256_hash": string,                                        // 可选，SHA256哈希值
    "top_chunks": [                                               // 顶级代码块
      {
        "content": string,                                        // 内容
        "range": {                                                // 最简单范围
          "start_line": number,                                   // 开始行
          "end_line_inclusive": number                            // 结束行（包含）
        },
        "score": number,                                          // 分数
        "relative_path": string                                   // 相对路径
      }
    ],
    "contents_start_at_line": number,                             // 内容开始行
    "cursor_position": {                                          // 光标位置
      "line": number,                                             // 行号
      "column": number                                            // 列号
    },
    "dataframes": [                                               // 数据框信息
      {
        "name": string,                                           // 名称
        "shape": string,                                          // 形状
        "data_dimensionality": number,                            // 数据维度
        "columns": [                                              // 列
          {
            "key": string,                                        // 键
            "type": string                                        // 类型
          }
        ],
        "row_count": number,                                      // 行数
        "index_column": string                                    // 索引列
      }
    ],
    "total_number_of_lines": number,                              // 总行数
    "language_id": string,                                        // 语言ID
    "selection": {                                                // 选择范围
      "start_position": {                                         // 开始位置
        "line": number,                                           // 行号
        "column": number                                          // 列号
      },
      "end_position": {                                           // 结束位置
        "line": number,                                           // 行号
        "column": number                                          // 列号
      }
    },
    "alternative_version_id": number,                             // 可选，替代版本ID
    "diagnostics": [                                              // 诊断信息
      {
        "message": string,                                        // 消息
        "range": {                                                // 范围
          "start_position": {                                     // 开始位置
            "line": number,                                       // 行号
            "column": number                                      // 列号
          },
          "end_position": {                                       // 结束位置
            "line": number,                                       // 行号
            "column": number                                      // 列号
          }
        },
        "severity": "error" | "warning" | "information" | "hint", // 严重程度
        "related_information": [                                  // 相关信息
          {
            "message": string,                                    // 消息
            "range": {                                            // 范围
              "start_position": {                                 // 开始位置
                "line": number,                                   // 行号
                "column": number                                  // 列号
              },
              "end_position": {                                   // 结束位置
                "line": number,                                   // 行号
                "column": number                                  // 列号
              }
            }
          }
        ]
      }
    ],
    "file_version": number,                                       // 可选，文件版本
    "cell_start_lines": [number],                                 // 单元格开始行
    "workspace_root_path": string                                 // 工作区根路径
  },
  "diff_history": [string],                                       // 差异历史
  "model_name": string,                                           // 可选，模型名称
  "linter_errors": {                                              // 可选，Linter错误
    "relative_workspace_path": string,                            // 文件相对于工作区的路径
    "errors": [                                                   // 错误数组
      {
        "message": string,                                        // 错误消息
        "range": {                                                // 范围
          "start_position": {                                     // 开始位置
            "line": number,                                       // 行号
            "column": number                                      // 列号
          },
          "end_position": {                                       // 结束位置
            "line": number,                                       // 行号
            "column": number                                      // 列号
          }
        },
        "source": string,                                         // 可选，来源
        "related_information": [                                  // 相关信息数组
          {
            "message": string,                                    // 相关信息消息
            "range": {                                            // 相关信息范围
              "start_position": {                                 // 开始位置
                "line": number,                                   // 行号
                "column": number                                  // 列号
              },
              "end_position": {                                   // 结束位置
                "line": number,                                   // 行号
                "column": number                                  // 列号
              }
            }
          }
        ],
        "severity": "error" | "warning" | "information" | "hint"  // 可选，严重程度
      }
    ],
    "file_contents": string                                       // 文件内容
  },
  "context_items": [                                              // 上下文项
    {
      "contents": string,                                         // 内容
      "symbol": string,                                           // 可选，符号
      "relative_workspace_path": string,                          // 相对工作区路径
      "score": number                                             // 分数
    }
  ],
  "diff_history_keys": [string],                                  // 差异历史键
  "give_debug_output": boolean,                                   // 可选，提供调试输出
  "file_diff_histories": [                                        // 文件差异历史
    {
      "file_name": string,                                        // 文件名
      "diff_history": [string],                                   // 差异历史
      "diff_history_timestamps": [number]                         // 差异历史时间戳
    }
  ],
  "merged_diff_histories": [                                      // 合并差异历史
    {
      "file_name": string,                                        // 文件名
      "diff_history": [string],                                   // 差异历史
      "diff_history_timestamps": [number]                         // 差异历史时间戳
    }
  ],
  "block_diff_patches": [                                         // 块差异补丁
    {
      "start_model_window": {                                     // 开始模型窗口
        "lines": [string],                                        // 行
        "start_line_number": number,                              // 开始行号
        "end_line_number": number                                 // 结束行号
      },
      "changes": [                                                // 变更
        {
          "text": string,                                         // 文本
          "range": {                                              // 范围
            "start_line_number": number,                          // 开始行号
            "start_column": number,                               // 开始列
            "end_line_number": number,                            // 结束行号
            "end_column": number                                  // 结束列
          }
        }
      ],
      "relative_path": string,                                    // 相对路径
      "model_uuid": string,                                       // 模型UUID
      "start_from_change_index": number                           // 开始变更索引
    }
  ],
  "is_nightly": boolean,                                          // 可选，是否为nightly版本
  "is_debug": boolean,                                            // 可选，是否为调试模式
  "immediately_ack": boolean,                                     // 可选，立即确认
  "enable_more_context": boolean,                                 // 可选，启用更多上下文
  "parameter_hints": [                                            // 参数提示
    {
      "label": string,                                            // 标签
      "documentation": string                                     // 可选，文档
    }
  ],
  "lsp_contexts": [                                               // LSP上下文
    {
      "uri": string,                                              // URI
      "symbol_name": string,                                      // 符号名称
      "positions": [                                              // 位置
        {
          "line": number,                                         // 行
          "character": number                                     // 字符
        }
      ],
      "context_items": [                                          // 上下文项
        {
          "uri": string,                                          // 可选，URI
          "type": string,                                         // 类型
          "content": string,                                      // 内容
          "range": {                                              // 可选，范围
            "start_line": number,                                 // 开始行
            "start_character": number,                            // 开始字符
            "end_line": number,                                   // 结束行
            "end_character": number                               // 结束字符
          }
        }
      ],
      "score": number                                             // 分数
    }
  ],
  "cpp_intent_info": {                                            // 可选，代码补全意图信息
    "source": string                                              // 来源
  },
  "workspace_id": string,                                         // 可选，工作区ID
  "additional_files": [                                           // 附加文件
    {
      "relative_workspace_path": string,                          // 相对工作区路径
      "is_open": boolean,                                         // 是否打开
      "visible_range_content": [string],                          // 可见范围内容
      "last_viewed_at": number,                                   // 可选，最后查看时间
      "start_line_number_one_indexed": [number],                  // 从1开始索引的起始行号
      "visible_ranges": [                                         // 可见范围
        {
          "start_line_number": number,                            // 开始行号
          "end_line_number_inclusive": number                     // 结束行号（包含）
        }
      ]
    }
  ],
  "control_token": "quiet" | "loud" | "op",                       // 可选，控制标记
  "client_time": number,                                          // 可选，客户端时间
  "filesync_updates": [                                           // 文件同步更新
    {
      "model_version": number,                                    // 模型版本
      "relative_workspace_path": string,                          // 相对工作区路径
      "updates": [                                                // 更新数组
        {
          "start_position": number,                               // 开始位置（字符偏移量）
          "end_position": number,                                 // 结束位置（字符偏移量）
          "change_length": number,                                // 变更长度
          "replaced_string": string,                              // 替换的字符串
          "range": {                                              // 范围
            "start_line_number": number,                          // 开始行号
            "start_column": number,                               // 开始列
            "end_line_number_inclusive": number,                  // 结束行号（包含）
            "end_column": number                                  // 结束列
          }
        }
      ],
      "expected_file_length": number                              // 预期文件长度
    }
  ],
  "time_since_request_start": number,                             // 请求开始后的时间
  "time_at_request_send": number,                                 // 请求发送时的时间
  "client_timezone_offset": number,                               // 可选，客户端时区偏移
  "lsp_suggested_items": {                                        // 可选，LSP建议项
    "suggestions": [                                              // 建议
      {
        "label": string                                           // 标签
      }
    ]
  },
  "supports_cpt": boolean                                         // 可选，是否支持CPT
}
```

### 响应格式 (SSE流格式)

事件类型及对应数据格式：

1. **model_info**
```json
{
  "type": "model_info",
  "is_fused_cursor_prediction_model": boolean,
  "is_multidiff_model": boolean
}
```

2. **range_replace**
```json
{
  "type": "range_replace",
  "start_line_number": number,
  "end_line_number_inclusive": number,
  "text": string
}
```

3. **cursor_prediction**
```json
{
  "type": "cursor_prediction",
  "relative_path": string,
  "line_number_one_indexed": number,
  "expected_content": string,
  "should_retrigger_cpp": boolean
}
```

4. **text**
```json
{
  "type": "text",
  "text": string
}
```

5. **done_edit**
```json
{
  "type": "done_edit"
}
```

6. **done_stream**
```json
{
  "type": "done_stream"
}
```

7. **debug**
```json
{
  "type": "debug",
  "model_input": string,
  "model_output": string,
  "total_time": string,   // 可选
  "stream_time": string,
  "ttft_time": string,
  "server_timing": string // 可选
}
```

8. **error**
```json
{
  "type": "error",
  "message": string
}
```

9. **stream_end**
```json
{
  "type": "stream_end"
}
```

#### 来源可选值

- line_change
- typing
- option_hold
- linter_errors
- parameter_hints
- cursor_prediction
- manual_trigger
- editor_change
- lsp_suggestions

## 鸣谢

感谢以下项目和贡献者:

- [cursor-api](https://github.com/wisdgod/cursor-api) - 本项目本身
- [zhx47/cursor-api](https://github.com/zhx47/cursor-api) - 提供了本项目起步阶段的主要参考
- [luolazyandlazy/cursorToApi](https://github.com/luolazyandlazy/cursorToApi)

### 偷偷写在最后的话

虽然作者觉得~骗~收点钱合理，但不强求，要是**主动自愿**发我我肯定收（因为真有人这么做，虽然不是赞助），赞助很合理吧

不是**主动自愿**就算了，不是很缺，给了会很感动罢了。

虽然不是很建议你赞助，但如果你赞助了，大概可以：

* 测试版更新
* 要求功能
* 问题更快解决

即使如此，我也保留可以拒绝赞助和拒绝要求的权利。

求赞助还是有点不要脸了，接下来是吐槽：

~~辛辛苦苦做这个也不知道是为了谁，好累。其实还有很多功能可以做，比如直接传token支持配置（其实这个要专门做一个页面），这个作为rc.4的计划之一吧。~~

~~主要没想做用户管理，所以不存在是否接入LinuxDo的问题。虽然那个半成品公益版做好了就是了。~~

就说这么多，没啥可说的，不管那么多，做就完了。\[doge\] 自己想象吧。

~~为什么一直说要跑路呢？主要是有时Cursor的Claude太假了，堪比gpt-4o-mini，我对比发现真没啥差别，比以前差远了，无力了，所以不太想做了。我也感觉很奇怪。~~

~~查询额度会在一开始检测导致和完成时的额度有些差别，但是懒得改了，反正差别不大，对话也没响应内容，恰好完成了统一。~~

有人说少个二维码来着，还是算了。如果觉得好用，给点支持。其实没啥大不了的，没兴趣就不做了。不想那么多了。

要不给我邮箱发口令红包？

过了差不多两个多月，继续吐槽：

我都不知道为什么现在还在更新，明明我自己都不用的，一看到bug反馈我就尽量马上去解决问题。不知道说什么好了。

真得给我磕一个。

过了5月有余，走走停停，真不容易啊！
