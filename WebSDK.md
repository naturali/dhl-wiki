# DHL Web SDK

Use to create your DHL(Dui Hua Liu) web client.

## 📦 Install

- Use [npm](https://www.npmjs.com/package/dhl-core):

  ```bash
  npm install dhl-core --save
  ```

- Use Yarn

  ```bash
  yarn add dhl-core
  ```

- Use CDN

  ```javascript
  <script src="https://dhl-sdk.oss-cn-beijing.aliyuncs.com/web_sdk/1.0.0/dhl.js" />
  ```

## 🔨 Usage

- From node_modules

  ```typescript
  import { DHL } from 'dhl-core';

  // Create a DHL object
  const client = new DHL(
    {
      appId: 'your app id',
      appKey: 'your app key',
      appSecret: 'your secret key'
    },
    () => {
      // init completed
    }
  );
  ```

- from CDN

  ```javascript
  <script src="https://dhl-sdk.oss-cn-beijing.aliyuncs.com/web_sdk/1.0.0/dhl.js"></script>
  <script>
    var client = new DHL({
      appId: "your app id",
      appKey: "your app key",
      appSecret: "your secret key"
    },()=>{
      // init completed
    });
  </script>
  ```

## 🌈 Methods

- Connect Websocket and listen messages from server.

  ```typescript
  connectWebsocket(user: UserInfo,
    onMessage: (msgResults: MessageResult[]) => void,
    onOpen?: () => void,
    onClose?: () => void,
    onError?: (error: any) => void)
  ```

- Send messages and Receive Server callback

  ```typescript
  send(msgRequest: MessageRequest,
    onSuccess?: (msgResult: MessageResult[]) => void,
    onFailed?: (code: number, message: string) => void)

  interface MessageRequest {
    reqId: string;
    appId?: string;
    platformType?: string;
    type: number;
    message?: string;
    audioUrl?: string;
    imageUrl?: string;
    videoUrl?: string;
    userId?: string;
    agentId?: string;
    agentName?: string;
    forceHandleManually?: boolean;
  }

  // Request type
  export const RequestType = {
    NORMAL: dhl.DHLRequestType.Normal,
    AGENT_LIST: dhl.DHLRequestType.AgentList,
    WELCOME_MESSAGE: dhl.DHLRequestType.WelcomeMessage
  };

  // You can receive message results from during onSuccess callback
  interface MessageResult {
    id: string;
    serverMsgId: number | Long;
    creatTime: Date;
    receiverId?: string;
    senderId?: string;
    userId?: string;
    agentId?: string;
    messageType?: number;
    content?: MessageContent; // Use to save message detail
  }

  // Use for messageType
  const MessageType = {
    TEXT: 1,
    IMAGE: 2,
    AUDIO: 3,
    VIDEO: 4,
    SINGLE_CARD: 5,
    MULTI_CARD: 6,
    TEXT_LIST: 7,
    OTHERS: 8
  };

  // Use to save message detail
  interface MessageContent {
    reqId?: string;
    message?: string;
    audioUrl?: string;
    videoUrl?: string;
    url?: string;
    title?: string;
    description?: string;
    imgUrl?: string;
    script?: string;
    candidates?: (string[] | null);
    listCard?: ListCard[];
    listCardType?: number;
  }

  // Use for card list type and text list type
  interface ListCard {
    title?: string;
    avatar?: string;
    description?: string;
    description2?: string;
    link?: string;
    script?: string;
    coreference?: string;
    useCoreference?: boolean;
  }
  ```

- MessageResult Filed Detail

  - MessageResult Filed

| type           | filed       |                                                              |
| -------------- | ----------- | ------------------------------------------------------------ |
| string         | id          | 获取消息的 uuid，在生成消息时会自动赋值，对于 MessageRequest 可以使用 MessageRequest.isSame(MessageResult result) 方法判断是否是同一条消息 |
| Date           | creatTime   | 获取消息接收的时间                                           |
| string         | receiverId  | 获取消息接收者 id                                            |
| string         | senderId    | 获取消息发送者 id                                            |
| string         | agentId     | 获取消息对应的 Agent Id                                      |
| number         | messageType | 获取消息类型<br>MessageResult.TEXT 为文字类型<br>MessageResult.IMAGE 为图片类型<br>MessageResult.AUDIO 为音频类型<br>MessageResult.VIDEO 为视频类型<br>MessageResult.SINGLE_CARD 为卡片类型,卡片保存在 listcard 里<br>MessageResult.MULTI_CARD 为卡片列表类型<br>MessageResult.TEXT_LIST 为文字列表类型 |
| MessageContent | content     | 获取消息内容 详见下节                                        |

  - MessageContent Filed

  **注：** 每条收到的消息都可能有备选选项，通过 `getCandidates` 获取

| 消息类型 | 消息内容                                                            |
| -------- | ------------------------------------------------------------------- |
| 文本消息 | message 消息文字                                                    |
| 图片消息 | title 图片标题<br>description 图片描述<br>imgUrl 图片链接           |
| 音频消息 | audioUrl 音频链接                                                   |
| 视频消息 | videoUrl 视频链接                                                   |
| 卡片列表 | listCard 返回一个 `ListCard[]` 数组类型的对象<br>title 卡片列表标题 |

  - ListCard 属性说明

| 返回类型 | 接口           | 说明                     |
| -------- | -------------- | ------------------------ |
| string   | title          | 获取卡片标题             |
| string   | avatar         | 获取卡片图标             |
| string   | description    | 获取卡片描述             |
| string   | description2   | 获取卡片副描述           |
| string   | link           | 获取跳转链接             |
| string   | script         | 获取链接注入的 script    |
| string   | coreference    | 获取卡片点击时发送的文字 |
| boolean  | useCoreference | 是否使用 coreference     |

## License

MIT License.
