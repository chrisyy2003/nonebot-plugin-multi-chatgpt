# 多账户ChatGPT

>   项目源于ChatGPT公测一周的时间内，群中500+人数导致的请求频率问题，从而需要多个Bot分担请求压力。
>
>   此项目是直接基于revChatGPT，基于逆向方式插件并不调用API接口，并且API接口也不需要多账号。
>
>   目前cloudflare的限制，存在不加代理启动成功之后，只能回复一句话的问题（基于逆向API的应该都有这个问题）。

## 安装

第一种方式

```
nb plugin install nonebot_plugin_multi_chatgpt
```

---

第二种方式，使用一下命令安装

```
pip3 install nonebot_plugin_multi_chatgpt --upgrade
```

随后在`bot.py`中加上如下代码，加载插件

```
nonebot.load_plugin('nonebot_plugin_multi_chatgpt')
```

## 配置

在`env.dev`中填入如下配置，根据注释可以自行修改
```
chatgpt_token_path = "config/chatgpt_token.yml" # token路径
chatgpt_command_prefix = "。"                   # 指令前缀
chatgpt_need_at = False                         # 是否需要@
chatgpt_image_render = False                    # 是否需要图片渲染
chatgpt_proxy = "http://127.0.0.1:6512"         # 代理
```

bot的token，请写在您配置的`chatgpt_token_path`下面，默认路径是`config/chatgpt_img_config.yml`

多个token请按照如下格式填写
```
session_tokens:
  - XXX
  - YYY
```

获取token得方法，打开`Application`选项卡`Cookie`，复制值`__Secure-next-auth.session-token`并其中的值即可。

![](https://chrisyy-images.oss-cn-chengdu.aliyuncs.com/img/image-20221205094326498.png)

## 图片相关

### 配置

采用命令进行配置，在此进行简单说明

- 在 bot 连接时，将会在`机器人项目/`自动创建一个名为 `config_multi_chatgpt` 的文件夹（如果不存在），用于存放输出图片相关的配置文件
- 配置文件名为 `img_out_config.yml`, 初始化时，将写入 `{"global":False}`，代表全局不开启图片输出
- 在使用以下命令时，会分别在配置中加入`{"groups":[123456,45656,xxx]}` `{"users": [17960000,666,xxx]}` 其分别代表开启图片输出的**群**和**用户**

### 指令


| 操作       | 命令                 | 权限             | 备注                                               | 命令别名                   |
|----------|--------------------| ---------------- |--------------------------------------------------|------------------------|
| 全局图片输出   | 全局图片开<br />全局图片关   | SUPERUSER        | 开启后，在任何时候都返回图片                                   | gpt全局图片开<br />gpt全局图片关 |
| 群图片输出    | 群图片开<br />群图片关     | 群主、管理员、SU | 需要在群内发送，开启后，在该群任何时候都返回图片                         | gpt群图片开<br />gpt群图片关   |
| 用户输出     | 对我输出开<br />对我输出关   | /                | 开启后，对于该用户在任何时候都返回图片                              | 对我图片输出开<br />对我图片输出关   |
| 某条消息输出   | -p 你好<br />你好 -p   | /                | 在某次请求时，在你要说的话前或者后加`-p` <br /> 则对此条消息，会返回图片       |                        |
| 查看配置     | 查看输出               | SUPERUSER        | 返回一张图片，包含了图片输出的开关配置                              | 查看图片开关<br />查看图片输出     |
| su删除某开启的群 | 图片输出删除 g 1111 2222 | SUPERUSER        | 关闭某几个开启图片输出的群，`g`作为参数,<br />不带参数默认为`g`，后跟群号用空格分隔 | 输出删除 g 1111 2222       |
| su删除某开启的用户 | 图片输出删除 u 1111 2222 | SUPERUSER        | 对某几个人关闭输出，`u`作为参数,<br />不带参数默认为`g`，后跟QQ号用空格分隔    | 输出删除 u 1111 2222       |


### 流程

1. 机器人启动时，对配置文件进行检测，不存在则创建
2. 消息预处理:
   - 如果消息中包含`-p`，则对此条消息进行图片输出（无论全局、群、用户是否开启），将`-p`去除后，再进行chatGPT请求
   - 如果消息中不包含`-p`则不处理
3. 请求chatGPT，得到结果后，对结果进行处理：
   - 如果消息中代码块符号` ``` `不完整，则补全（chatGPT有时返回不完整）
   - 根据配置发送图片或者文本
4. `-p` 或 全局开启，优先级最高，二者之一存在，则对此条消息进行图片输出
5. 若非上述二者：
   - 群聊
     - 如果群开启图片输出，则返回图片
     - 如果全局关闭但用户开启，则返回图片
   - 私聊
     - 如果用户开启图片输出，则返回图片

# Todo

- [X]  返回值渲染为图片
