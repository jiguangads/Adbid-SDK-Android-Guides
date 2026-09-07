# JAdbid Android SDK 集成总览

当前发版形态为普通包、公有云，只输出 `jadbid-android-{version}-release.aar`。宿主负责声明所需渠道依赖、配置 Manifest，并安全注入 App ID 与广告位 ID。

| 项目 | 当前口径 |
| --- | --- |
| 主 AAR | `jadbid-android-{version}-release.aar` |
| 渠道依赖 | 宿主按需声明，不内嵌到普通包 |
| 最低系统 | Android 6.0 / API 23 |
| Java | 8+ |
| 默认聚合平台 | TradPlus |
| 公开广告类型 | 横幅、插屏、激励视频、原生、开屏 |
| 参考工程 | Demo 压缩包内的 `jadbid-android-example/standard` |

## 推荐阅读

1. 按[JAdbid Android SDK 集成指南](./JAdbid%20Android%20SDK%20集成指南.md)完成依赖、初始化与外部配置。
2. 按[公开 API 文档](./API-接口说明.md)接入具体广告类型。
3. 解压仓库 `demo/jadbid-android-example-1.0.1-source.zip`，使用其中的 `standard` 模块进行编译与真机验收。

## 关键边界

- 开屏广告只在宿主显式调用 `showAd` 时展示；加载成功不等于已展示。
- 开屏场景由宿主设置超时和放弃策略，SDK 不替宿主控制首屏跳转。
- `JSplashAd` 的公开回调由 JAdbid 派发到主线程；新请求会使旧请求回调失效，`destroy()` 幂等。既有四类的回调线程与销毁竞态语义取决于当前适配器实现。
- JAdbid 不新增自有数据采集、埋点、缓存或网络上报，隐私同意沿用现有初始化流程。
