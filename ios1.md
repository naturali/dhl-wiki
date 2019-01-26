#NaturaliSDK接口文档
###SDK初始化
引用头文件

```
#import <NaturaliSDK.h>
```
注册SDK

```
[NaturaliSDK registerAppId:@"yourAppId"
                    appKey:@"yourAppKey"
                 appSecret:@"yourAppSecret"];
```

设置默认的用户id

```
//设置userId,发送消息的用户id
    [NaturaliSDK setUserId:@"myUserId"];
```
添加动态实体

```
NATDynamicEntityValue *value1 = [NATDynamicEntityValue new];
value1.keyword = @"keyword1";
value1.aliases = @[@"alias1",@"alias2"];

NATDynamicEntityValue *value2 = [NATDynamicEntityValue new];
value2.keyword = @"keyword2";
value2.aliases = @[@"alias3",@"alias4"];

NATDynamicEntityValue *value3 = [NATDynamicEntityValue new];
value3.keyword = @"keyword3";
value2.aliases = @[@"alias5",@"alias4"];

NATDynamicEntity *entity = [NATDynamicEntity new];
entity.typeName = @"yourTypeName";
entity.values = @[value1,value2,value3];

[NaturaliSDK addDynamicEntities:@[entity]];
```
###语音识别
创建语音识别器并设置代理对象，回调将通过代理对象返回。

```
self.recognizer = [[NATSpeechRecognizer alloc] initWithDelegate:self];
```
开始录音

```
[self.recognizer startRecording];
```
或者直接开始语音使用对话流

```
[self.recognizer startDialogRecordingWithAgentId:self.agentIdLabel.text];
```
结束录音

```
[self.recognizer stopRecording];
```
中途取消录音

```
[self.recognizer cancelRecording];
```
实现语音识别的回调方法

```
- (void)onCompleted:(NSError *)error {
    //语音录制结束，若无异常，则error为nil
    NSLog(@"speech recognizer complete with error : %@",error);
}

- (void)onResult:(NSString *)result isDone:(BOOL)isDone voiceUrl:(NSString *)voiceUrl {
    //实时监听语音录制的识别结果，当语音录制结束时，isDone为true
    self.queryInput.text = result;
}

#pragma mark - dialog manager delegate
- (void)didReceiveResponse:(NATDialogResponse *)response {
    //接收对话流消息，可能包含文本和多媒体资源地址.
    self.receiveTextView.text = response.content;
    [self.linkButton setTitle:response.linkUrl forState:UIControlStateNormal];
    
    NSData *imageData = [NSData dataWithContentsOfURL:[NSURL URLWithString:response.imageUrl]];
    self.receiveImageView.image = [UIImage imageWithData:imageData];
}
```
其他可选监听回调

```
@optional

/*!
 *  开始录音
 *  当调用了`startRecording`函数之后，如果没有发生错误则会回调此函数。
 */
- (void)onBeginOfSpeech;

/*!
 *  停止录音
 *  当调用了`stopRecording`函数之后，会回调此函数
 */
- (void)onEndOfSpeech;

/*!
 *  取消识别回调
 *  当调用了`cancel`函数之后，会回调此函数

 */
- (void)onCancel;
```
###对话流
设置对话流的代理对象，用于接收对话消息.

```
[NATDialogManager sharedInstance].delegate = self;
```
发送对话流消息,至少需要文本和agentId参数

```
NATDialogRequest *request = [[NATDialogRequest alloc] init];
request.query = query;
request.agentId = self.agentIdLabel.text;
[[NATDialogManager sharedInstance] sendDialogRequest:request compeltion:^(BOOL success, NSError *error, NSString *reqId) {
    if (error) {
        NSLog(@"发送对话发生错误：\n%@",error);
    } else if (success) {
        NSLog(@"发送成功");
    }
}];
```
监听发送成功的消息内容和接收到的消息内容

```
#pragma mark - dialog manager delegate
- (void)didReceiveResponse:(NATDialogResponse *)response {
    //接收对话流消息，可能包含文本和多媒体资源地址.
    self.receiveTextView.text = response.content;
    [self.linkButton setTitle:response.linkUrl forState:UIControlStateNormal];
    
    NSData *imageData = [NSData dataWithContentsOfURL:[NSURL URLWithString:response.imageUrl]];
    self.receiveImageView.image = [UIImage imageWithData:imageData];
}
```
**注:用户发送成功的消息也会生成消息记录通过本方法返回**

结束并重置与指定agent的对话上下文

```
 [[NATDialogManager sharedInstance] endConversationWithAgentId:self.agentIdLabel.text compeltion:^(BOOL success, NSError *error, NSString *requestId) {
        NSLog(@"conversation end:%@ , \nerror:%@",success ? @"yes" : @"no", error);
    }];
```