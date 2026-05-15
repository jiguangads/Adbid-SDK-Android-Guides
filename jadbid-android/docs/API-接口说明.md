# JAdbid SDK 对外 API 接口说明

本文档描述 `com.engagelab.ads.api` 包下对外暴露的类及其公开方法，供接入方查阅。

---

## 1. JAdManager

**包名**：`com.engagelab.ads.api.JAdManager`  
**说明**：JAdbid SDK 初始化与全局管理，单例。负责初始化适配器、两阶段初始化（海外隐私）、以及海外隐私相关 API 的转发。

### 1.1 获取实例

| 方法 | 说明 |
|------|------|
| `static JAdManager getInstance()` | 获取单例。 |

### 1.2 初始化

| 方法 | 说明 |
|------|------|
| `void init(Context context, JAdConfig config)` | 直接初始化 SDK。若已初始化则忽略；若此前调用了 `prepareInit` 则清除 prepare 状态并执行初始化。 |
| `void prepareInit(Context context, JAdConfig config)` | 仅保存 context、config，不执行 adapter 初始化。用于海外隐私：先 prepare，完成 UMP 等后再调 `initWhenConsentReady()`。 |
| `void initWhenConsentReady()` | 隐私就绪后再初始化。若已 `prepareInit` 且未初始化则执行与 `init` 相同的逻辑；否则 no-op。 |
| `static boolean isPrepared()` | 是否已 prepare 且尚未初始化。 |
| `static boolean isInitialized()` | SDK 是否已初始化。 |
| `static boolean waitForInitialization(long timeoutMs)` | 等待初始化完成。`timeoutMs` 超时毫秒，0 表示不等待。返回是否在超时前完成。 |

### 1.3 配置与版本

| 方法 | 说明 |
|------|------|
| `static String getVersion()` | 获取 SDK 版本号。 |
| `static JAdConfig getConfig()` | 获取当前配置（初始化后有效）。 |
| `static IAdAdapter getAdapterInstance()` | 获取当前适配器实例（供内部或高级用法）。 |
| `void setDebugMode(boolean debugMode)` | 设置调试模式。 |
| `static String getISO()` | 获取设备当前 ISO 国家码（如 `"US"`、`"CN"`）；SDK 未初始化时返回 null。 |
| `static JAdInfo getAdInfo(String adUnitId)` | 按广告位 ID 获取已缓存的广告元数据；未缓存或 SDK 未初始化时返回 null。 |
| `void setMaxDatabaseSize(long maxSize)` | 设置 SDK 本地数据库最大容量（字节），如 `50 * 1024 * 1024L` 表示 50 MB。 |

### 1.4 海外隐私 API

以下方法委托给当前适配器，用于 GDPR/CCPA/LGPD 等合规。

| 方法 | 说明 |
|------|------|
| `void setOpenPersonalizedAd(boolean open)` | 个性化推荐广告开关；默认开启。 |
| `void setPrivacyUserAgree(boolean agree)` | 隐私信息控制开关；默认开启。 |
| `void setCCPADoNotSell(Context context, boolean doNotSell)` | CCPA 加州「不出售」：false 表示加州用户不上报数据，true 表示接受上报。 |
| `void setLGPDConsent(Context context, int consent)` | LGPD 巴西同意：0 允许上报设备数据，1 不允许。 |
| `void checkCurrentArea(Context context, PrivacyRegionListener listener)` | 查询当前地区（欧/中/加州/巴西），用于在初始化前设置 CCPA/LGPD。 |
| `boolean isFirstShowGDPR(Context context)` | 是否已做过 GDPR 选择。 |
| `void setIsFirstShowGDPR(Context context, boolean firstShow)` | 标记已展示过 GDPR 选择。 |
| `void setGDPRListener(JGDPRListener listener)` | 设置 GDPR 授权弹窗结果监听；传 null 则清除监听。 |
| `void setPrivacyListener(JPrivacyListener listener)` | 设置隐私授权弹窗结果监听；传 null 则清除监听。 |
| `void setAllowMessagePush(boolean allow)` | 设置是否允许消息推送。 |
| `boolean isAllowMessagePush()` | 查询是否允许消息推送。 |

---

## 2. JBannerAd（横幅广告）

**包名**：`com.engagelab.ads.api.JBannerAd`  
**说明**：横幅广告 API。构造时要求 SDK 已初始化（最多等待 5 秒，超时抛 `IllegalStateException`）。

### 2.1 构造

| 方法 | 说明 |
|------|------|
| `JBannerAd(String adUnitId)` | 使用广告位 ID 创建实例。 |

### 2.2 方法

| 方法 | 说明 |
|------|------|
| `void loadAd(Activity activity, JBannerAdListener listener)` | 加载横幅广告。 |
| `View getBannerView()` | 获取 Banner 的 View，需加载成功后才有内容。 |
| `void destroy()` | 销毁广告。 |

### 2.3 回调接口 JBannerAdListener

| 方法 | 说明 |
|------|------|
| `void onAdLoaded(JAdInfo adInfo)` | 广告加载成功。 |
| `void onAdLoadFailed(JAdError error)` | 广告加载失败。 |
| `void onAdShown(JAdInfo adInfo)` | 广告展示。 |
| `void onAdClicked(JAdInfo adInfo)` | 广告点击。 |
| `void onAdClosed(JAdInfo adInfo)` | 广告关闭。 |

---

## 3. JInterstitialAd（插屏广告）

**包名**：`com.engagelab.ads.api.JInterstitialAd`  
**说明**：插屏广告 API。构造时要求 SDK 已初始化（最多等待 5 秒，超时抛 `IllegalStateException`）。

### 3.1 构造

| 方法 | 说明 |
|------|------|
| `JInterstitialAd(String adUnitId)` | 使用广告位 ID 创建实例。 |

### 3.2 方法

| 方法 | 说明 |
|------|------|
| `void loadAd(Activity activity, JInterstitialAdListener listener)` | 加载插屏广告。 |
| `void showAd(Activity activity)` | 展示插屏广告，需先加载成功且 `isReady()` 为 true。 |
| `boolean isReady()` | 是否已加载并可展示。 |
| `void destroy()` | 销毁广告。 |

### 3.3 回调接口 JInterstitialAdListener

| 方法 | 说明 |
|------|------|
| `void onAdLoaded(JAdInfo adInfo)` | 广告加载成功。 |
| `void onAdLoadFailed(JAdError error)` | 广告加载失败。 |
| `void onAdShown(JAdInfo adInfo)` | 广告展示。 |
| `void onAdClicked(JAdInfo adInfo)` | 广告点击。 |
| `void onAdClosed(JAdInfo adInfo)` | 广告关闭。 |
| `void onAdShowFailed(JAdError error)` | 广告展示失败。 |
| `void onVideoPlayStart(JAdInfo adInfo)` | 视频播放开始（视频插屏）。 |
| `void onVideoPlayEnd(JAdInfo adInfo)` | 视频播放结束（视频插屏）。 |

---

## 4. JRewardedVideoAd（激励视频广告）

**包名**：`com.engagelab.ads.api.JRewardedVideoAd`  
**说明**：激励视频广告 API。构造时要求 SDK 已初始化（最多等待 5 秒，超时抛 `IllegalStateException`）。

### 4.1 构造

| 方法 | 说明 |
|------|------|
| `JRewardedVideoAd(String adUnitId)` | 使用广告位 ID 创建实例。 |

### 4.2 方法

| 方法 | 说明 |
|------|------|
| `void loadAd(Activity activity, JRewardedVideoAdListener listener)` | 加载激励视频广告。 |
| `void showAd(Activity activity)` | 展示激励视频，需先加载成功且 `isReady()` 为 true。 |
| `boolean isReady()` | 是否已加载并可展示。 |
| `void destroy()` | 销毁广告。 |

### 4.3 回调接口 JRewardedVideoAdListener

| 方法 | 说明 |
|------|------|
| `void onAdLoaded(JAdInfo adInfo)` | 广告加载成功。 |
| `void onAdLoadFailed(JAdError error)` | 广告加载失败。 |
| `void onAdShown(JAdInfo adInfo)` | 广告展示。 |
| `void onAdClicked(JAdInfo adInfo)` | 广告点击。 |
| `void onAdClosed(JAdInfo adInfo)` | 广告关闭。 |
| `void onAdShowFailed(JAdError error)` | 广告展示失败。 |
| `void onAdRewarded(JAdInfo adInfo)` | 激励发放（用户完整观看）。奖励信息从 `adInfo.rewardName` / `adInfo.rewardNumber` 读取。 |
| `void onVideoPlayStart(JAdInfo adInfo)` | 视频播放开始。 |
| `void onVideoPlayEnd(JAdInfo adInfo)` | 视频播放结束。 |

---

## 5. JNativeAd（原生广告）

**包名**：`com.engagelab.ads.api.JNativeAd`  
**说明**：原生广告 API，支持模板渲染和自渲染两种方式。构造时要求 SDK 已初始化（最多等待 5 秒，超时抛 `IllegalStateException`）。

### 5.1 构造

| 方法 | 说明 |
|------|------|
| `JNativeAd(String adUnitId)` | 使用广告位 ID 创建实例。 |

### 5.2 方法

| 方法 | 说明 |
|------|------|
| `void loadAd(Activity activity, JNativeAdListener listener)` | 加载原生广告。 |
| `boolean isReady()` | 是否已加载并可展示。 |
| `void showAd(ViewGroup container, int layoutResId)` | **模板渲染**：传入容器和布局 ID，由底层 SDK 填充素材。调用前请先通过 `isReady()` 确认就绪。 |
| `Object getNativeAdObject()` | **自渲染**：获取底层原生广告对象，由开发者自行拼接 UI。具体类型由适配器决定。 |
| `void destroy()` | 销毁广告，释放资源。 |

### 5.3 回调接口 JNativeAdListener

| 方法 | 说明 |
|------|------|
| `void onAdLoaded(JAdInfo adInfo)` | 广告加载成功。 |
| `void onAdLoadFailed(JAdError error)` | 广告加载失败。 |
| `void onAdShown(JAdInfo adInfo)` | 广告展示。 |
| `void onAdClicked(JAdInfo adInfo)` | 广告点击。 |
| `void onAdClosed(JAdInfo adInfo)` | 广告关闭。 |

---

## 6. JAdInfo（广告信息）

**包名**：`com.engagelab.ads.bean.JAdInfo`  
**说明**：广告回调中携带的广告元数据，不暴露具体平台类型。所有回调方法的 `adInfo` 参数均为此类型（可能为 null）。

### 常用字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `tpAdUnitId` | String | TradPlus 广告位 ID |
| `true_adunit_id` | String | 实际展示的底层广告位 ID |
| `adSourceName` | String | 广告源名称（如 "admob"、"facebook"） |
| `ecpm` | String | eCPM（美元） |
| `ecpmcny` | String | eCPM（人民币） |
| `ecpmLevel` | String | eCPM 价格档位 |
| `rewardName` | String | 激励名称（激励视频专用） |
| `rewardNumber` | int | 激励数量（激励视频专用） |
| `isoCode` | String | 用户所在国家 ISO 码 |
| `requestId` | String | 请求 ID |
| `impressionId` | String | 曝光 ID |
| `isBiddingNetwork` | boolean | 是否竞价广告 |
| `waterfallIndex` | int | 瀑布流索引 |

---

## 7. 通用说明

- **初始化顺序**：使用任意 `J*Ad` 前需先调用 `JAdManager.init()` 或先 `prepareInit()` 再在隐私就绪后 `initWhenConsentReady()`。构造 `J*Ad` 时若未初始化会最多等待 5 秒，超时或未初始化则抛 `IllegalStateException`。
- **Context**：初始化与部分隐私 API 需使用 `Context`，建议使用 `Application#getApplicationContext()` 做长期持有。
- **配置**：通过 `JAdConfig.Builder` 构建 `JAdConfig`，包含 appId、debugMode、testMode、adapterType 等，见 `com.engagelab.ads.config.JAdConfig`。`adapterType` 使用 `JAdapterType.TYPE_DEFAULT`（int 常量），**不是字符串**。
- **错误码**：加载失败等通过各 `J*AdListener` 的 `onAdLoadFailed(JAdError error)` 回调，错误码定义见 `com.engagelab.ads.error.JAdError`。
- **回调参数**：所有回调方法均携带 `JAdInfo adInfo` 参数（除失败回调传 `JAdError`），`adInfo` 可能为 null，使用前需判空。

---

## 附录：错误码定义

| 常量 | 错误码 | 说明 |
|------|--------|------|
| `ERROR_CODE_NO_FILL` | 1001 | 无广告填充 |
| `ERROR_CODE_NETWORK` | 1002 | 网络错误 |
| `ERROR_CODE_TIMEOUT` | 1003 | 超时 |
| `ERROR_CODE_NOT_INIT` | 1004 | 未初始化 |
| `ERROR_CODE_INVALID_PARAMS` | 1005 | 参数无效 |
| `ERROR_CODE_AD_LOADING` | 1006 | 广告正在加载中 |
| `ERROR_CODE_AD_NOT_READY` | 1007 | 广告未准备好 |
| `ERROR_CODE_INTERNAL` | 1008 | 内部错误 |
| `ERROR_CODE_ADAPTER_NOT_FOUND` | 1009 | 适配器未找到 |
| `ERROR_CODE_UNKNOWN` | 9999 | 未知错误 |
