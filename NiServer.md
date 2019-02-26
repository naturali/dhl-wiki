# 对话流 Restful Api 文档

## 概要介绍

对话流 Restful Api 是对话流系统的用户在server端调用对话流能力的接口，如需在客户端使用对话流系统功能请参考sdk文档

## 准备工作

**1、服务地址**

https://<span></span>dhlmixer.naturali.io

**2、申请Api Key**

使用对话流 Restful Api 之前请先确保在对话流平台注册并申请了Api Key

**3、数据格式**

对话流 Restful Api 的数据格式采用了protobuf，请下载proto文件并生成相应语言的代码以供使用

共有以下五个proto文件:

[dhlmixer.proto](#dhlmixer)

[dhl.proto](#dhl)

[dynamic_entity.proto](#dynamic_entity)

[filled_attribute.proto](#filled_attribute)

[dhl_response.proto](#dhl_response)

使用golang语言开发时生成代码的命令如下：

```protoc -I ./ --go_out=plugins=grpc:./ dhlmixer.proto dhl.proto filled_attribute.proto dynamic_entity.proto dhl_response.proto```

**4、platform type**

使用对话流 Restful Api，请在参数中存在platform_type的时候传入参数值 dhl_restful，例如[KerfuMessage](#KerfuMessage)中的platform_type

## Api概览

| API | 描述 |
| :------ | :------ |
| [QueryWithKerfuMessage](#QueryWithKerfuMessage) | 发送信息给对话流系统 |
| [GetKerfuMessages](#GetKerfuMessages) | 获取信息历史记录 |
| [EventConnection](#EventConnection) | websocket长连接，用于发送和接收控制信息 |
| [Speech](#Speech) | 语音识别 |
| [GetAccessToken](#GetAccessToken) | 获取访问对话流系统的access token |

一个典型的对话流客户端应该以下述流程运行：

**1、调用 [GetAccessToken](#GetAccessToken) 获取token**

**2、调用 [QueryWithKerfuMessage](#QueryWithKerfuMessage) 向对话流系统发起请求，并解析响应**

**3、调用 [EventConnection](#EventConnection) 发起长连接监听系统推送的事件**

**4、收到新消息通知时调用 [GetKerfuMessages](#GetKerfuMessages) 获取对话流系统主动推送的最新消息**

## 访问控制

调用对话流 Restful Api 需要在每个请求头中携带被授权的 access token，获取token的方法详见 Api [GetAccessToken](#GetAccessToken) 的说明。
access token 为JWT格式，每一个新获取的token的有效期为24小时，过期后无法使用，需重新获取token。
使用token的方式为在请求头中添加Authorization，例如:

```Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhcHAiOiJOYXR1cmFsaS1CRCIsImV4cCI6MTU0NzM2Njc2Miwia2V5IjoidGYyMmI5WUVqMGN3NHBCTUVUSVE4OHR6RXZvcG45RU8ifQ.hjrzquJ04oAvQVwyUK_rSQSZ34fW1r7i0-7sTvxucMw```

http请求需在每一个请求中添加Authorization请求头，websocket请求需要在建立连接的请求中添加上述请求头，如果 access token 验证未通过会返回401错误

## Api请求语法及参数返回值说明

参数列表以及返回值均为protobuf，可以参考proto文件

**1、<span id="QueryWithKerfuMessage">QueryWithKerfuMessage</span>**

向对话流发送信息，并获取对话流系统的回复

请求语法：

```
POST /v1/kerfu_messages HTTP/1.1
Authorization: $access_token$
Content-Type: application/protobuf
```

请求参数: [KerfuMessage](#KerfuMessage)

返回值: [KerfuResponse](#KerfuResponse)

**2、<span id="GetKerfuMessages">GetKerfuMessages</span>**

获取信息历史记录，可以用来通过系统推送的MessageId获取系统主动推送的信息内容

请求语法：

```
GET /v1/message_history?session_id=$session_id$&message_id=$message_id$&before=$before$&after=$after$&page=$page$&page_size=$page_size$&message_type=$message_type$ HTTP/1.1
Authorization: $access_token$
```

请求参数:

| 名称 | 类型 | 是否必须 | 描述 |
| :------ | :------ | :------ | :------ |
| session_id | int32 | 是 | 需要查询消息的session的id |
| message_id | int64 | 否 | 需要获取的消息的id |
| before | int32 | 否 | 获取该消息id之前的若干消息 |
| after | int32 | 否 | 获取该消息id之后的若干消息 |
| page | int32 | 否 | 获取消息分页, 默认值为0 |
| page_size | int32 | 否 | 获取消息的每页的消息数，默认值为20 |
| message_type | int32 | 否 | 过滤所获取的消息类型 |

\* message_id, before, after 三个参数必选其一

\* message_type为int32类型，具体类型参考proto文件

返回值: [KerfuMessageList](#KerfuMessageList)

**3、<span id="EventConnection">EventConnection</span>**

websocket长连接，用来获取和发送一些控制信号

请求语法：

建立连接

```
GET /v1/event_action HTTP/1.1
Authorization: $access_token$
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Protocol: event_action
```

收发数据

发送: [KerfuAction](#KerfuAction) (message_type: BinaryMessage)

接收: [KerfuEvent](#KerfuEvent)

**4、<span id="Speech">Speech</span>**

websocket长连接，可以用来上传语音数据(pcm/mp3/flac)获取语音识别结果，也可以使用语音数据调用对话流系统并获得对话流系统的响应

请求语法：

建立连接

```
GET /v1/speech HTTP/1.1
Authorization: $access_token$
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Protocol: speech
```

收发数据

发送: [SpeechData](#SpeechData) (message_type: BinaryMessage)

接收: [SpeechResult](#SpeechResult)

**5、<span id="GetAccessToken">GetAccessToken</span>**

通过对话流系统分配的 app_key 和 app_secret 来获取访问其他api所需的access token

请求语法：

```
POST /v1/access_token HTTP/1.1
Content-Type: application/protobuf
```

请求参数: [AuthenticationParams](#AuthenticationParams)

返回值: [AccessToken](#AccessToken)

## proto数据结构说明

**1、<span id="KerfuMessage">KerfuMessage</span>**

KerfuMessage是发送给对话流系统的消息以及对话流系统的响应消息的类型

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| message_id | int64 | 消息id |
| session_id | int32 | 消息所属session的id |
| message_type | [KerfuMessageType](#KerfuMessageType) | 消息类型 |
| platform_type | string | 用户所属平台标识 |
| app_id | string | 用户所属org标识 |
| user_id | int64 | 用户唯一id |
| user_name | int32 | 用户名，用于显示 |
| agent_id | string | 用户对话的agent的唯一标识 |
| agent_name | string | 用户对话的agent显示名 |
| timestamp | string | 时间戳 |
| request | [DHLMixerRequestData](#DHLMixerRequestData) | 用户发送的消息数据，仅出现于向对话流系统发起的请求消息中 |
| response | [DHLMixerResponseData](#DHLMixerResponseData) | 对话流发送的响应数据，仅出现在对话流系统的响应消息中 |

\* request 和 response 只会出现其中之一

**2、<span id="KerfuResponse">KerfuResponse</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| ack | [KerfuMessageAck](#KerfuMessageAck) | 发送消息的ack |
| messages | repeated [KerfuMessage](#KerfuMessage) | 响应消息列表 |

**3、<span id="KerfuMessageList">KerfuMessageList</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| messages | repeated [KerfuMessage](#KerfuMessage) | 消息列表 |

**4、<span id="KerfuAction">KerfuAction</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| seq | string | 发送消息的ack |
| action | [Action](#Action) | action类型 |
| authentication_data | [KerfuAuthenticationData](#KerfuAuthenticationData) | 认证数据 |
| end_conversation_data | [EndConversationData](#EndConversationData) | 结束对话数据 |

\* authentication_data 和 end_conversation_data 只会出现其中之一

**5、<span id="KerfuEvent">KerfuEvent</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| event | [Event](#Event) | 事件类型 |
| reply_event_data | [KerfuReplyEventData](#KerfuReplyEventData) | 对于action的响应的数据 |
| message_posted_data | [KerfuMessagePostedEventData](#KerfuMessagePostedEventData) | 新消息提醒，收到这个数据需要去主动拉取最新消息 |

\* reply_event_data 和 message_posted_data 只会出现其中之一

**6、<span id="SpeechData">SpeechData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| app_id | string | 用户所属org标识 |
| eof | int32 | 0 : 上传部分音频数据；-1 : 音频数据上传结束 |
| raw_data | bytes | 音频数据 |
| audio_type | string | 音频数据类型 可选项为 pcm, mp3 和 flac |
| user_id | string | 用户唯一标识 |
| platform_type | string | 用户所属平台标识 |
| agent_id | string | 用户对话的agent标识 |
| agent_name | string | 用户对话的agent名称 |
| user_name | string | 用户名，用于显示 |
| one_shot | bool | 是否在语音识别结束之后直接使用结果向对话流系统发起请求 |
| request_data | [DHLMixerRequestData](#DHLMixerRequestData) | 对话流请求数据 |

**7、<span id="SpeechResult">SpeechResult</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| eof | int32 | 0 : 部分语音识别结果; -1 : 语音识别最终结果以及对话流请求响应 |
| result | string | 语音识别结果 |
| dhl_error | [KerfuError](#KerfuError) | 对话流请求失败的错误提示 |
| response | [KerfuResponse](#KerfuResponse) | 对话流请求响应数据 |

**8、<span id="AuthenticationParams">AuthenticationParams</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| app_id | string | 用户所属org标识 |
| app_key | string | 向对话流平台申请的 app_key |
| app_secret | string | 对话流平台生成的 app_secret |

**9、<span id="AccessToken">AccessToken</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| access_token | string | 访问 restful api 时所需的token |

**10、<span id="KerfuMessageType">KerfuMessageType</span>**

\* 枚举类型

| 名称 | 值 | 描述 |
| :------ | :------ | :------ |
| Any | 0 | 任意消息类型 |
| Request | 1 | 对话流请求 |
| Response | 2 | 对话流响应 |
| PhantomResponse | 3 | 对话流响应但不直接展示给用户，可能是需要人工确认的响应等等 |

**11、<span id="DHLMixerRequestData">DHLMixerRequestData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| req_id | string | 请求id，客户端生成，会在针对该请求的服务端响应中出现 |
| message | string | 请求内容 |
| voice_url | string | 请求内容音频url |
| resource_url | string | 请求消息资源url，用于图片视频等非文本类型的请求中 |
| message_content_type | [MessageContentType](#MessageContentType) | 请求消息类型，可能是文本、图片等 |
| force_handle_manually | bool | 是否强制要求人工服务 |
| dhl_request_type | dhl.DHLRequestType | 对话流请求类型，参见dhl.proto |
| dynamic_entities | repeated dhl.DynamicEntity | 动态实体，参见dhl.proto |
| global_attributes | dhl.FilledAttribute | 全局属性，参见dhl.proto |
| intent_name | string | 意图名 |
| local_attributes | repeated dhl.FilledAttribute | 属性，参见dhl.proto |

**12、<span id="DHLMixerResponseData">DHLMixerResponseData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| req_id | string | 请求id |
| message | string | deprecated |
| resource_url | string | 响应消息资源url，用于图片视频等非文本类型的响应中 |
| message_content_type | [MessageContentType](#MessageContentType) | 响应消息类型，可能是文本、图片等 |
| response_type | [ResponseType](#ResponseType) | 响应类型，可能是自动响应、人工响应等 |
| dhl_script | dhl.DHLScript | 对话流系统的响应数据，参见dhl.proto |
| support_platform | string | 人工客服平台 |
| support_app | string | deprecated |
| support_uid | string | 人工客服id |

**13、<span id="KerfuMessageAck">KerfuMessageAck</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| session_id | int32 | 消息所属session的id |
| message_id | int64 | 消息id |
| timestamp | int64 | 服务端时间戳 |

**14、<span id="Action">Action</span>**

\* 枚举类型

| 名称 | 值 | 描述 |
| :------ | :------ | :------ |
| Authentication | 0 | 客户端鉴权请求 |
| EndConversation | 1 | 客户端结束对话请求 |

**15、<span id="KerfuAuthenticationData">KerfuAuthenticationData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| platform_type | string | 用户所属平台标识 |
| app_id | string | 用户所属org标识 |
| user_id | string | 用户id |
| is_support | bool | 是否客服连接，用户端传false |

**16、<span id="EndConversationData">EndConversationData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| platform_type | string | 用户所属平台标识 |
| app_id | string | 用户所属org标识 |
| user_id | string | 用户id |
| agent_id | string | agent标识 |

**17、<span id="Event">Event</span>**

\* 枚举类型

| 名称 | 值 | 描述 |
| :------ | :------ | :------ |
| ActionReply | 0 | 对于客户端请求的回复 |
| MessagePosted | 1 | 有新消息需要拉取 |

**18、<span id="KerfuReplyEventData">KerfuReplyEventData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| seq_reply | string | 客户端请求id |
| status | string | 客户端请求是否成功 |

**19、<span id="KerfuMessagePostedEventData">KerfuMessagePostedEventData</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| session_id | int32 | 新消息所属session标识 |
| message_id | int64 | 新消息id |
| message_type | [KerfuMessageType](#KerfuMessageType) | 新消息类型 |

**20、<span id="KerfuError">KerfuError</span>**

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| error_code | int32 | 错误码 |
| error_message | string | 错误描述 |

**21、<span id="MessageContentType">MessageContentType</span>**

\* 枚举类型

| 名称 | 值 | 描述 |
| :------ | :------ | :------ |
| Text | 0 | 文本消息 |
| Image | 1 | 图片消息 |
| Voice | 2 | 语音消息 |
| Video | 3 | 视频消息 |
| Script | 4 | 脚本消息，见于对话流系统的响应中 |

**22、<span id="ResponseType">ResponseType</span>**

\* 枚举类型

| 名称 | 值 | 描述 |
| :------ | :------ | :------ |
| Auto | 0 | 自动响应 |
| SemiAuto | 1 | 对话流系统响应后经人工确认或修改的半自动响应 |
| Manually | 2 | 人工客服回复 |

## proto文件

**1、<span id="dhlmixer">dhlmixer.proto</span>**

```
syntax = "proto3";

package dhlmixer;

import "dhl.proto";
import "dynamic_entity.proto";
import "filled_attribute.proto";

option go_package = "p";

option java_multiple_files = true;
option java_package = "io.naturali.common.dhl";

message SpeechData {
    string app_id = 1;
    int32 eof = 2; // 0 for voice data; -1 for end of voice
	bytes raw_data = 3;
	string audio_type = 4;
	string user_id = 5;
	string user_info = 6;
	string platform_type = 7;
	string agent_id = 8;
	string agent_name = 9;
	string user_name = 10;
	string data = 11;
	bool one_shot = 12;
	DHLMixerRequestData request_data = 13;
}

message SpeechResult {
    int32 eof = 1; // 0 for partial speech result; -1 for speech result
	string result = 2;
	string voice_url = 3;
	oneof dhl_response {
        KerfuError dhl_error = 4;
        KerfuResponse response = 5;
    }
}

enum KerfuMessageType {
    Any = 0;
    Request = 1;
    Response = 2;
    PhantomResponse = 4;
}

enum MessageContentType {
    Text = 0;
    Image = 1;
    Voice = 2;
    Video = 3;
    Script = 4;
}

enum ResponseType {
    Auto = 0;
    SemiAuto = 1;
    Manually = 2;
}

message DHLMixerRequestData {
    string req_id = 1;
    string message = 2;
    string voice_url = 3;
    string resource_url = 4;
    MessageContentType message_content_type = 5;
    bool force_handle_manually = 6;
    dhl.DHLRequestType dhl_request_type = 7; // dhl request type; reference dhl.proto
    repeated dhl.DynamicEntity dynamic_entities = 8;
    repeated dhl.FilledAttribute global_attributes = 9;
    string intent_name = 10;
    repeated dhl.FilledAttribute local_attributes = 11;
}

message DHLMixerResponseData {
    string req_id = 1;
    string message = 2;
    string resource_url = 3;
    MessageContentType message_content_type = 4;
    ResponseType response_type = 5;
    dhl.DHLScript dhl_script = 6;
    string support_platform = 7;
    string support_app = 8;
    string support_uid = 9;
}

message KerfuMessage {
    int64 message_id = 1;
    int32 session_id = 2;
    KerfuMessageType message_type = 3;
    string platform_type = 4; // dhl_sdk_ios; dhl_sdk_android; wechat_open_platform; wechat_mini_program; dhl_restful;
    string app_id = 5;
    string user_id = 6;
    string user_name = 7;
    string agent_id = 8;
    string agent_name = 9;
    int64 timestamp = 10;
    oneof content {
        DHLMixerRequestData request = 11;
        DHLMixerResponseData response = 12;
    }
}

message KerfuError {
   int32 error_code = 1;
   string error_message = 2;
}

message KerfuMessageAck {
    int32 session_id = 1;
    int64 message_id = 2;
    int64 timestamp = 3;
}

message KerfuResponse {
    KerfuMessageAck ack = 1;
    repeated KerfuMessage messages = 2;
}

message KerfuMessageFilter {
    int32 session_id = 1;
    int64 message_id = 2;
    int64 before = 3;
    int64 after = 4;
    int32 page = 5; // default 0
    int32 page_size = 6; // default 10
    KerfuMessageType message_type = 7;
}

message KerfuMessageList {
    repeated KerfuMessage messages = 1;
}

enum Action {
    Authentication = 0;
    EndConversation = 1;
}

enum Event {
    ActionReply = 0;
    MessagePosted = 1;
}

message KerfuAuthenticationData {
    string platform_type = 1;
    string app_id = 2;
    string user_id = 3;
    bool is_support = 4;
}

message EndConversationData {
    string platform_type = 1;
    string app_id = 2;
    string user_id = 3;
    string agent_id = 4;
}

message KerfuAction {
    string seq = 1;
    Action action = 2;
    oneof data {
        KerfuAuthenticationData authentication_data = 3;
        EndConversationData end_conversation_data = 4;
    }
}

message KerfuReplyEventData {
    string seq_reply = 1;
    string status = 2;
}

message KerfuMessagePostedEventData {
    int32 session_id = 1;
    int64 message_id = 2;
    KerfuMessageType message_type = 3;
}

message KerfuEvent {
    Event event = 1;
    oneof data {
        KerfuReplyEventData reply_event_data = 2;
        KerfuMessagePostedEventData message_posted_data = 3;
    }
}

message AuthenticationParams {
    string app_id = 1;
    string app_key = 2;
    string app_secret = 3;
}

message AccessToken {
    string access_token = 2;
}
```

**2、<span id="dhl">dhl.proto</span>**

```
syntax = "proto3";

package dhl;

import "dhl_response.proto";

option go_package = "p";

option java_multiple_files = true;
option java_package = "io.naturali.common.dhl";

enum DHLRequestType {
    Normal = 0; // normal request
    AgentList = 1; // get agent
    WelcomeMessage = 2; // get welcome message
}

message DHLAgentInfo {
    string id = 1;
    string name = 2;
    string org = 3;
    string description = 4;
    string icon_url = 5;
    string type = 6;
}

message DHLAgentResponse {
    repeated DHLAgentInfo agent_list = 1;
}

message Script {
    oneof script_data {
        string text_response = 1;
        CardResponse card_response = 2;
        DHLAgentResponse agent_response = 3;
    }
}

message ChatMessage {
    oneof chat_message {
        string text_msg = 1;
        CardResponse card_msg = 2;
        string image_response_url = 3;
    }
}

message DHLChatResponse {
    repeated ChatMessage msgs = 1;
    repeated string candidates = 2; // should this be in ChatMessage instead?
}

message DHLScript {
    Script script = 1; // TODO delete after dhl_response is set properly
    repeated string candidates = 2; // TODO delete after dhl_response is set properly
    string modified_query = 3;
    string message = 4;
    oneof dhl_response {
        DHLAgentResponse agent_response = 5;
        DHLChatResponse chat_response = 6;
    }
}
```

**3、<span id="dynamic_entity">dynamic_entity.proto</span>**

```
syntax = "proto3";

package dhl;

option go_package = "p";

option java_multiple_files = true;
option java_package = "io.naturali.common.dhl";

message DynamicEntityValue {
    string keyword = 1;
    repeated string aliases = 2;
}

message DynamicEntity {
    string type_name = 1;
    repeated DynamicEntityValue values = 2;
}
```

**4、<span id="filled_attribute">filled_attribute.proto</span>**

```
syntax = "proto3";

package dhl;

option java_multiple_files = true;
option java_package = "io.naturali.common.dhl";

message FilledAttribute {
    string name = 1;
    string value = 2;
    int64 timestamp = 3;
}
```

**5、<span id="dhl_response">dhl_response.proto</span>**

```
syntax = "proto3";

package dhl;

option java_multiple_files = true;
option java_package = "io.naturali.common.dhl";

message FilledAttribute {
    string name = 1;
    string value = 2;
    int64 timestamp = 3;
}
```
