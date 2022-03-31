## 概述

网易会议iOS SDK提供了一套简单易用的接口，允许开发者通过调用NEMeeting SDK(以下简称SDK)提供的API，快速地集成音视频会议功能至现有iOS应用中。

## 快速接入

### 开发环境

| 名称        | 要求         |
| :---------- | :----------- |
| iOS版本     | 9.0以上      |
| CPU架构支持 | ARM64、ARMV7 |
| IDE         | XCode        |
| 其他        | CocoaPods    |

### SDK集成

1. 新建iOS工程

    a. 运行XCode，选择Create a new Xcode project，选择Single View App，选择Next。
    ![new android project](../../images/ios_new_project.png)

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
    pod 'NEMeetingSDK'
    ```

    执行pod命令，安装SDK

    ```
    pod install
    ```

3. 权限处理

    网易会议SDK正常工作需要摄像头、麦克风权限，用户需要在工程中的Info.list文件中配置相关的权限信息

    ![auth](../../images/auth.png)

    以上权限需要在进入会议之前由用户根据需要进行权限申请，权限申请的代码如下：

    ```objective-c
    //相机权限申请
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo 
     								         completionHandler:^(BOOL granted) {}];
    //麦克风权限申请
    [AVCaptureDevice requestAccessForMediaType:AVMediaTypeAudio 
                             completionHandler:^(BOOL granted) {}];
    ```

4. 渲染view注册。在info.plist文件中注册平台渲染视图，保证正常渲染。

    `io.flutter.embedded_views_preview     String       YES`

    ![viewconfig](../../images/viewconfig.png)

5. SDK初始化

    在使用SDK其他功能之前首先需要完成SDK初始化，初始化操作建议在**AppDelegate.m**的**application:didFinishLaunchingWithOptions:**方法执行。代码示例如下：
    ```objective-c
        NEMeetingSDKConfig *config = [[NEMeetingSDKConfig alloc] init];
        config.appKey = @"YOUR_APP_KEY";
        [[NEMeetingSDK getInstance] initialize:config
                                      callback:^(NSInteger resultCode, NSString *resultMsg, id result) {
            if (resultCode == ERROR_CODE_SUCCESS) {
                //TODO when initialize success
            } else {
                //TODO when initalize fail
            }
        }];
    ```

    **注意：其他操作一定更要等到初始化接口的回调返回之后再执行，否则会失败。**

6. 调用相关接口完成特定功能，详情请参考API文档。

  

## 业务开发

### 初始化

#### 描述

在使用SDK其他接口之前，首先需要完成初始化操作。

#### 业务流程

1. 配置初始化相关参数

```objective-c
NEMeetingSDKConfig *config = [[NEMeetingSDKConfig alloc] init];
config.appKey = [DemoConfig shareConfig].appKey; //应用APPKey
```

2. 调用接口并进行回调处理，该接口无额外回调结果数据

```objective-c
[[NEMeetingSDK getInstance] initialize:config
                          callback:^(NSInteger resultCode, NSString *resultMsg, id result) 
{
			if (resultCode == ERROR_CODE_SUCCESS) {
      		//初始化成功
      } else {
          //初始化失败
      }
}];
```

#### 注意事项

- 其他操作一定更要等到初始化接口的回调返回之后再执行，否则会失败

--------------------
