# Android 开发集成

## 概要介绍

对话流 Android SDK 分为 `common` (基础模块) 和 `speech` (语音识别模块) 两部分。基础模块可以单独使用，语音识别模块需要依赖基础模块使用

## 基础模块集成

基础模块 `common` SDK 是对话流服务其他能力（语音识别等）的基础，提供了消息收发，语音、图片、视频、文件上传功能，本节将介绍基础模块的集成步骤。当需要集成多个模块的时候，请确保每个模块的版本号完全相同。

### 准备工作

**1、系统要求**

```
minSdkVersion 21

ndk {
    // SDK 目前仅支持 armeabi 一种 SO 库架构。该方案牺牲少许性能，可显著减小 apk 的大小
    abiFilters 'armeabi'
}

compileOptions {
    sourceCompatibility = '1.8'
    targetCompatibility = '1.8'
}
```

**2、依赖添加**

首先从网站上下载 `common-release-xxxxx.aar` 并复制到主工程中与 `src` 目录平级的 `libs` 目录下，并在主工程的 `build.gradle` 文件中添加 `dependencies` :

```
repositories {
    flatDir {
        // 指定 `aar` 文件的目录
        dirs 'libs'
    }
}

dependencies {
    // 添加依赖
    // 由于 SDK 使用 Kotlin 语言进行开发，所以需要添加 Kotlin 的基础库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$rootProject.ext.kotlinVersion"

    // Kotlin 协程库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$rootProject.ext.coroutinesVersion"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$rootProject.ext.coroutinesVersion"

    // Gson 解析库
    implementation "com.google.code.gson:gson:$rootProject.ext.gsonVersion"

    // RxJava
    implementation "io.reactivex.rxjava2:rxandroid:$rootProject.ext.rxAndroidVersion"
    implementation "io.reactivex.rxjava2:rxjava:$rootProject.ext.rxJavaVersion"

    // androidx 注解库
    implementation "androidx.annotation:annotation:$rootProject.ext.annotationVersion"

    // Proto
    implementation "io.grpc:grpc-protobuf-lite:$rootProject.ext.grpcVersion"
    implementation 'javax.annotation:javax.annotation-api:1.2'

    // OkHttp
    implementation("com.squareup.okhttp3:okhttp:3.12.0")

    //DBFlow
    annotationProcessor "com.github.Raizlabs.DBFlow:dbflow-processor:${rootProject.ext.dbflowVersion}"
    implementation "com.github.Raizlabs.DBFlow:dbflow-core:${rootProject.ext.dbflowVersion}"
    implementation "com.github.Raizlabs.DBFlow:dbflow:${rootProject.ext.dbflowVersion}"
    implementation "com.github.Raizlabs.DBFlow:dbflow-kotlinextensions:${rootProject.ext.dbflowVersion}"

    implementation(name: 'common-release-xxxxx', ext: 'aar')
}
```

在整个工程的最外层 `build.gradle` 中配置 `version` 和 `repositories` :

```
allprojects {
    repositories {
        // DBFlow
        maven { url 'https://jitpack.io' }
    }
}

ext {
    // 三方库文件的版本信息
    kotlinVersion = "1.3.10"
    coroutinesVersion = "1.0.0"
    annotationVersion = "1.0.0"
    rxJavaVersion = "2.2.3"
    rxAndroidVersion = "2.1.0"
    grpcVersion = "1.16.1"
    gsonVersion = "2.8.5"
    dbflowVersion = "4.2.4"
}
```

**3、权限和 Key 配置**

在 `AndroidManifest.xml` 中加入以下配置:

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="xxx.xxx.xxx">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <!-- required for user data upload service -->
    <uses-permission
        android:maxSdkVersion="18"
        android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">

        <!-- add for naturali server checking -->
        <meta-data
            android:name="io.naturali.dhl.APP_KEY"
            android:value="app_key" />

        <meta-data
            android:name="io.naturali.dhl.SECRET_KEY"
            android:value="secret_key" />

        <meta-data
            android:name="io.naturali.dhl.APP_ID"
            android:value="app_id" />
    </application>
</manifest>
```

**4、混淆配置**

对话流 SDK aar 自带混淆配置文件，无需特意对其进行混淆配置

**5、初始化**

在 `Application.onCreate()` 方法中初始化：

```
public class BaseApplication extends Application {

    /**
     * 每个进程都会创建自己的 Application 并调用 onCreate 方法，建议初始化时对进程进行判断
    **/
    public void onCreate() {
        // ... init others

        // SDK 初始化，SDK的所有操作都需要在初始化之后才能调用
        MessageManager.init(context);
    }
}
```

在用户 **登陆** 之后初始化用户信息，每次切换用户需要重新初始化，避免数据混乱：

```
    // user id 用于标识当前用户的唯一id，由调用方生成，自行维护
    // user name 选填，可以不填，用于后台一些可视化界面的提示
    UserInfo info = new UserInfo("userId", "userName");
    MessageManager.initUserInfo(info);
```

至此，您已经可以开始使用对话流 SDK 提供的各种功能了。

### 消息收发

* SDK 的消息发送对象均为 `MessageRequest` ，通过Builder模式生成。消息的类型根据 `video > audio > image > text` 的优先级自动判断，如果四个域均未赋值，则视为无效请求，不进行发送操作。创建一条消息的方法为：

    ```
    MessageRequest request = new MessageRequest.Builder()
        .agentId("agent id") // 当前 agent id，必填
        .agentName("") // 当前 agent 名字，用于后台可视化信息，选填
        .userId("user id") // user id，选填
        .query("text 请求") // 发送文字消息时填写
        .audioFilePath("") // audio 文件的本地路径，发送 audio 消息时填写
        .audioUrl("") // audio 文件的云存储地址，与 audioFilePath 二选一
        .videoFilePath("") // video 文件的本地路径，发送 video 消息时填写
        .videoUrl("") // video 文件的云存储地址，与 videoFilePath 二选一
        .imageFilePath("") // image 文件的本地路径，发送 image 消息时填写
        .imageUrl("") // image 文件的云存储地址，与 imageFilePath 二选一
        .forceHandleManually(false) // 是否强制跳转人工客服，默认为 false
        .requestType(MessageRequest.TYPE_NORMAL) // 请求类型，通常为Normal，可不填
        .build() // 生成 MessageRequest 实例，用于传入 MessageManager.sendMessage() 中
    ```

* 创建一个MessageManager实例，发送和接收消息。发送的消息会先存到数据库中，并回调 `onReceive()` 方法返回 `MessageResult` ，因此我们推荐使用 `MessageResult` 作为 UI 显示的对象。当信息发送成功后，会再次更新数据库，并回调 `onReceive()` 方法返回同一个 `MessageResult` ，请注意对MessageResult进行去重和状态的判断，MessageResult 的 id 相同即为同一条消息。

    ```
    // 创建 MessageManager 实例
    final MessageManager manager = new MessageManager("agent id");

    // 注册 MessageManager 回调
    manager.setListener(new MessageListener() {
        @Override
        public void onReceive(MessageResult messageResult) {
            // 接收到的单条消息
        }

        @Override
        public void onReceive(List<MessageResult> list) {
            // 接收到的消息列表
        }

        @Override
        public void onError(QueryError error) {
            // 发生异常回调
        }
    });

    // 发送消息，传入 MessageRequest
    manager.sendMessage(request);

    // 删除消息，传入 MessageResult，不可以使用新建的实例
    manager.deleteMessage(messageResult);

    // 读取历史信息
    manager.loadMessage();
    ```

* MessageResult 参数说明

| 返回类型 | 接口 | 说明 |
| - | - | - |
| String | getId | 获取消息的uuid，在生成消息时会自动赋值，对于 MessageRequest 可以使用 MessageRequest.isSame(MessageResult result) 方法判断是否是同一条消息 |
| Date | getCreateTime | 获取消息接收的时间 |
| String | getReceiverId | 获取消息接收者 id |
| String | getSenderId | 获取消息发送者 id |
| String | getAgentId | 获取消息对应的 Agent Id |
| String | getSessionId | 获取消息对应的聊天窗口 Session Id |
| Int | getMessageType | 获取消息类型<br>MessageResult.TYPE_TEXT 为文字类型<br>MessageResult.TYPE_IMAGE 为图片类型<br>MessageResult.TYPE_AUDIO 为音频类型<br>MessageResult.TYPE_VIDEO 为视频类型<br>MessageResult.TYPE_LINK 为链接类型<br>MessageResult.TYPE_MIXED 为混合类型<br>MessageResult.TYPE_OTHER 为未知类型 |
| Int | getStatus | 获取消息状态<br>MessageResult.STATUS_DONE = 0 消息发送成功<br>MessageResult.STATUS_CREATED = 1 消息创建<br>MessageResult.STATUS_FAILED = 2 消息发送失败 |
| MessageContent | getContent | 获取消息内容 详见下节 |

* MessageContent 参数说明

**注：** 每条收到的消息都可能有备选选项，通过 `getCandidates` 获取

| 消息类型 | 消息内容 |
| ------- | ------ |
| 文本消息 | getMessage 消息文字 |
| 图片消息 | getTitle 图片标题<br>getDescription 图片描述<br>getImgUrl 图片链接 |
| 音频消息 | getAudioUrl 音频链接 |
| 视频消息 | getVideoUrl 视频链接 |
| 链接消息 | getTitle 链接标题<br>getDescription 链接描述<br>getScript 链接需要注入的JavaScript脚本<br>getUrl 链接（以 `http` 或者 `https` 开头为网页链接，其他的为Android deeplink） |

## 语音识别模块集成

### 准备工作

**1、系统要求**

同 `common` 库

**2、依赖添加**

首先从网站上下载 `speech-release-xxxxx.aar` 并复制到主工程中与 `src` 目录平级的 `libs` 目录下，并在主工程的 `build.gradle` 文件中添加 `dependencies` ：

```
repositories {
    flatDir {
        // 指定 `aar` 文件的目录
        dirs 'libs'
    }
}

dependencies {
    // 添加依赖 需要先添加 common 相关依赖，详情见上一节
    // 由于 SDK 使用 Kotlin 语言进行开发，所以需要添加 Kotlin 的基础库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$rootProject.ext.kotlinVersion"

    implementation "androidx.annotation:annotation:$rootProject.ext.annotationVersion"
    implementation 'net.sourceforge.javaflacencoder:java-flac-encoder:0.3.6'

    implementation(name: 'speech-release-xxxxx', ext: 'aar')
}
```

在整个工程的最外层 `build.gradle` 中配置 `version` 和 `repositories` :

```
ext {
    // 三方库文件的版本信息
    kotlinVersion = "1.3.10"
    annotationVersion = "1.0.0"
}
```

**3、权限和 Key 配置**

在 `AndroidManifest.xml` 中加入以下配置:

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="xxx.xxx.xxx">

    // 录音权限
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    // 以下部分与 common 相同
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <!-- required for user data upload service -->
    <uses-permission
        android:maxSdkVersion="18"
        android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">

        <!-- add for naturali server checking -->
        <meta-data
            android:name="io.naturali.dhl.APP_KEY"
            android:value="app_key" />

        <meta-data
            android:name="io.naturali.dhl.SECRET_KEY"
            android:value="secret_key" />

        <meta-data
            android:name="io.naturali.dhl.APP_ID"
            android:value="app_id" />
    </application>
</manifest>
```

**4、混淆配置**

对话流 SDK aar 自带混淆配置文件，无需特意对其进行混淆配置

**5、初始化**

在 `Application.onCreate()` 方法中初始化：

```
public class BaseApplication extends Application {

    /**
     * 每个进程都会创建自己的 Application 并调用 onCreate 方法，建议初始化时对进程进行判断
    **/
    public void onCreate() {
        // ... init others

        // Speech SDK 初始化，SDK的所有操作都需要在初始化之后才能调用
        SpeechRecognizerWrapper.getInstance().init(this);
    }
}
```

至此，您已经可以开始使用语音识别 SDK 提供的各种功能了。

### 配置参数

```
Bundle bundle = new SpeechExtras.Builder()
    .agentId("agent id") // 当前 agent id
    .userId("user id") // 当前用户 user id
    .forceHandleManually(false) // 是否强制进入人工客服，默认为 false
    .oneShot(false) // 是否在返回语音结果的同时返回对话流服务结果，默认为 false
    .build(); // 生成 Bundle 文件，用于传入 SpeechRecognizerWrapper.getInstance().start() 方法中
```

**注：** 开启 `oneShot` 之后不要使用语音识别的文字结果调用 MessageManager.sendMessage() 发送消息，否则会收到重复的回复信息，因为 `oneShot` 本身就会发起一次请求

### 语音识别

调用 `SpeechRecognizerWrapper.getInstance().start(SpeechListener listener, Bundle bundle)` 开始麦克风收音

```
// 开始语音识别
SpeechRecognizerWrapper.getInstance().start(new SpeechListener() {

    @Override
    public void onReadyForSpeech(Bundle bundle) {
        // 资源准备完毕
    }

    @Override
    public void onRmsChanged(float v) {
        // 说话音量大小监听
    }

    @Override
    public void onBeginningOfSpeech() {
        // 开始语音识别
    }

    @Override
    public void onBufferReceived(byte[] bytes) {
        // 收到的 buffer 信息，暂时无用
    }

    @Override
    public void onEndOfSpeech() {
        // 语音识别结束
    }

    @Override
    public void onResults(Bundle bundle) {
        // 收到的最终识别结果
        String text = bundle
            .getString(SpeechRecognizer.RESULTS_RECOGNITION);
    }

    @Override
    public void onPartialResults(Bundle bundle) {
        // 收到的部分识别结果
        String text = bundle
            .getString(SpeechRecognizer.RESULTS_RECOGNITION);
    }

    @Override
    public void onError(String s, int i) {
        // 语音识别异常
    }

    @Override
    public void onEvent(int i, Bundle bundle) {
    }
}, bundle);

// 停止语音识别
SpeechRecognizerWrapper.getInstance().stop()

// 取消语音识别
SpeechRecognizerWrapper.getInstance().cancel()
```
