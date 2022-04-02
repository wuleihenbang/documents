## 概述

互动直播iOS SDK提供了一套简单易用的接口，允许开发者通过调用NELiveKit SDK(以下简称SDK)提供的API，快速地集成互动直播功能至现有iOS应用中。

## 快速接入

### 开发环境

| 名称        | 要求         |
| :---------- | :----------- |
| iOS版本     | 10.0以上      |
| CPU架构支持 | ARM64、ARMV7 |
| IDE         | XCode        |
| 其他        | CocoaPods    |

### SDK集成

1. 新建iOS工程

    a. 运行XCode，选择Create a new Xcode project，选择Single View App，选择Next。
    ![new ios project](../../images/ios_new_project.png)

    b. 配置工程相关信息，选择Next。
    ![configure project](../../images/ios_configure_project.png)

    c. 然后选择合适的工程本地路径，选择Create完成工程创建。

2. 通过CocoaPods集成SDK

    进入到工程路径执行pod命令，生成Podfile文件，注意CocoaPods版本使用1.9.1以上的，防止因为版本过低导致无法拉取sdk。
    ```groovy
    pod init
    ```
    打开Podfile文件添加如下代码，保存。

    ```
    pod 'NELiveKit'
    ```

    执行pod命令，安装SDK

    ```
    pod install
    ```

3. 权限处理

    NELiveKit应用正常工作需要摄像头、麦克风权限，用户需要在工程中的Info.list文件中配置相关的权限信息

    ![auth](../../images/auth.png)

    以上权限需要在进入直播之前由用户根据需要进行权限申请，权限申请的代码如下：

    ```objective-c
    //相机权限申请
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo 
     								         completionHandler:^(BOOL granted) {}];
    //麦克风权限申请
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeAudio 
                             completionHandler:^(BOOL granted) {}];
    ```

4. SDK初始化

    在使用SDK其他功能之前首先需要完成SDK初始化，初始化操作建议在**AppDelegate.m**的**application:didFinishLaunchingWithOptions**方法执行。代码示例如下：

    ```objective-c
        NELiveKitOptions *o = [[NELiveKitOptions alloc] init];
        o.appKey = kAppKey;

        [[NELiveKit shared] initializeWithOptions:o callback:^(NSInteger code, NSString * _Nullable msg) {
        
        }];
    ```

    **注意：其他操作一定更要等到初始化接口的回调返回之后再执行，否则会失败。**

5. 调用相关接口完成特定功能，详情请参考API文档。

- [登录鉴权](#登录鉴权)
    ```objective-c
    //Token登录
    [[NELiveKit shared] loginWithAccount:accountId token:accessToken callback:^(NSInteger code, NSString * _Nullable msg) {

    }];
    
    ```
- [开始直播](#创建直播)
    ```objective-c
    [[NELiveKit shared] startLiveWithLiveTopic:topic liveType:NEliveRoomTypePkLive nickName:nickName cover:cover callback:^(NSInteger code, NSString * _Nullable msg, NELiveDetail * _Nullable detail) {

    }];
    ```
- [退出直播](#停止直播)
    ```objective-c
    [[NELiveKit shared] stopLiveWithCallback:^(NSInteger code, NSString * _Nullable msg) {
    }];
    ```
  
- [加入其他直播间](#加入其他直播间)
    ```objective-c
    [[NELiveKit shared] joinLiveWithLive:info callback:^(NSInteger code, NSString * _Nullable msg, NSString * _Nullable rtmpPullUrl) {

    }];
    ```
- [退出其他直播间](#退出其他直播间)
    ```objective-c
    [[NELiveKit shared] leaveLiveWithCallback:^(NSInteger code, NSString * _Nullable msg) {
    }];
    ```
- [注销登录](#注销)
    ```objective-c
    [[NELiveKit shared] logoutWithCallback:^(NSInteger code, NSString * _Nullable msg) {
    }];
    ```

## 业务开发

### 初始化

#### 描述

在使用SDK其他接口之前，首先需要完成初始化操作。

#### 业务流程


调用接口并进行回调处理，该接口无额外回调结果数据

```objective-c
    NELiveKitOptions *o = [[NELiveKitOptions alloc] init];
    o.appKey = kAppKey;

    [[NELiveKit shared] initializeWithOptions:o callback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when initialize success
        } else {
            // TODO: when initialize fail
        }
    }];
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

    ```objective-c
    NSString *accountId = @"accountId";
    NSString *accountToken = @"accountToken";
    ```

2. 登录并进行回调处理，该接口无额外回调结果数据

    ```objective-c
    [[NELiveKit shared] loginWithAccount:accountId token:accountToken callback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            //TODO: when login success
        } else {
            //TODO: when login fail
        }
    }];
    ```

#### 注意事项

- SDK不提供账号注册机制，第三方应用集成SDK时需要为第三方应用的用户帐号绑定互动直播系统中的帐号

--------------------

### 创建直播间

#### 描述

在已经完成SDK登录鉴权的状态下，创建并开始一个新的直播间。

#### 业务流程

调用创建直播接口并进行回调处理。

```objective-c
    [[NELiveKit shared] startLiveWithLiveTopic:topic liveType:NEliveRoomTypePkLive nickName:nickName cover:cover callback:^(NSInteger code, NSString * _Nullable msg, NELiveDetail * _Nullable detail) {
        if (code == 0) {
            // TODO: when startLive success
        } else {
            // TODO: when startLive fail
        }
    }];
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

```objective-c
    [[NELiveKit shared] stopLiveWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when stopLive success
        } else {
            // TODO: when stopLive fail
        }
    }];
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

```objective-c
    [[NELiveKit shared] joinLiveWithLive:info callback:^(NSInteger code, NSString * _Nullable msg, NSString * _Nullable rtmpPullUrl) {
        if (code == 0) {
            // TODO: when joinLive success
        } else {
            // TODO: when joinLive fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 离开直播间

#### 描述

在已登录的状态下，离开一个当前正在进行中的直播间。

#### 业务流程

调用离开直播间接口并进行回调处理。

```objective-c
    [[NELiveKit shared] leaveLiveWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when leaveLive success
        } else {
            // TODO: when leaveLive fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------


### 邀请其他主播PK

#### 描述

在已开播的状态下，邀请其他主播进行PK。

#### 业务流程

调用邀请其他主播PK接口并进行回调处理。

```objective-c
    [[NELiveKit shared] invitePKWithTargetAccountId:userUuid rule:rule callback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when invite pk success
        } else {
            // TODO: when invite pk fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------


### 取消邀请其他主播PK

#### 描述

在已开播的状态下，取消邀请其他主播进行PK。

#### 业务流程

调用取消邀请其他主播PK接口并进行回调处理。

```objective-c
    [[NELiveKit shared] cancelPKInviteWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when invite canceled success
        } else {
            // TODO: when invite canceled fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 接受其他主播的PK邀请

#### 描述

在已开播的状态下，接受其他主播的PK邀请。

#### 业务流程

调用接受其他主播的PK邀请接口并进行回调处理。

```objective-c
    [[NELiveKit shared] acceptPKWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when accept invite success
        } else {
            // TODO: when accept invite fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 拒接其他主播的PK邀请

#### 描述

在已开播的状态下，拒绝其他主播的PK邀请。

#### 业务流程

调用拒接其他主播的PK邀请接口并进行回调处理。

```objective-c
    [[NELiveKit shared] rejectPKWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when reject invite success
        } else {
            // TODO: when reject invite fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 给主播打赏礼物

#### 描述

在加入其他主播的直播间的状态下，给该主播进行打赏。

#### 业务流程

调用给主播打赏礼物接口并进行回调处理。

```objective-c
    [[NELiveKit shared] rewardWithGiftId:gift.giftId callback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when reward success
        } else {
            // TODO: when reward fail
        }
    }];
```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 获取直播间成员列表

#### 描述

在加入其他主播的直播间或者自己开播的状态下，获取成员列表。

#### 业务流程

获取直播间成员列表。

```objective-c
    [NELiveKit shared].members
```

#### 注意事项

- 该接口仅支持直播间内调用
--------------------

### 查询当前直播间是否在PK

#### 描述

在加入其他主播的直播间或者自己开播的状态下，查询是否PK。

#### 业务流程

获取直播间PK状态。

```objective-c
    [NELiveKit shared].isPKing
```

#### 注意事项

- 该接口仅支持直播间内调用
--------------------

### 查询直播间信息

#### 描述

在已登录状态下，查询一个直播间信息

#### 业务流程

1. 调用查询直播间信息接口并进行回调处理，可根据错误码判断是否成功

    ```objective-c
    [[NELiveKit shared] fetchLiveInfoWithLiveRecordId:liveRecordId callback:^(NSInteger code, NSString * _Nullable msg, NELiveDetail * _Nullable info) {
        if (code == 0) {
            // TODO: when fetch live info success
        } else {
            // TODO: when fetch live info fail
        }
    }];
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用
--------------------

### 查询直播间列表

#### 描述

在已登录状态下，查询直播间列表

#### 业务流程

1. 调用查询直播间列表接口并进行回调处理，可根据错误码判断是否成功

    ```objective-c
    [[NELiveKit shared] fetchLiveListWithPageNum:_page pageSize:20 liveStatus:NELiveStatusLiving liveType:NEliveRoomTypePkLive callback:^(NSInteger code, NSString * _Nullable msg, NELiveList * _Nullable list) {
        if (code == 0) {
            // TODO: when fetch live list success
        } else {
            // TODO: when fetch live list fail
        }
    }];
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用

--------------------

### 直播间内的操作

#### 描述

在直播间内，可以做切换摄像头、开关音频、开关视频等操作

#### 业务流程

1. 调用查询直播间列表接口并进行回调处理，可根据错误码判断是否成功

    ```objective-c
   //开启预览
   [[NELiveKit shared].liveMediaController startPreviewWithCanvas:localRender];
   //切换摄像头
   [[NELiveKit shared].liveMediaController switchCamera];
   //开关音频
    [[NELiveKit shared].liveMediaController muteLiveAudioWithCallback:^(NSInteger code, NSString * _Nullable msg) {
                                
    }];
    [[NELiveKit shared].liveMediaController unmuteLiveAudioWithCallback:^(NSInteger code, NSString * _Nullable msg) {
                                
    }];
   //开关视频
    [[NELiveKit shared].liveMediaController muteLiveVideo];
    [[NELiveKit shared].liveMediaController unmuteLiveVideo];
   //在自己的直播间静音PK的主播
    [[NELiveKit shared].liveMediaController mutePKAudioWithCallback:^(NSInteger code, NSString * _Nullable msg) {
    }];

    [[NELiveKit shared].liveMediaController unmutePKAudioWithCallback:^(NSInteger code, NSString * _Nullable msg) {
    }];
   //绑定PK时，对方的画布
   [[NELiveKit shared].liveMediaController setupRemoteViewWithCanvas:canvas];
    ```

#### 注意事项

- 该接口仅支持登录和未登录状态调用

--------------------

### 监听直播间状态

#### 描述

通过注册直播间状态回调接口，可获取到直播间状态变更的通知。

#### 业务流程

1. 设置直播间监听器，并在回调方法中处理感兴趣的事件

    ```objective-c
    [NELiveKit shared].listener = self;

    // 登录被踢出
    - (void)onLoginKickOut {
    
    }
    // 直播结束
   - (void)onLiveEndWithReason:(NSInteger)reason {
    
    }
    // 有观众加入
   - (void)onMembersJoinWithMembers:(NSArray<NERoomMember *> *)members {
    
    }
    // 有观众离开
   - (void)onMembersLeaveWithMembers:(NSArray<NERoomMember *> *)members {
    
    }
    // 收到PK邀请
   - (void)onPKInvitedWithActionAnchor:(NELivePKAnchor *)actionAnchor {
    
    }
    // PK超时
   - (void)onPKTimeoutWithActionAnchor:(NELivePKAnchor *)actionAnchor {
    
    }
    // 对方接受PK
   - (void)onPKAcceptedWithActionAnchor:(NELivePKAnchor *)actionAnchor {
    
    }
    // PK邀请取消
   - (void)onPKCanceledWithActionAnchor:(NELivePKAnchor *)actionAnchor {
    
    }
    // PK邀请被拒绝
   - (void)onPKRejectedWithActionAnchor:(NELivePKAnchor *)actionAnchor {
    
    }
    // PK开始
   - (void)onPKStartWithPkStartTime:(NSInteger)pkStartTime pkCountDown:(NSInteger)pkCountDown inviter:(NELivePKAnchor *)inviter invitee:(NELivePKAnchor *)invitee {
    
    }
    // PK结束
   - (void)onPKEndWithReason:(NSInteger)reason pkEndTime:(NSInteger)pkEndTime inviterRewards:(NSInteger)inviterRewards inviteeRewards:(NSInteger)inviteeRewards countDownEnd:(BOOL)countDownEnd {
    
    }
    // PK惩罚开始
   - (void)onPKPunishingStartWithPkPenaltyCountDown:(NSInteger)pkPenaltyCountDown inviterRewards:(NSInteger)inviterRewards inviteeRewards:(NSInteger)inviteeRewards {
    
    }
    // 收到打赏
   - (void)onRewardReceivedWithRewarderUserUuid:(NSString *)rewarderUserUuid rewarderUserName:(NSString *)rewarderUserName giftId:(NSInteger)giftId anchorReward:(NELiveAnchorReward *)anchorReward otherAnchorReward:(NELiveAnchorReward *)otherAnchorReward {
    
    }
    // 收到聊天室消息
   - (void)onMessagesReceivedWithMessages:(NSArray<NERoomMessage *> *)messages {
    
    }
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

    ```objective-c
    [[NELiveKit shared] logoutWithCallback:^(NSInteger code, NSString * _Nullable msg) {
        if (code == 0) {
            // TODO: when logout success
        } else {
            // TODO: when logout fail
        }
    }];
    ```

#### 注意事项

- 账号注销后，登录状态被清空，不再允许创建直播间

--------------------