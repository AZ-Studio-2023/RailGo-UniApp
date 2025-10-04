# laoqianjunzi-widget 桌面小组件模块

本模块用于在 Android 系统桌面添加自定义小组件，支持 LOGO 图标组件和好友电量测试组件。适用于基于 uni-app/UTS 的项目开发，兼容 Android 7.1（API 25）及以上系统。

## 功能简介

- 支持一键添加 LOGO 图标小组件到桌面
- 支持一键添加测试小组件到桌面
- 自动检测系统版本及小组件是否已添加
- 兼容主流 Android 手机品牌
- 支持小组件周期性刷新、网络数据展示
- 提供详细的 API 和使用示例

## 安装方法

将本模块放置于 `uni_modules` 目录下，确保路径为：

```
uni_modules/laoqianjunzi-widget/
```

## 主要 API

### 1. 添加 LOGO 图标小组件

```typescript
import { addAppWidget } from "@/uni_modules/laoqianjunzi-widget"

addAppWidget()
```

### 2. 添加测试小组件

```typescript
import { addTestWidget } from "@/uni_modules/laoqianjunzi-widget"

addTestWidget()
```

## 组件说明

### AppWidget

- 负责 LOGO 图标小组件的生命周期管理
- 支持周期性内容刷新
- 支持自定义布局与样式

### TestWidget

- 负责测试小组件的生命周期管理
- 支持网络请求动态刷新内容
- 支持自定义布局与样式

## 权限说明

请确保在 `AndroidManifest.xml` 中声明以下权限：

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="com.android.launcher.permission.INSTALL_SHORTCUT" />
<uses-permission android:name="android.permission.REQUEST_INSTALL_SHORTCUT" />
<uses-permission android:name="android.permission.REQUEST_PIN_APPWIDGET" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.BIND_APPWIDGET" />
```

## 常见问题

- **Q：为什么无法添加小组件？**  
  A：请确认系统版本为 Android 7.1 及以上，并已授予相关权限。

- **Q：小组件内容无法刷新？**  
  A：请检查网络请求接口是否可用，或查看日志输出。

- **Q：如何自定义小组件布局？**  
  A：请修改 `res/layout` 目录下的布局 XML 文件，并在代码中引用。

## 目录结构

```
uni_modules/
  laoqianjunzi-widget/
    utssdk/
      app-android/
        AppWidget.uts
        TestWidget.uts
        index.uts
        AndroidManifest.xml
        ...
    readme.md
```

## 更新日志

- v1.0.0 初始版本，支持 LOGO 图标和测试组件添加

## 联系与反馈

如有问题或建议，请通过 GitHub Issues 或邮箱联系作者。
