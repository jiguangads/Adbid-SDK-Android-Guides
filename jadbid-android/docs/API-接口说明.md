# JAdbid SDK 公开 API

本文以 `ad-core/src/main/java/com/engagelab/ads` 源码为准。当前公开支持横幅、插屏、激励视频、原生和开屏 5 类广告。

## 1. 初始化

### JAdManager

| 方法 | 说明 |
| --- | --- |
| `static JAdManager getInstance()` | 获取单例 |
| `void init(Context, JAdConfig)` | 直接初始化 |
| `void prepareInit(Context, JAdConfig)` | 保存配置但不初始化适配器 |
| `void initWhenConsentReady()` | 隐私条件满足后完成初始化 |
| `static boolean isPrepared()` | 是否已 prepare 且尚未初始化 |
| `static boolean isInitialized()` | 是否已初始化 |
| `static boolean waitForInitialization(long)` | 在给定毫秒内等待初始化 |
| `static String getVersion()` | 获取 SDK 版本 |
| `static JAdConfig getConfig()` | 获取当前配置 |
| `static IAdAdapter getAdapterInstance()` | 获取当前根适配器 |
| `void setDebugMode(boolean)` | 切换调试模式 |
| `static String getISO()` | 获取当前 ISO 国家码 |
| `static JAdInfo getAdInfo(String)` | 查询缓存的广告元数据 |
| `void setMaxDatabaseSize(long)` | 设置既有本地数据库容量上限 |

隐私相关既有方法：

```java
void setOpenPersonalizedAd(boolean open)
void setPrivacyUserAgree(boolean agree)
void setCCPADoNotSell(Context context, boolean doNotSell)
void setLGPDConsent(Context context, int consent)
void checkCurrentArea(Context context, PrivacyRegionListener listener)
boolean isFirstShowGDPR(Context context)
void setIsFirstShowGDPR(Context context, boolean firstShow)
void setGDPRListener(JGDPRListener listener)
void setPrivacyListener(JPrivacyListener listener)
void setAllowMessagePush(boolean allow)
boolean isAllowMessagePush()
```

所有广告能力沿用同一初始化与隐私状态，不创建独立同意通道。各参数含义应结合适用地区政策和宿主应用的隐私合规流程使用。

## 2. 既有四类广告

| 类型 | 构造 | 主要方法 |
| --- | --- | --- |
| 横幅 `JBannerAd` | `JBannerAd(String adUnitId)` | `loadAd(Activity, JBannerAdListener)`、`getBannerView()`、`destroy()` |
| 插屏 `JInterstitialAd` | `JInterstitialAd(String adUnitId)` | `loadAd(Activity, JInterstitialAdListener)`、`showAd(Activity)`、`isReady()`、`destroy()` |
| 激励视频 `JRewardedVideoAd` | `JRewardedVideoAd(String adUnitId)` | `loadAd(Activity, JRewardedVideoAdListener)`、`showAd(Activity)`、`isReady()`、`destroy()` |
| 原生 `JNativeAd` | `JNativeAd(String adUnitId)` | `loadAd(Activity, JNativeAdListener)`、`isReady()`、`showAd(ViewGroup, int)`、`getNativeAdObject()`、`destroy()` |

既有四类签名保持不变。完整字段模型见 `JAdInfo`，失败对象见 `JAdError`。

对应回调：

```java
// JBannerAdListener / JNativeAdListener
void onAdLoaded(JAdInfo adInfo)
void onAdLoadFailed(JAdError error)
void onAdShown(JAdInfo adInfo)
void onAdClicked(JAdInfo adInfo)
void onAdClosed(JAdInfo adInfo)

// JInterstitialAdListener 另有
void onAdShowFailed(JAdError error)
void onVideoPlayStart(JAdInfo adInfo)
void onVideoPlayEnd(JAdInfo adInfo)

// JRewardedVideoAdListener 另有
void onAdShowFailed(JAdError error)
void onAdRewarded(JAdInfo adInfo)
void onVideoPlayStart(JAdInfo adInfo)
void onVideoPlayEnd(JAdInfo adInfo)
```

## 3. 开屏广告

### JSplashAd

```java
JSplashAd(String adUnitId)
void loadAd(Activity activity, JSplashAdListener listener)
boolean isReady()
void showAd(ViewGroup container)
void destroy()
```

`loadAd` 仅预加载，不自动展示。`showAd` 只接收容器，不接收 Activity。广告位为空会抛 `IllegalArgumentException`；SDK 未初始化或根适配器不支持开屏时，构造会抛 `IllegalStateException`。

### JSplashAdListener

```java
void onAdLoaded(JAdInfo adInfo)
void onAdLoadFailed(JAdError error)
void onAdShown(JAdInfo adInfo)
void onAdClicked(JAdInfo adInfo)
void onAdClosed(JAdInfo adInfo)
void onAdShowFailed(JAdError error)
```

宿主必须自行控制开屏等待上限和后续页面跳转；超时、主动放弃或页面销毁后调用 `destroy()`。

## 4. 广告信息

回调中的广告元数据统一使用 `JAdInfo`，不公开具体平台对象。常用字段包括：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `tpAdUnitId` | String | 聚合广告位 ID |
| `true_adunit_id` | String | 底层实际广告位 ID |
| `adSourceName` | String | 广告源名称 |
| `ecpm` / `ecpmcny` | String | 美元/人民币 eCPM |
| `ecpmLevel` | String | 价格档位 |
| `rewardName` / `rewardNumber` | String / int | 激励名称与数量 |
| `isoCode` | String | 国家或地区 ISO 码 |
| `requestId` / `impressionId` | String | 请求与曝光标识 |
| `isBiddingNetwork` | boolean | 是否竞价广告 |
| `waterfallIndex` | int | 瀑布流索引 |

字段可能为空或为默认值，使用前应判空。业务日志不得直接打印整对象、广告位、请求标识或其他潜在敏感字段。

## 5. 生命周期与开屏状态语义

- 使用任意 `J*Ad` 前先完成 `JAdManager` 初始化。
- 各类型展示入口不同：横幅通过 `getBannerView()` 取 View；插屏和激励视频使用 `showAd(Activity)`；原生使用 `showAd(ViewGroup, int)`；开屏使用 `showAd(ViewGroup)`。
- 页面离开或不再使用广告实例时调用对应的 `destroy()`；既有四类的回调线程和销毁竞态语义取决于当前适配器实现，宿主操作 Android View 时仍须回到主线程。
- `JSplashAd` 的公开回调由 JAdbid 派发到主线程；同一实例再次 `loadAd` 会使旧请求回调失效，`destroy()` 幂等且销毁后不再加载、展示或派发回调。
- `JSplashAd` 的 Activity 或容器为空时通过失败回调报告参数错误，广告未就绪时通过 `onAdShowFailed` 报告。
- 业务日志不得直接记录第三方异常、广告位或用户标识。

## 6. 错误码

| 常量 | 值 | 说明 |
| --- | ---: | --- |
| `ERROR_CODE_NO_FILL` | 1001 | 无填充 |
| `ERROR_CODE_NETWORK` | 1002 | 网络错误 |
| `ERROR_CODE_TIMEOUT` | 1003 | 超时 |
| `ERROR_CODE_NOT_INIT` | 1004 | 未初始化 |
| `ERROR_CODE_INVALID_PARAMS` | 1005 | 参数无效 |
| `ERROR_CODE_AD_LOADING` | 1006 | 正在加载 |
| `ERROR_CODE_AD_NOT_READY` | 1007 | 未准备好 |
| `ERROR_CODE_INTERNAL` | 1008 | 内部错误 |
| `ERROR_CODE_ADAPTER_NOT_FOUND` | 1009 | 未找到适配器 |
| `ERROR_CODE_UNKNOWN` | 9999 | 未知错误 |
