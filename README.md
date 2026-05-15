# Adbid Android SDK

Adbid Android SDK 是移动广告聚合 SDK，帮助开发者快速集成多家广告源，实现竞价与瀑布流混合变现。SDK 对外暴露统一的广告加载/展示 API，内部通过适配器模式屏蔽各广告平台差异，接入方无需关心底层实现细节。

## 版本信息

| 项目     | 值                           |
| -------- | ---------------------------- |
| 当前版本 | 1.0.0                        |
| 最低支持 | minSdk 23                    |
| 目标编译 | compileSdk 34 / targetSdk 34 |
| 产物格式 | AAR                          |

## 支持的广告类型

| 类型         | 类名               | 说明                                     |
| ------------ | ------------------ | ---------------------------------------- |
| 横幅广告     | `JBannerAd`        | 页面内嵌入的 Banner 视图，支持自定义容器 |
| 插屏广告     | `JInterstitialAd`  | 全屏/半屏插屏，含视频插屏                |
| 激励视频广告 | `JRewardedVideoAd` | 用户完整观看后发放奖励                   |
| 原生广告     | `JNativeAd`        | 支持模板渲染与自渲染两种模式             |

## 支持的广告源

SDK 通过适配器聚合以下主流广告平台：

| 广告源                | 依赖                                                   |
| --------------------- | ------------------------------------------------------ |
| Google AdMob          | `com.google.android.gms:play-services-ads:24.9.0`      |
| Meta Audience Network | `com.facebook.android:audience-network-sdk:6.21.0`     |
| Pangle (穿山甲国际版) | `com.pangle.global:pag-sdk:7.8.5.8`                    |
| Unity Ads             | `com.unity3d.ads:unity-ads:4.16.5`                     |
| Mintegral             | `com.mbridge.msdk.oversea:mbridge_android_sdk:17.0.61` |
| ironSource            | `com.ironsource.sdk:mediationsdk:9.2.0`                |
| Bigo Ads              | `com.bigossp:bigo-ads:5.6.2`                           |
| Yandex Mobile Ads     | `com.yandex.android:mobileads:7.18.1`                  |
| InMobi                | `com.inmobi.monetization:inmobi-ads-kotlin:11.1.0`     |
| Fyber                 | `com.fyber:marketplace-sdk:8.4.1`                      |
| Xiaomi Ads            | `com.mi.ads:columbus-sdk:4.0.7.3`                      |
| Vungle                | `com.vungle:vungle-ads:7.6.3`                          |

## 仓库结构

```
jadbid-android-sdk/
├── jadbid-android/
│   ├── release/
│   │   └── jadbid-android-1.0.0-release.aar   # SDK 主产物
│   └── docs/
│       ├── 集成说明文档.md                       # 集成导航入口
│       └── API-接口说明.md                       # API 参考文档
└── demo/
    ├── jadbid-android-example/                  # Demo 源码工程
    └── jadbid-android-example-1.0.0-source.zip  # Demo 压缩包
```

## 快速集成

### 1. 添加 AAR

将 `jadbid-android-{version}-release.aar` 放入模块 `libs/` 目录，并在 `build.gradle` 中声明：

```groovy
dependencies {
    implementation fileTree(include: ['*.aar'], dir: 'libs')
}
```

### 2. 声明广告源依赖

根据业务需要，在模块 `build.gradle` 中添加对应广告平台的 Maven 依赖（参见上方「支持的广告源」表格）。

### 3. 初始化 SDK

在 `Application.onCreate()` 中准备初始化参数：

```java
JAdConfig config = new JAdConfig.Builder("YOUR_APP_ID")
        .setDebugMode(BuildConfig.DEBUG)
        .setAdapterType(JAdapterType.TYPE_DEFAULT)
        .build();

JAdManager.getInstance().prepareInit(this, config);
```

### 4. 海外隐私合规

海外场景需在隐私授权完成后再初始化：

```java
// Google UMP 授权完成后
if (consentInformation.canRequestAds()) {
    JAdManager.getInstance().initWhenConsentReady();
}
```

国内场景可直接初始化：

```java
JAdManager.getInstance().init(this, config);
```

### 5. 加载与展示广告

以横幅广告为例：

```java
JBannerAd bannerAd = new JBannerAd("YOUR_AD_UNIT_ID");
bannerAd.loadAd(activity, new JBannerAdListener() {
    @Override
    public void onAdLoaded(JAdInfo adInfo) {
        View bannerView = bannerAd.getBannerView();
        if (bannerView != null) {
            container.removeAllViews();
            container.addView(bannerView);
        }
    }

    @Override
    public void onAdLoadFailed(JAdError error) {
        // 处理加载失败
    }

    // ... 其他回调
});
```

激励视频与插屏广告需先 `loadAd()`，成功后调用 `showAd()` 展示：

```java
JRewardedVideoAd rewardedAd = new JRewardedVideoAd("YOUR_AD_UNIT_ID");
rewardedAd.loadAd(activity, new JRewardedVideoAdListener() {
    @Override
    public void onAdLoaded(JAdInfo adInfo) {
        if (rewardedAd.isReady()) {
            rewardedAd.showAd(activity);
        }
    }

    @Override
    public void onAdRewarded(JAdInfo adInfo) {
        // 发放奖励
    }

    // ... 其他回调
});
```

## 海外隐私合规

SDK 提供 GDPR / CCPA / LGPD 等隐私合规 API：

```java
JAdManager manager = JAdManager.getInstance();

// 个性化广告开关（默认开启）
manager.setOpenPersonalizedAd(true);

// CCPA：false 表示加州用户不上报数据
manager.setCCPADoNotSell(context, false);

// LGPD：0 允许上报，1 不允许
manager.setLGPDConsent(context, 0);

// 按地区自动设置
manager.checkCurrentArea(context, new PrivacyRegionListener() {
    @Override
    public void onSuccess(boolean isEu, boolean isCn, boolean isCalifornia, boolean isBr) {
        if (isCalifornia) manager.setCCPADoNotSell(context, false);
        if (isBr) manager.setLGPDConsent(context, 0);
    }

    @Override
    public void onFailed() { /* fallback */ }
});
```

## Demo 工程

仓库包含完整的示例工程，覆盖所有广告类型的接入流程：

- **源码位置**：`demo/jadbid-android-example/`
- **模块**：`standard`（标准包接入方式）
- **构建**：`./gradlew :standard:assembleDebug`

Demo 中广告 ID 通过 `build.gradle` 的 `buildConfigField` 配置，可替换为你的正式 ID。

## 文档索引

| 文档                                                | 说明                  |
| --------------------------------------------------- | --------------------- |
| [集成说明文档](jadbid-android/docs/集成说明文档.md) | 集成导航与快速说明    |
| [API 接口说明](jadbid-android/docs/API-接口说明.md) | 完整 API 参考与错误码 |

## 错误码

| 错误码 | 常量                           | 说明           |
| ------ | ------------------------------ | -------------- |
| 1001   | `ERROR_CODE_NO_FILL`           | 无广告填充     |
| 1002   | `ERROR_CODE_NETWORK`           | 网络错误       |
| 1003   | `ERROR_CODE_TIMEOUT`           | 超时           |
| 1004   | `ERROR_CODE_NOT_INIT`          | 未初始化       |
| 1005   | `ERROR_CODE_INVALID_PARAMS`    | 参数无效       |
| 1006   | `ERROR_CODE_AD_LOADING`        | 广告正在加载中 |
| 1007   | `ERROR_CODE_AD_NOT_READY`      | 广告未准备好   |
| 1008   | `ERROR_CODE_INTERNAL`          | 内部错误       |
| 1009   | `ERROR_CODE_ADAPTER_NOT_FOUND` | 适配器未找到   |
| 9999   | `ERROR_CODE_UNKNOWN`           | 未知错误       |
