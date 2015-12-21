# 消息推送服务总览

消息推送，使得开发者可以即时地向其应用程序的用户推送通知或者消息，与用户保持互动，从而有效地提高留存率，提升用户体验。平台提供整合了 Android 推送、iOS 推送、Windows Phone 推送和 Web 网页推送的统一推送服务。

除了 iOS、Android SDK 做推送服务之外，你还可以通过我们的 REST API 来发送推送请求。

## 文档贡献

我们欢迎和鼓励大家对本文档的不足提出修改建议。请访问我们的 [Github 文档仓库](https://github.com/leancloud/docs) 来提交 Pull Request。

## 基本概念

### Installation

Installation 表示一个允许推送的设备的唯一标示，对应 [数据管理](/data.html?appid={{appid}}) 平台中的 `_Installation` 表。它就是一个普通的对象，主要属性包括:

名称|适用平台|描述
---|---|---
badge|iOS|呈现在应用图标右上角的红色圆形数字提示，例如待更新的应用数、未读信息数目等。
channels| |设备订阅的频道
deviceProfile|iOS|在应用有多个推送证书的场景下，deviceProfile 用于指定该设备对应的证书名。
deviceToken|iOS|APNS 推送的唯一标识符
deviceType| |设备类型，目前支持 "ios"、"android"、"wp"、"web"。
ID|Windows Phone|仅对微软平台的设备（微软平板和手机）有效
installationId|Android|LeanCloud SDK 为每个 Android 设备产生的唯一标识符
subscriptionUri|Windows Phone|MPNS（微软推送服务）推送的通道
timeZone| |设备设定的时区

增删改查设备，请看后面的 SDK 说明和 [REST API](#使用_REST_API_推送消息) 一节。


### Notification

对应 `_Notification` 表，表示一条推送消息，它包括下列属性：

名称|适用平台|描述
---|---|---
data| |本次推送的消息内容，JSON 对象。
invalidTokens|iOS|本次推送遇到多少次由 APNS 返回的 [INVALID TOKEN](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/CommunicatingWIthAPS.html#//apple_ref/doc/uid/TP40008194-CH101-SW12) 错误。**如果这个数字过大，请留意证书是否正常。**
prod|iOS|使用什么环境证书。**dev** 表示开发证书，**prod** 表示生产环境证书。
status| |本次推送的状态，**in queue** 表示仍然在队列，**done** 表示完成，**schedule** 表示定时推送任务等待触发中。
subscribers| |本次推送的接收设备数目，注意这个数字并不表示实际送达，而是说当时符合查询条件的、并且已经推送给 Apple APNS 或者 Android Push Server 的总设备数。
where| |本次推送查询 _Installation 表的条件，符合这些查询条件的设备将接收本条推送消息。

如何发送消息也请看下面的详细指南。

推送本质上是根据一个 query 条件来查询 _Installation 表里符合条件的设备，然后将消息推送给设备。因为 `_Installation` 是一个可以完全自定义属性的 Key-Value Object，因此可以实现各种复杂条件推送，例如频道订阅、地理位置信息推送、特定用户推送等。

这里重点说明一下 **subscribers** 这个属性，它的值表示我们查找出来的符合条件的 Installation 数量。当值为 0 时，表示没有找到任何符合目标条件的设备，这时自然所有人都收不到推送通知；当值不为 0 时，仅仅说明找到了这么多符合条件的设备，但并不保证这些设备都能收到推送通知。

## iOS 消息推送
请阅读 [iOS 推送开发文档](./ios_push_guide.html)。

## Android 消息推送
请阅读 [Android 推送开发文档](./android_push_guide.html)。

## Windows Phone 消息推送
请阅读 [Windows Phone 推送开发文档](./dotnet_push_guide.html)。

## 云代码和 JavaScript 创建推送
请阅读 [JavaScript SDK 指南 - Push 通知](./js_guide.html#Push_通知)。
我们还提供单独的 [JavaScript 推送客户端](https://github.com/leancloud/js-push-sdk/) 用于在网页中收发推送。

## 使用 REST API 推送消息

### Installation

当 App 安装到用户设备后，如果要使用消息推送功能，LeanCloud SDK 会自动生成一个 Installation 对象。Installation 对象包含了推送所需要的所有信息。你可以使用 REST API，通过 Installation 对象进行消息推送。

#### 保存 Installation

##### 保存 iOS 设备的 DeviceToken

iOS 设备通常使用 DeviceToken 来唯一标识一台设备。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "ios",
        "deviceToken": "abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789",
        "channels": [
          "public", "protected", "private"
        ]
      }' \
  https://leancloud.cn/1.1/installations
```

##### 保存 Android 设备的 installationId

对于 Android 设备，LeanCloud SDK 会自动生成 uuid 作为 installationId 保存到 LeanCloud。使用 REST API 来保存  installationId 的方法如下：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "deviceType": "android",
        "installationId": "12345678-4312-1234-1234-1234567890ab",
        "channels": [
          "public", "protected", "private"
        ]
      }' \
  https://leancloud.cn/1.1/installations
```

`installationId` 必须在应用内唯一。

##### 订阅和退订频道

通过设置 `channels` 属性来订阅某个推送频道：

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels": [
          "customer"
        ]
      }' \
  https://leancloud.cn/1.1/installations/mrmBZvsErB
```

退订一个频道：

```
curl -X PUT \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels": {
           "__op":"Remove",
           "objects":["customer"]
        }
       }' \
  https://leancloud.cn/1.1/installations/mrmBZvsErB
```

`channels` 本质上是数组属性，因此可以使用标准 [REST API](./rest_api.html#数组) 操作。

#### 自定义属性

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "userObjectId": "<用户的 objectId>"
      }' \
  https://leancloud.cn/1.1/installations/mrmBZvsErB
```

### 推送消息

通过 `POST /1.1/push` 来推送消息给设备，`push` 接口支持下列属性：

名称|描述
---|---
channels|推送给哪些频道，将作为条件加入 where 对象。
data|推送的内容数据，JSON 对象，请参考 [消息内容](#消息内容_Data)。
expiration_interval|消息过期的相对时间，从调用 API 的时间开始算起，单位是「秒」。
expiration_time|消息过期的绝对日期时间
prod|**仅对 iOS 有效**。设置使用开发证书（**dev**）还是生产证书（**prod**）。当设备设置了 deviceProfile 时我们优先按照 deviceProfile 指定的证书推送。
push_time|定期推送时间
where|检索 _Installation 表使用的查询条件，JSON 对象。

注意：为防止由于大量证书错误产生的性能问题，我们对使用**开发证书**的推送做了设备数量的限制。
目前当您使用开发证书推送时，至多可以一次性给 20000 个设备进行推送。如果满足您推送条件的设备超过 20000 个，
系统会拒绝此次推送，并在推送记录页面体现。因此，在开发过程中使用开发证书推送时，请合理设置您的推送条件。

>我们建议给 iOS 设备的推送都设置过期时间，才能保证推送的当时如果用户设置了飞行模式，在关闭飞行模式之后可以收到推送消息，可以参考 [Stackoverflow 帖子](http://stackoverflow.com/questions/24026544/push-notification-is-not-being-delivered-when-iphone-comes-back-online)。

#### 消息内容 Data

对于 iOS 设备，`data` 属性可以是：

```
{
  "data": {
   "alert":             "消息内容",
   "category":          "通知分类名称",
   "badge":             "未读消息数目，应用图标边上的小红点数字，可以是数字，也可以设置为 Increment 这个字符串（大小写敏感）",
   "sound":             "声音文件名，前提在应用里存在",
   "content-available": "如果你在使用 Newsstand, 设置为1来开始一次后台下载"
  }
}
```

并且 iOS 设备支持 `alert` 本地化消息推送：

```
{
  "data":{
    "alert": {
      "title":           "标题",
      "title-loc-key":   "",
      "body":            "消息内容",
      "action-loc-key":  "",
      "loc-key":         "",
      "loc-args":        [""],
      "launch-image":    ""
     }
   }
}
```

详情参考 [Apple 文档](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html)。

如果是 Android 设备，默认的消息栏通知 `data` 支持下列属性：

```
{
  "data":{
    "alert":      "消息内容",
    "title":      "显示在通知栏的标题"
  }
}
```

如果自定义 Receiver，需要设置 `action`，当然也可以自己加属性了:

```
{
  "data":{
    "alert":      "消息内容",
    "title":      "显示在通知栏的标题",
    "action":     "com.your_company.push",
    "fromUserId": "自定义属性"
  }
}
```

Windows Phone 设备类似，也支持 `title` 和 `alert`，同时支持 `wp-param` 用于定义打开通知的时候打开的是哪个 Page：

```
{
  "data":{
    "alert":      "消息内容",
    "title":      "显示在通知栏的标题",
    "wp-param":   "/chat.xaml?NavigatedFrom=Toast Notification"
  }
}
```

但是如果想一次 push 调用**推送不同的数据给不同类型的设备**， `data` 属性同时支持设定设备特定消息，例如：

```
{
  "data":{
    "ios": {
      "alert":             "消息内容",
      "badge":             "未读消息数目，应用图标边上的小红点数字，可以是数字，也可以设置为 Increment 这个字符串（大小写敏感）",
      "sound":             "声音文件名，前提在应用里存在",
      "content-available": "如果你在使用 Newsstand, 设置为 1 来开始一次后台下载"
    },
    "android": {
      "alert":             "消息内容",
      "title":             "显示在通知栏的标题",
      "action":            "com.your_company.push",
      "fromUserId":        "自定义属性"
    },
    "wp":{
      "alert":             "消息内容",
      "title":             "显示在通知栏的标题",
      "wp-param":          "/chat.xaml?NavigatedFrom=Toast Notification"
    }
  }
}
```

#### iOS 测试和生产证书区分

我们现在支持上传两个环境的 iOS 推送证书：测试和生产环境，你可以通过设定 `prod` 属性来指定使用哪个环境证书。

```
{
  "prod": "dev",
  "data": {
    "alert": "test"
  }
}
```

如果是 `dev` 值就表示使用开发证书，`prod` 值表示使用生产证书，默认使用**生产证书**。注意，当设备设置了 deviceProfile 时我们优先按照 deviceProfile 指定的证书推送。

#### 推送查询条件

_Installation 表中的所有属性，无论是内置的还是自定义的，都可以作为查询条件通过 where 来指定，并且支持 [REST API](./rest_api.html#查询) 定义的各种复杂查询。

后文会举一些例子，更多例子请参考 [REST API](./rest_api.html#查询) 查询文档。

#### expiration_time、expiration_interval 和 push_time

`expiration_time` 属性用于指定消息的过期时间，如果客户端收到消息的时间超过这个绝对时间，那么消息将不显示给用户。`expiration_time` 是一个 UTC 时间的字符串，格式为 `YYYY-MM-DDTHH:MM:SS.MMMMZ`。

```
{
      "expiration_time": "2015-11-11T00:51:13Z",
      "data": {
        "alert": "过期时间为北京时间 11 月 11 号 8:51。"
      }
}
```

`expiration_interval` 也可以用于指定过期时间，不过这是一个相对时间，以*秒为单位*，从 API 调用时间点开始计算起：

```
{
      "expiration_interval": "86400",
      "data": {
        "alert": "收到 Push API 调用的 24 小时后过期。"
      }
}
```

`push_time` 是定期推送的时间，格式为 `YYYY-MM-DDTHH:MM:SS.MMMMZ` 的 UTC 时间，也可以结合 `expiration_interval`设定过期时间：

```
{
      "push_time":           "2015-11-11T00:51:13.931ZZ",
      "expiration_interval": "86400",
      "data": {
        "alert": "北京时间 11 月 11 号 8:51 发送这条推送，24 小时后过期"
      }
}
```

下面是一些推送的例子

#### 推送给所有的设备
```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "data": {
          "alert": "LeanCloud 向您问好！"
        }
      }' \
  https://leancloud.cn/1.1/push
```

#### 发送给特定的用户

* 发送给 public 频道的用户

```sh
curl -X POST \
-H "X-LC-Id: {{appid}}"          \
-H "X-LC-Key: {{appkey}}"        \
-H "Content-Type: application/json" \
-d '{
      "where":{
        "channels":
          {"$regex":"\\Qpublic\\E"}
      },
      "data": {
        "alert": "LeanCloud 向您问好！"
      }
    }' \
https://leancloud.cn/1.1/push
```

或者更简便的方式

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels":[ "public"],
        "data": {
          "alert": "LeanCloud 向您问好！"
        }
      }' \
  https://leancloud.cn/1.1/push

```

* 发送给某个 installation id 的用户

```sh
curl -X POST \
-H "X-LC-Id: {{appid}}"          \
-H "X-LC-Key: {{appkey}}"        \
-H "Content-Type: application/json" \
-d '{
      "where":{
          "installationId":"57234d4c-752f-4e78-81ad-a6d14048020d"
          },
      "data": {
        "alert": "LeanCloud 向您问好！"
      }
    }' \
https://leancloud.cn/1.1/push
```

* 推送给不活跃的用户

```sh
curl -X POST \
-H "X-LC-Id: {{appid}}"          \
-H "X-LC-Key: {{appkey}}"        \
-H "Content-Type: application/json" \
-d '{
      "where":{
          "updatedAt":{
              "$lt":{"__type":"Date","iso":"2015-06-29T11:33:53.323Z"}
            }
      },
      "data": {
          "alert": "LeanCloud 向您问好！"
      }
    }' \
https://leancloud.cn/1.1/push
```

* 根据查询条件做推送：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "inStock": true
        },
        "data": {
          "alert": "您关注的商品已经到货，请尽快购买。"
        }
      }' \
  https://leancloud.cn/1.1/push
```

用 where 查询的都是 _Installations 表中的属性。这里假设该表存储了 inStock 的布尔属性。

* 根据地理信息位置做推送：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "owner": {
            "$inQuery": {
              "location": {
                "$nearSphere": {
                  "__type": "GeoPoint",
                  "latitude": 30.0,
                  "longitude": -20.0
                },
                "$maxDistanceInMiles": 10.0
              }
            }
          }
        },
        "data": {
          "alert": "北京明日最高气温 40 摄氏度。"
        }
      }' \
  https://leancloud.cn/1.1/push
```

上面的例子假设 installation 有个 owner 属性指向 _User 表的记录，并且用户有个 location 属性是 GeoPoint 类型，我们就可以根据地理信息位置做推送。

#### 使用 CQL 查询推送

上述 `where` 的查询条件都可以使用 [CQL](./cql_guide.html) 查询替代，例如查询某个设备推送：

```sh
curl -X POST \
-H "X-LC-Id: {{appid}}"          \
-H "X-LC-Key: {{appkey}}"        \
-H "Content-Type: application/json" \
-d '{
      "cql":"select * from _Installation where installationId='xxxxxxxxxxxxx'",
      "data": {
        "alert": "LeanCloud 向您问好！"
      }
    }' \
https://leancloud.cn/1.1/push
```

#### 推送消息属性

##### 消息过期

 过期时间，可以是绝对时间：
```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "expiration_time": "2015-10-07T00:51:13Z",
        "data": {
          "alert": "您的优惠券将于 10 月 7 日到期。"
        }
      }' \
  https://leancloud.cn/1.1/push
```

也可以是相对时间（从推送 API 调用开始算起，结合 push_time 做定期推送）:
```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "push_time": "2015-06-28T00:51:13.931Z",
        "expiration_interval": 518400,
        "data": {
          "alert": "您未使用的代金券将于 2015 年 7 月 4 日过期。"
        }
      }' \
  https://leancloud.cn/1.1/push
```

##### 定制消息属性：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}"          \
  -H "X-LC-Key: {{appkey}}"        \
  -H "Content-Type: application/json" \
  -d '{
        "channels": [
          "employee"
        ],
        "data": {
          "alert": "本周五下午三点组织大家看电影《大圣归来》。",
          "badge": "Increment",
          "sound": "cheering.caf",
          "title": "员工福利"
        }
      }' \
  https://leancloud.cn/1.1/push
```


## Installation 自动过期和清理

每当用户打开应用，我们都会更新该设备的 _Installation 表中的 `updatedAt` 时间戳。用户如果长期没有更新 _Installation 的 `updatedAt` 时间戳，也就意味着该用户长期没有打开过应用。当超过 360 天没有打开过应用时，我们会将这个用户在 _Installation 表中的记录删除。不过请不要担心，当用户再次打开应用的时候，仍然会自动创建一个新的 Installation 用于推送。

对于 iOS 设备，除了上述过期机制外还多拥有一套过期机制。当我们根据 Apple 推送服务的反馈获取到某设备的 deviceToken 已过期时，我们也会将这个设备在 _Installation 表中的信息删除，并标记这个已过期的 deviceToken 为无效，丢弃后续所有发送到该 deviceToken 的消息。

## 推送问题排查

推送因为环节较多，跟设备和网络都相关，并且调用都是异步化，因此问题比较难以查找，这里提供一些有用助于排查消息推送问题的技巧。

### 推送结果查询

所有经过 `/push` 接口发出的消息的都可以在控制台的存储里的 _Notification 表看到。每次调用 `/push` 都将产生一条新的 `_Notification` 对象表示一次推送。该表的各属性含义请参考 [Notification 表详解](#Notification) 。

`/push` 接口会返回新建的 `_Notification` 对象的 `objectId`，你就可以在这张表里查找消息推送的结果。


### iOS 推送排查建议

一些建议和提示：

* 请确保在项目 Info.plist 文件中使用了正确的 **Bundle Identifier**。
* 请确保在 **Project** > **Build Settings** 设置了正确的 **provisioning profile**。
* 尝试 clean 项目并重启 Xcode
* 尝试到 [Apple Developer](https://developer.apple.com/account/overview.action) 重新生成 **provisioning profile**，修改 Apple ID，再改回来，然后重新生成。你需要重新安装 provisioning profile，并在 **Project** > **Build Settings** 里重新设置。
* 打开 XCode Organizer，从电脑和 iOS 设备里删除所有过期和不用的 provisioning profile。
* 如果编译和运行都没有问题，但是你仍然收不到推送，请确保你的应用打开了接收推送权限，在 iOS 设备的 **设置** > **通知** > **你的应用** 里确认。
* 如果权限也没有问题，请确保使用了正确的 **provisioning profile**。 打包你的应用。如果你上传了开发证书并使用开发证书推送，那么必须使用 **Development Provisioning Profile** 构建你的应用。如果你上传了生产证书，并且使用生产证书推送，请确保你的应用使用 **Distribution Provisioning Profile** 签名打包。**Ad Hoc** 和 **App Store Distribution Provisioning Profile** 都可以接收使用生产证书发送的消息。
* 当在一个已经存在的 Apple ID 上启用推送，请记得重新生成 **provisioning profile**，并到 XCode Organizer 更新。
* 生产环境的推送证书必须在提交到 App Store 之前启用推送并生成，否则你需要重新提交 App Store。
* 请在提交 App Store 之前，使用 Ad Hoc Profile 测试生产环境推送，这种情况下的配置最接近于 App Store。
* 检查 _Notifcation 表的 `subscribers` 和 `status`，确认推送状态和接收设备数目正常。


### Android 排查建议

* 请确保设备正确调用了 `AVInstallation` 保存了设备信息到 _Installation 表。
* 可以在控制台的 **消息** > **推送** > **帮助** 根据 `installationId` 查询设备是否在线。
* 请确保 `com.avos.avoscloud.PushService` 添加到 AndroidManifest.xml 文件中。
* 如果使用自定义 Receiver，请确保在 AndroidManifest.xml 中声明你的 Receiver，并且保证 data 里的 action 一致。
