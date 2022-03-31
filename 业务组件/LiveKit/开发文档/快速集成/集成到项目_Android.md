# 互动直播组件
互动直播Android SDK提供了一套简单易用的接口，允许开发者通过调用NELiveKit SDK(以下简称SDK)提供的API，快速地集成互动直播功能至现有Android应用中。

## 快速接入
 ### 开发环境
   | 名称 | 要求 |
   | :------ | :------ |
   | JDK版本  | >1.8.0 |
   | 最小Android API 版本 | API 21, Android 5.0 |
   | CPU架构支持 | ARM64、ARMV7 |
   | IDE | Android Studio |
   | 其他 | 依赖androidx，不支持support库 |

### SDK集成

1. 新建Android工程

    a. 运行Android Sudio，顶部菜单依次选择“File -> New -> New Project...”新建工程，选择'Phone and Tablet' -> 'Empty Activity' 单击Next。

    ![new android project](../../images/android_create_project.png)

    b. 配置工程相关信息，请注意Minimum API Level为API 21。

    ![configure project](../../images/android_create_project_set.png)

    c. 单击'Finish'完成工程创建。

2. 添加SDK编译依赖

    修改工程目录下的'app/build.gradle'文件，添加互动直播SDK的依赖。
    ```groovy
    android {
      // 添加 packagingOptions，否则可能会造成资源文件冲突
      packagingOptions {
        pickFirst 'lib/arm64-v8a/libc++_shared.so'
        pickFirst 'lib/armeabi-v7a/libc++_shared.so'
      }
    }

    dependencies {
	    //声明SDK依赖，版本可根据实际需要修改
	    implementation 'com.netease.yunxin.kit.live:livekit:1.0.0'
    }
	```
    之后通过顶部菜单'Build -> Make Project'构建工程，触发依赖下载，完成后即可在代码中引入SDK中的类和方法。

3. 权限处理

    互动直播SDK正常工作需要应用获取以下权限
    ```xml
    <!-- 网络相关 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

    <!-- 读写外部存储 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <!-- 多媒体 -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    ```
    以上权限已经在SDK内部进行声明，开发者可以不用在```AndroidManifest.xml```文件中重新声明这些权限，但运行时的权限申请需要应用开发者自己编码实现，可在应用首页中统一申请，详情可参考[Android运行时权限申请示例](https://developer.android.google.cn/guide/topics/permissions/overview)。如果运行时对应权限缺失，SDK可能无法正常工作，如直播时无图像、对方听不到己方声音等。

4. SDK初始化

    在使用SDK其他功能之前首先需要完成SDK初始化，初始化操作需要保证在**Application**的**onCreate**方法中执行。代码示例如下：
    ```java
    public class MyApplication extends Application {
    
        private static final String TAG = "MyApplication";
    
        @Override
        public void onCreate() {
            super.onCreate();
            NELiveKitOptions options = new NELiveKitOptions(Constants.APPKEY, extras);
            NELiveKit.getInstance().initialize(this, options, new NELiveCallback<Unit>() {
                @Override
                public void onSuccess(Unit unit) {
                    //TODO when initialize success
                }
                public void onFailure(int code, String msg) {
                    //TODO when initialize fail
                }
            });
        }
    }
    ```

5. 调用相关接口完成特定功能，详情请参考API文档。

- [登录鉴权](#登录鉴权)
    ```java
    //Token登录
    NELiveKit.getInstance().login(String account, String token, NELiveCallback<Unit> callback);
    
    ```
- [开始直播](#创建直播)
    ```java
    NELiveKit.getInstance().startLive(int liveType, String liveTopic, String cover, NELiveCallback<LiveInfo> callback)
    ```
- [退出直播](#停止直播)
    ```java
    NELiveKit.getInstance().stopLive(NELiveCallback<Unit> callback)
    ```
  
- [加入其他直播间](#加入其他直播间)
    ```java
    NELiveKit.getInstance().joinLive(liveInfo: LiveInfo, NELiveCallback<LiveInfo> callback)
    ```
- [退出其他直播间](#退出其他直播间)
    ```java
    NELiveKit.getInstance().leaveLive(NELiveCallback<Unit> callback)
    ```
- [注销登录](#注销)
    ```java
    NELiveKit.getInstance().logout(NELiveCallback<Unit> callback)
    ```

## 业务开发

### 初始化

#### 描述

在使用SDK其他接口之前，首先需要完成初始化操作。

#### 业务流程


调用接口并进行回调处理，该接口无额外回调结果数据

    ```java
    NELiveKit.getInstance().initialize(this, options, new NELiveCallback<Unit>() {
                @Override
                public void onSuccess(Unit unit) {
                    //TODO when initialize success
                }
                public void onFailure(int code, String msg) {
                    //TODO when initialize fail
                }
    });
    ```

--------------------

### 登录鉴权

#### 描述

请求SDK进行登录鉴权，只有完成SDK登录鉴权才允许创建直播间。SDK提供了多种登录方式可供选择，不同的登录接口需要不同的入参数。说明如下：

| 登录方式 | 说明 | 接口 | 参数 | 其他                              |
| :------ | :------ | :------ | :------ |:--------------------------------|
| Token登录 | 无 | `NELiveKit#login` | accountId、accountToken | 账号信息需要从互动直播服务器获取，由接入方自行实现相关业务逻辑 |


#### 业务流程

1. 获取登录用账号ID和Token。Token由网易互动直播应用服务器下发，但SDK不提供对应接口获取该信息，需要开发者自己实现。

    ```java
    String accountId = "accountId";
    String accountToken = "accountToken";
    ```

2. 登录并进行回调处理，该接口无额外回调结果数据

    ```java
    NELiveKit.getInstance().login(accountId, accountToken, new NELiveCallback<Unit>() {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when login success
        }
        public void onFailure(int code, String msg) {
            //TODO when login fail
        }
    });
    ```

#### 注意事项

- SDK不提供账号注册机制，第三方应用集成SDK时需要为第三方应用的用户帐号绑定互动直播系统中的帐号

--------------------

### 创建直播间

#### 描述

在已经完成SDK登录鉴权的状态下，创建并开始一个新的直播间。

#### 业务流程

调用创建直播接口并进行回调处理。

    ```java
    NELiveKit.getInstance().startLive(int liveType, String liveTopic, String cover, new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when startLive success
        }
        public void onFailure(int code, String msg) {
            //TODO when startLive fail
        }
    });
    ```

2. 创建直播成功后，可以通过查询直播间列表查看该直播间,其他观众可通过直播号加入到该直播中来。

#### 注意事项

- 该接口仅支持**在登录鉴权成功后调用**，其他状态下调用不会成功

--------------------

### 停止直播间

#### 描述

在已经完成SDK登录鉴权的状态下，停止一个自己创建的直播间。

#### 业务流程

调用停止直播接口并进行回调处理。

    ```java
    NELiveKit.getInstance().stopLive(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when stopLive success
        }
        public void onFailure(int code, String msg) {
            //TODO when stopLive fail
        }
    });
    ```

2. 停止直播成功后，其他观众不可以再通过直播号加入到该直播中来。

#### 注意事项

- 该接口仅支持**在登录鉴权成功后调用**，其他状态下调用不会成功

--------------------

### 加入直播间

#### 描述

在已登录的状态下，加入一个当前正在进行中的直播间。

#### 业务流程

调用加入直播间接口并进行回调处理。

    ```java
    NELiveKit.getInstance().joinLive(String roomUuid, new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when joinLive success
        }
        public void onFailure(int code, String msg) {
            //TODO when joinLive fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 离开直播间

#### 描述

在已登录的状态下，离开一个当前正在进行中的直播间。

#### 业务流程

调用离开直播间接口并进行回调处理。

    ```java
    NELiveKit.getInstance().leaveLive(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when leaveLive success
        }
        public void onFailure(int code, String msg) {
            //TODO when leaveLive fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------


### 邀请其他主播PK

#### 描述

在已开播的状态下，邀请其他主播进行PK。

#### 业务流程

调用邀请其他主播PK接口并进行回调处理。

    ```java
    NELiveKit.getInstance().invitePK(String accId, new NELiveCallback<AnchorPkInfo> {
        @Override
        public void onSuccess(AnchorPkInfo anchorPkInfo) {
            //TODO when invitePK success
        }
        public void onFailure(int code, String msg) {
            //TODO when invitePK fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------


### 取消邀请其他主播PK

#### 描述

在已开播的状态下，取消邀请其他主播进行PK。

#### 业务流程

调用取消邀请其他主播PK接口并进行回调处理。

    ```java
    NELiveKit.getInstance().cancelInvitePK(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when cancelInvitePK success
        }
        public void onFailure(int code, String msg) {
            //TODO when cancelInvitePK fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 接受其他主播的PK邀请

#### 描述

在已开播的状态下，接受其他主播的PK邀请。

#### 业务流程

调用接受其他主播的PK邀请接口并进行回调处理。

    ```java
    NELiveKit.getInstance().acceptPK(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when acceptPK success
        }
        public void onFailure(int code, String msg) {
            //TODO when acceptPK fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 拒接其他主播的PK邀请

#### 描述

在已开播的状态下，拒绝其他主播的PK邀请。

#### 业务流程

调用拒接其他主播的PK邀请接口并进行回调处理。

    ```java
    NELiveKit.getInstance().rejectPK(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when rejectPK success
        }
        public void onFailure(int code, String msg) {
            //TODO when rejectPK fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 给主播打赏礼物

#### 描述

在加入其他主播的直播间的状态下，给该主播进行打赏。

#### 业务流程

调用给主播打赏礼物接口并进行回调处理。

    ```java
    NELiveKit.getInstance().reward(int gift, new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when reward success
        }
        public void onFailure(int code, String msg) {
            //TODO when reward fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 获取直播间成员列表

#### 描述

在加入其他主播的直播间或者自己开播的状态下，获取成员列表。

#### 业务流程

获取直播间成员列表。

    ```java
    NELiveKit.getInstance().getMembers();
    ```

#### 注意事项

- 该接口仅支持直播间内调用
--------------------

### 查询当前直播间是否在PK

#### 描述

在加入其他主播的直播间或者自己开播的状态下，查询是否PK。

#### 业务流程

获取直播间PK状态。

    ```java
    NELiveKit.getInstance().isPK();
    ```

#### 注意事项

- 该接口仅支持直播间内调用
--------------------

### 查询直播间信息

#### 描述

在已登录状态下，查询一个直播间信息

#### 业务流程

1. 调用查询直播间信息接口并进行回调处理，可根据错误码判断是否成功

    ```java
    NELiveKit.getInstance().requestLiveInfo(String liveRecordId, new NELiveCallback<LiveInfo> {
        @Override
        public void onSuccess(LiveInfo unit) {
            //TODO when requestLiveInfo success
        }
        public void onFailure(int code, String msg) {
            //TODO when requestLiveInfo fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 查询直播间列表

#### 描述

在已登录状态下，查询直播间列表

#### 业务流程

1. 调用查询直播间列表接口并进行回调处理，可根据错误码判断是否成功

    ```java
   NELiveKit.getInstance().requestLiveList(int type, int status, int pageNum, int pageSize, new NELiveCallback<LiveListResponse> {
        @Override
        public void onSuccess(LiveListResponse unit) {
            //TODO when requestLiveList success
        }
        public void onFailure(int code, String msg) {
            //TODO when requestLiveList fail
        }
    });
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用

--------------------

### 监听直播间状态

#### 描述

通过注册直播间状态回调接口，可获取到直播间状态变更的通知。

#### 业务流程

1. 设置直播间监听器，并在回调方法中处理感兴趣的事件

    ```java
    NELiveListener listener = new NELiveListener() {
        @Override
        public void onLiveStarted() {
            //直播开始
        }
        @Override
        public void onLoginKickedOut() {
            //登录被踢出
        }
        @Override
        public void onLiveEnd(int reason) {
            //直播结束
        }
        @Override
        public void onTextMessageReceived(@NonNull NERoomTextMessage message) {
            //收到消息
        }
        @Override
        public void onRewardReceived(@NonNull RewardMsg rewardMsg) {
            //收到礼物消息
        }
        @Override
        public int onVideoFrameCallback(@NonNull VideoFrame videoFrame) {
            //视频帧回调，可以做美颜处理
        }
        @Override
        public void onPKInvited(@NonNull PkActionMsg pkUser) {
            //收到PK邀请
        }
        @Override
        public void onPKAccepted(@NonNull PkActionMsg pkUser) {
            //其他主播接受我的PK邀请
        }
        @Override
        public void onPKRejected(@NonNull PkActionMsg pkUser) {
            //其他主播拒绝我的PK邀请
        }
        @Override
        public void onPKCanceled(@NonNull PkActionMsg pkUser) {
            //其他主播取消PK邀请
        }
        @Override
        public void onPKTimeoutCanceled(@NonNull PkActionMsg pkUser) {
            //PK邀请超时
        }
        @Override
        public void onPKStart(@NonNull PkStartInfo startInfo) {
            //PK开始
        }
        @Override
        public void onPKPunishStart(@NonNull PkPunishInfo punishInfo) {
            //PK惩罚开始
        }
        @Override
        public void onPKEnd(@NonNull PkEndInfo endInfo) {
            //PK结束
        }
        @Override
        public void onMembersLeave(@NonNull List<? extends NERoomMember> members) {
            //直播间成员离开
        }
        @Override
        public void onMembersJoin(@NonNull List<? extends NERoomMember> members) {
            //直播间成员加入
        }

        @Override
        public void onError(int code) {
            //直播间错误
        }
    };
    NELiveKit.getInstance().setLiveListener(listener);
    ```

#### 注意事项

- 在SDK初始化成功后可以调用

--------------------

#### 注意事项

- 无

--------------------

### 注销

#### 描述

请求SDK注销当前已登录账号，返回未登录状态。

#### 业务流程

1. 调用接口并进行回调处理。该接口无额外回调结果数据，可根据错误码判断是否成功

    ```java
    NELiveKit.getInstance().logout(new NELiveCallback<Unit> {
        @Override
        public void onSuccess(Unit unit) {
            //TODO when logout success
        }
        public void onFailure(int code, String msg) {
            //TODO when logout fail
        }
    });
    ```

#### 注意事项

- 账号注销后，登录状态被清空，不再允许创建直播间

--------------------