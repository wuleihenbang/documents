# 跑通Android示例项目

网易智慧企业 在 GitHub 上提供一个开源的互动直播组件示例项目 [NELiveKit](https://github.com/netease-kit/NELiveKit)。本文介绍如何快速跑通该示例项目，体验互动直播功能。示例代码中包含了详细的API调用场景、参数封装以及回调处理。

示例项目包含的功能如下：

- 通过账号、token完成NELiveKit登录鉴权；注销登录
- 创建直播间、加入直播间
- 直播间内提供的其他功能(如邀请PK等)

### 运行示例程序

开发者根据个人需求，补充完成示例项目的appKey后，即可运行并体验直播功能。

##  前提条件

在开始运行示例项目之前，请确保您已完成以下操作：
1. 创建应用并开通所需功能以及服务 [参考文档](../应用创建和服务开通.md)。
2. 在官网首页通过 QQ、在线消息或电话等方式联系商务经理，申请互动直播组件 App Key

## 开发环境

在开始运行示例项目之前，请确保开发环境满足以下要求：

| 环境要求         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| JDK 版本         | 1.8.0 及以上版本                                             |
| Android API 版本 | API 23、Android 6.0 及以上版本                               |
| CPU架构          | ARM64、ARMV7                                                 |
| IDE              | Android Studio                                               |
| 其他             | 依赖 Androidx，不支持 support 库。Android 系统 4.3 或以上版本的移动设备。 |

## 操作步骤
  1. 创建云信项目和获取 APPKEY
       - 参考文档 [应用创建和服务开通](../../../云信控制平台/应用创建和服务开通.md)

  2. 配置示例项目
       考以下步骤配置示例项目：
        - 克隆[NELiveKit](https://github.com/netease-kit/NELiveKit) 仓库至本地.
        - 找到`NELiveKit/SampleCode/Android` 示例项目文件夹，在 `app/src/main/res/values/strings.xml` 文件中填写你从云信控制台获取的AppKey
          ```
          <?xml version="1.0" encoding="utf-8"?>
                <resources>
                 <!--TODO-->
                 <!--Replace With Your AppKey Here-->
                 <string name="appkey">Your AppKey</string>
          </resources>
          ```

  3. 集成SDK说明并配置
	在app/build.gradle文件中已经添加了互动直播SDK的依赖
	  
    ```groovy
    android {
      // 添加 packagingOptions，否则可能会造成资源文件冲突
      packagingOptions {
        pickFirst 'lib/arm64-v8a/libc++_shared.so'
        pickFirst 'lib/armeabi-v7a/libc++_shared.so'
      }
    }

    dependencies {
	    //NEMeeting-SDK
	    implementation 'com.netease.yunxin.kit.live:livekit:1.0.0'
    }
	  ```

  1. 编译并运行示例项目

       连接上 Android 设备后，用 Android Studio 打开 `NELiveKit/SampleCode/Android`  示例项目，然后编译并运行示例项目。
  
## 示例项目登录账号

为了体验示例项目中的创建直播间功能，在以上完善应用Appkey的基础上，还需要使用互动直播账号完成SDK的登录鉴权。

互动直播SDK的登录鉴权需要提供一个有效的互动直播账号(可使用互动直播PaaS服务提供的创建账号接口完成创建)，并通过对应的账号ID和TOKEN来完成登录鉴权。

完成互动直播账号的登录实现后，运行示例项目，可体验完整的互动直播功能。

