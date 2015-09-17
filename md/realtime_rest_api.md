#实时通信 REST API

## 对话数据操作

你可以通过 REST API 对对话（相应的聊天室、群组或单聊等）进行操作，例如提前创建聊天室，关联聊天室到其他数据实体。LeanCloud 实时通信系统采用透明的设计，对话数据在 LeanCloud 系统中是普通的数据表，表名为 **_Conversation**，你可以直接调用 [数据存储相关的 API 进行数据操作](./rest_api.html#对象-1)。**_Conversation** 表 包含一些内置的关键字段定义了对话的属性、成员等，你可以在 [实时通信概览 - 对话](./realtime_v2.html#对话_Conversation_) 了解。

### 创建一个对话

创建一个对话即在 _Conversation 表中创建一条记录。对于没有使用过实时通信服务的新用户， _Conversation 表会在第一条记录创建后出现。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"name":"My Private Room","m": ["BillGates", "SteveJobs"]}' \
  https://api.leancloud.cn/1.1/classes/_Conversation
```

上面的例子会创建一个最简单的对话，包括两个 client ID 为 BillGates 和 SteveJobs 的初始成员。对话创建成功会返回 objectId，即实时通信中的对话 ID，客户端就可以通过这个 ID 发送消息了。

常见的开放聊天室的场景，需要通过 REST API 预先创建聊天室，并把对话 ID 与应用内的某个对象关联（如视频、比赛等）。创建开放聊天室只需要包含一个 **tr** 参数，设置为 true 即可。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"name": "OpenConf","tr": true}' \
  https://api.leancloud.cn/1.1/classes/_Conversation
```

### 增删对话成员

你可以通过 REST API 操作对话数据的 **m** 字段来实现成员的增删。m  字段是一个数组字段，使用数组的操作符进行修改。

增加一个 client id 为 LarryPage 的用户到已有（以对话 id 5552c0c6e4b0846760927d5a 为例）对话：

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"m": {"__op":"AddUnique","objects":["LarryPage"]}}' \
  https://api.leancloud.cn/1.1/classes/_Conversation/5552c0c6e4b0846760927d5a
```

将不再活跃的 SteveJobs 清除出对话（以对话 id 5552c0c6e4b0846760927d5a 为例）：

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"m": {"__op":"Remove","objects":["SteveJobs"]}}' \
  https://api.leancloud.cn/1.1/classes/_Conversation/5552c0c6e4b0846760927d5a
```

对 _Conversation 表的查询等其他操作与普通表完全一致，可以参考 [REST API - 查询](./rest_api.html#查询) 的相应说明，这里不再赘述。

##获取聊天记录

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://leancloud.cn/1.1/rtm/messages/logs
```
###获取某个对话的聊天记录

参数 | 约束 | 说明
--- | --- | ---
convid | **必须** | 对话 id
max_ts | 可选 | 查询起始的时间戳，返回小于这个时间(不包含)的记录。默认是当前时间。
msgid | 可选 | 起始的消息 id，与 max_ts 一起作为查询的起点。
limit | 可选 | 返回条数限制，可选，默认 100 条，最大 1000 条。
peerid | 可选 | 查看者id（签名参数）
nonce | 可选 | 签名随机字符串（签名参数）
signature_ts | 可选 | 签名时间戳（签名参数）
signature | 可选 | 签名时间戳（签名参数）

为了保证获取聊天记录的安全性，可以开启签名认证（控制台 > **应用选项** > **聊天记录签名认证**）。了解更详细的签名规则请参考 [聊天签名方法](realtime.html#签名方法)。签名参数仅在开启应用选项后有效，如果没有开启选项，就不需要传签名参数。

签名采用 Hmac-sha1 算法，输出字节流的十六进制字符串 (hex dump)，签名的 key 必须是应用的 master key，签名的消息格式如下：

```
appid:peerid:convid:nonce:signature_ts
```

返回数据格式，JSON 数组，按消息记录从新到旧排序。

```json
[
  {
    "timestamp": 1408008498571,
    "conv-id":   "219946ef32e40c515d33ae6975a5c593",
    "data":      "今天天气不错！",
    "from":      "u111872755_9d0461adf9c267ae263b3742c60fa",
    "msg-id":    "vdkGm4dtRNmhQ5gqUTFBiA",
    "is-conv":   true,
    "is-room":   false,
    "to":        "5541c02ce4b0f83f4d44414e",
    "bin":       false,
    "from-ip":   "202.117.15.217"
  },
  ...
]
```

###获取某个用户发送的聊天记录
此接口仅支持 master key [鉴权认证](rest_api.html#更安全的鉴权方式)，建议仅在服务端使用。

参数 | 约束 | 说明
--- | --- | ---
from | **必须** | 发送人 id
max_ts | 可选 | 查询起始的时间戳，返回小于这个时间（不包含）的记录。默认是当前时间。
msgid | 可选 | 起始的消息 id，与 max_ts 一起作为查询的起点。
limit | 可选 | 返回条数限制，默认 100 条，最大 1000 条。

###获取应用的所有聊天记录
此接口仅支持 master key [鉴权认证](rest_api.html#更安全的鉴权方式)，建议仅在服务端使用

参数 | 约束 | 说明
--- | --- | ---
max_ts | 可选 | 查询起始的时间戳，返回小于这个时间（不包含）的记录。默认是 **当前时间**。
msgid | 可选 | 起始的消息 id，与 max_ts 一起作为查询的起点。
limit | 可选 | 返回条数限制，默认 100 条，最大 1000 条。

###聊天记录返回字段说明：

字段 | 说明
--- | ---
conv-id | 用于查询的对话 id
from | 消息来自 id
data | 消息内容
timestamp | 消息到达服务器的 Unix 时间戳（毫秒）
msg-id | 消息 id
is-conv | 是否是 v2 中对话模型的消息
is-room | 是否是早期版本中的群组消息
to | 早期版本中的收件人列表
bin | 是否是 BASE64 编码的二进制内容
from-ip | 消息的来源 IP

## 删除聊天记录
删除一条指定的聊天历史记录，必须采用 master key 授权，所以不建议在客户端使用此接口。

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://leancloud.cn/1.1/rtm/messages/logs?convid=219946ef32e40c515d33ae6975a5c593&msgid=PESlY&timestamp=1408008498571
```

参数 | 说明
--- | ---
convid | 对话 id
msgid | 消息 id
timestamp | 消息时间戳

### 构建对话 ID

实时通信中 convid 的构建规则为：目前版本中，convid 即对话 ID。

对早期版本来说：
* 对点对点通信，convid 为所有对话参与者的 peer id **排序** 后以半角冒号（:）分隔，做 md5 所得。如对话参与者 peer id 为 `u1234` 和 `u0988`，那么对话 ID 为 `bcd26a54e98687390b0abb4d83683d4b`。
* 对群组功能，convid 即群组 ID。

## 取未读消息数

您可以从服务器端通过 REST API 调用获取实时通信中，某个 Client ID 的未读消息数。

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://leancloud.cn/1.1/rtm/messages/unread/CLIENT_ID
```

返回：

```json
{"count": 4}
```

## 通过 REST API 发消息

我们目前提供 REST API 允许向一个已有对话发送消息。

**注意**，由于这个接口的管理性质，当你通过这个接口发送消息时，我们不会检查 **from_peer** 是否有权限给这个对话发送消息，而是统统放行，请谨慎使用这个接口。
如果你在应用中使用了我们内部定义的 [富媒体消息格式](./realtime_v2.html#消息_Message_)，在发送消息时 **message** 字段需要遵守一定的格式要求，下文 [富媒体消息格式说明](./realtime_rest_api.html#富媒体消息格式说明) 中将详细说明。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"from_peer": "1a", "message": "helloworld", "conv_id": "...", "transient": false}' \
  https://leancloud.cn/1.1/rtm/messages
```

参数 | 约束 | 说明
---|---|---
from_peer | |消息的发件人 id
conv_id | |发送到对话 id
transient | 可选|是否为暂态消息（**由于向后兼容的考虑，默认为 true**，请注意设置这个值。）
message || 消息内容（这里的消息内容的本质是字符串，但是我们对字符串内部的格式没有做限定，<br/>理论上开发者可以随意发送任意格式，只要大小不超过 5 KB 限制即可。）
no_sync | 可选|默认情况下消息会被同步给在线的 from_peer 用户的客户端，设置为 true 禁用此功能。
to_peers | 可选| 给系统对话发消息时指定具体的收消息者。
wait | 可选|同步调用等待返回，可以获得发送的报错信息。

返回说明：

默认情况下发送消息 API 使用异步的方式，调用后直接返回空结果 `{}`。当设置 wait 参数为真时，这个接口会等待后台发送结果至多 5 秒，如果遇到错误情况消息不能送达，将返回报错信息。
错误信息包含 code 与 reason 字段，其中 code 值遵守 [实时通信指南总览 - 服务器端错误码说明](realtime_v2.html#服务器端错误码说明)。
注意这个接口的发送成功并不表示当时客户端已经收到，但可以确认消息至少已经发送到客户端队列，即使当时客户端离线也会在下次上线时收到。

### 给系统对话发消息

利用 REST API 给通过系统对话给用户发消息时，除了 conv_id 需要设置为对应系统对话的 ID 以外，还需要设置 `to_peers`（数组）指定实际接收消息的 Client ID。

目前你可以在一次调用中传入至多 20 个 Client ID。

### 富媒体消息格式说明
富媒体消息的参数格式相对于普通文本来说，仅仅是将 message 参数换成了一个 JSON **字符串**。

>注意：由于 LeanCloud 实时通信中所有的消息都是文本，所以这里发送 JSON 结构时**需要首先序列化成字符串**。

#### 文本消息

```
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"from_peer": "1a", "message": "{\"_lctype\":-1,\"_lctext\":\"这是一个纯文本消息\",\"_lcattrs\":{\"a\":\"_lcattrs 是用来存储用户自定义的一些键值对\"}}", "conv_id": "...", "transient": false}' \
  https://leancloud.cn/1.1/rtm/messages
```

发送文本消息可以按照以上的格式进行，参数说明：

参数 |约束| 说明
--- |---|---
_lctype | |富媒体消息的类型<br/><br/><pre>文本消息 -1<br/>图像　　 -2<br/>音频　　 -3<br/>视频　　 -4<br/>地理位置 -5<br/>通用文件 -6</pre>
_lctext | |富媒体消息的文　字说明
_lcattrs | |JSON 字符串，用来给开发者存储自定义属性。
_lcfile | |如果是包含了文件（图像，音频，视频，通用文件）的消息 ，<br/>_lcfile 就包含了它的文件实体的相关信息。
url | |文件在上传之后的物理地址
objId |可选 |文件对应的在 _File 表里面的 objectId
metaData | 可选|文件的元数据

**以上参数针对所有富媒体消息都有效**。

#### 图像消息

在新版本的聊天中，支持了内建的富媒体消息格式，以下针对整个消息体 JSON 格式化之后的参数说明，例如如下的图像消息：

```
  {
    "_lctype":    -2,                    //必要参数
    "_lctext":    "图像的文字说明",
    "_lcattrs": {
      "a":        "_lcattrs 是用来存储用户自定义的一些键值对",
      "b":        true,
      "c":        12
    },
    "_lcfile": {
      "url":      "http://ac-p2bpmgci.clouddn.com/246b8acc-2e12-4a9d-a255-8d17a3059d25", //必要参数
      "objId":    "54699d87e4b0a56c64f470a4//文件对应的AVFile.objectId",
      "metaData": {
        "name":   "IMG_20141223.jpeg",   //图像的名称
        "format": "png",                 //图像的格式
        "height": 768,                   //单位：像素
        "width":  1024,                  //单位：像素
        "size":   18                     //单位：b
      }
    }
  }
```


以上是完整版的格式，如果想简单的发送一个 URL 可以参照以下格式：

```
  {
    "_lctype": -2,
    "_lcfile": {
      "url":   "http://ac-p2bpmgci.clouddn.com/246b8acc-2e12-4a9d-a255-8d17a3059d25"
    }
  }
```

#### 音频消息

与图像类似，音频格式的完整格式如下：

```
  {
    "_lctype":      -3,
    "_lctext":      "这是一个音频消息",
    "_lcattrs": {
      "a":          "_lcattrs 是用来存储用户自定义的一些键值对"
    },
    "_lcfile": {
      "url":        "http://ac-p2bpmgci.clouddn.com/246b8acc-2e12-4a9d-a255-8d17a3059d25",
      "objId":      "54699d87e4b0a56c64f470a4//文件对应的AVFile.objectId",
      "metaData": {
        "name":     "我的滑板鞋.wav",
        "format":   "wav",
        "duration": 26,    //单位：秒
        "size":     2738   //单位：b
      }
    }
  }
```

简略版：

```
  {
    "_lctype": -3,
    "_lcfile": {
      "url":   "http://www.somemusic.com/x.mp3"
    }
  }
```

#### 视频消息

完整版：

```
  {
    "_lctype":      -4,
    "_lctext":      "这是一个视频消息",
    "_lcattrs": {
      "a":          "_lcattrs 是用来存储用户自定义的一些键值对"
    },
    "_lcfile": {
      "url":        "http://ac-p2bpmgci.clouddn.com/99de0f45-171c-4fdd-82b8-1877b29bdd12",
      "objId":      "54699d87e4b0a56c64f470a4", //文件对应的 AVFile.objectId
      "metaData": {
        "name":     "录制的视频.mov",
        "format":   "avi",
        "duration": 168,      //单位：秒
        "size":     18689     //单位：b
      }
    }
  }
```

简略版：

```
  {
    "_lctype": -4,
    "_lcfile": {
      "url":   "http://www.somevideo.com/Y.flv"
    }
  }
```

#### 通用文件消息

```
  {
    "_lctype": -6,
    "_lctext": "这是一个普通文件类型",
    "_lcattrs": {
      "a":     "_lcattrs 是用来存储用户自定义的一些键值对"
    },
    "_lcfile": {
      "url":   "http://www.somefile.com/jianli.doc",
      "name":  "我的简历.doc",
      "size":  18689          //单位：b
    }
  }
```

简略版：

```
  {
    "_lctype": -6,
    "_lcfile": {
      "url":   "http://www.somefile.com/jianli.doc",
      "name":  "我的简历.doc"
    }
  }
```

#### 地理位置消息

```
  {
    "_lctype":     -5,
    "_lctext":     "这是一个地理位置消息",
    "_lcattrs": {
      "a":         "_lcattrs 是用来存储用户自定义的一些键值对"
    },
    "_lcloc": {
      "longitude": 23.2,
      "latitude":  45.2
    }
  }
```

简略版：

```
  {
    "_lctype":     -5,
    "_lcloc": {
      "longitude": 23.2,
      "latitude":  45.2
    }
  }
```

## 获取暂态对话的在线人数

你可以通过这个 API 获得暂态对话的在线人数。由于暂态对话没有成员列表支持，所以通常使用这个 API 获得当前的在线人数。

参数 | 说明
--- | ---
gid | 暂态对话的 id

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://leancloud.cn/1.1/rtm/transient_group/onlines?gid=...
```

返回：

```json
{"result": 0}
```

这个 API 也可以用于获取早期版本开放群组的在线人数。


## 查询在线状态

在线状态查询 API 可以一次至多查询 20 个 Client ID 当前是否在线：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"peers": ["7u", "8b", "3h", ...]}' \
  https://leancloud.cn/1.1/rtm/online
```

参数 | 说明
--- | ---
peers | 要查询的 ID 列表

返回：

在线的 ID 列表

```json
{"results":["7u"]}
```
