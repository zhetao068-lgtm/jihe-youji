# 离线数学应用如何限制网页层的访问权限

**摘要**　讨论动态绘图需要脚本、而练习本身可以离线的场景。以几何有迹体验版为例，说明哪些访问由应用约束，哪些验证需要逐平台完成。

## 网页层的边界

浏览器渲染负责几何图、自测和草稿。公开体验版只有一个随应用分发的页面，其资源数量已知。Windows 外壳为页面和脚本提供两个固定地址，通过宿主检查请求方法与地址，匹配时从嵌入资源读取，其余请求返回错误。应用同时拒绝新窗口、下载、外部协议和网页权限申请。`connect-src 'none'` 为网页连接增加一层限制。[Microsoft WebView2 安全指南](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/security)

Android 外壳将压缩的页面解开后交给 WebView，页面中的脚本需匹配内容安全策略中声明的摘要。APK 清单没有声明应用级网络权限。WebView 设置阻止网络加载、文件和内容访问，禁用混合内容及 Cookie。草稿仍可通过应用内网页存储使用。[Android WebView 本地内容](https://developer.android.com/develop/ui/views/layout/webapps/load-local-content) · [Android WebSettings](https://developer.android.com/reference/android/webkit/WebSettings)

## 数据生命周期

练习页面保存收藏、最近使用、草稿和进度。这些记录使用一个专用键，Windows 体验版的数据目录和 Android 体验版的应用标识也都独立。清除记录时，界面删除对应键、取消待保存计时并在重置界面期间暂停写入。下一次保存使用重置后的状态，避免恢复刚清除的内容。

## 验证方法

- 检查 Windows ZIP 的文件清单和 APK 权限列表。检查公开资源是否误包含源映射、个人路径、签名材料或完整题库。
- 用公开发布页提供的 SHA-256 核对下载。使用 Android 签名工具核对 APK 签名方案及签名者指纹。文件摘要用于检查一致性，可信度还取决于获取哈希和文件的渠道。[Android apksigner](https://developer.android.com/tools/apksigner)
- 验证清除记录时先创建一项收藏，再清理、重载，最后确认收藏没有恢复。分别记录宿主检查、桌面浏览器和真机测试的结果。

**适用范围**　这是应用层隔离设计。系统网络策略、浏览器运行时更新和设备的整体安全由用户所在环境管理。Android 原生安装和触控兼容性仍需真机测试。
