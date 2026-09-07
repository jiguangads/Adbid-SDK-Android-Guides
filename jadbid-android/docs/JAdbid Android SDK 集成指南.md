# JAdbid Android SDK 集成指南

## 1. 环境要求

- Android API 23+
- Java 8+
- 普通包、公有云
- 主 AAR：`jadbid-android-{version}-release.aar`

## 2. 引入普通包

将主 AAR 放入宿主 `app/libs/`：

```gradle
dependencies {
    implementation files('libs/jadbid-android-{version}-release.aar')
    // 按实际渠道声明 TradPlus 与渠道 SDK；版本以发布清单为准。
}
```

普通包不内嵌宿主所需的渠道 Maven 依赖。仓库 `demo/jadbid-android-example-1.0.1-source.zip` 中的 `jadbid-android-example/standard/build.gradle` 是当前可编译参考，接入方应结合自身依赖治理锁定版本并处理重复依赖。

## 3. 外部配置

不要在代码或仓库中硬编码真实值。示例工程支持 Gradle property 或同名环境变量：

- `JADBID_APP_ID`
- `GOOGLE_ADMOB_APP_ID`
- `BANNER_AD_UNIT_ID`
- `INTERSTITIAL_AD_UNIT_ID`
- `REWARDED_VIDEO_AD_UNIT_ID`
- `NATIVE_AD_UNIT_ID`
- `SPLASH_AD_UNIT_ID`

例如在未纳入版本控制的用户级 `gradle.properties` 或 CI 密钥变量中配置。任一必需值为空时，示例会阻止初始化或请求并提示缺失的属性名。

若接入 AdMob，Manifest 中的 Application ID 必须来自安全配置；不要把真实 ID 写入库模块、示例源码、日志或文档。

## 4. 初始化与隐私

直接初始化：

```java
JAdConfig config = new JAdConfig.Builder(appId)
        .setDebugMode(BuildConfig.DEBUG)
        .setAdapterType(JAdapterType.TYPE_DEFAULT)
        .build();
JAdManager.getInstance().init(applicationContext, config);
```

需要先取得海外隐私同意时，先调用 `prepareInit(context, config)`，完成 UMP/地区合规流程且允许请求后再调用 `initWhenConsentReady()`。所有广告能力沿用同一隐私通道，也不应在同意前预加载。

## 5. 广告类型接入

### 5.1 开屏

```java
JSplashAd ad = new JSplashAd(splashAdUnitId);
ad.loadAd(activity, listener);
if (ad.isReady()) ad.showAd(container);
```

`loadAd` 不自动展示。宿主应设置有限超时，超时或用户放弃后调用 `destroy()` 并忽略迟到回调；参考实现为 5 秒。

## 6. 生命周期与线程

- 在 Activity/Fragment 销毁时调用对应广告实例的 `destroy()`。
- 不复用已经销毁的实例；需要新请求时创建或重新加载未销毁实例。
- `JSplashAd` 的公开回调由 JAdbid 派发到主线程；既有四类的回调线程取决于当前适配器实现。宿主的 Android View 操作应始终切回主线程。
- 不把广告位 ID、用户标识或第三方原始错误写入日志。

## 7. 混淆

```proguard
-keep class com.engagelab.ads.** { *; }
-keep interface com.engagelab.ads.** { *; }
```

渠道 SDK 的混淆规则以对应渠道官方要求和发布包 consumer rules 为准。

## 8. 验收

本地可验证 AAR、示例编译、API 签名和生命周期单测。真实填充、曝光、点击、关闭及隐私行为必须使用测试责任方安全提供的公有云配置在真机验证；未提供外部 ID 或设备时，不得把本地构建结果表述为真机广告验收通过。
