# 影盒 (Yinghe)

一款开源的 Android 影音播放应用，支持点播、直播、多线路播放。本项目同时提供 **TV 版** 和 **手机版** 两个独立应用，基于 [takagen99/Box](https://github.com/takagen99/Box) 二次开发。

## 功能特性

- 📺 **点播 + 直播 + 回看** 一体化
- 🔍 全站搜索、快速搜索、搜索历史
- 🕘 观看历史（可设为首页默认显示）
- ⭐ 收藏、推送播放
- 🎬 多播放器：系统 / IJK / EXO，支持硬解软解切换
- 📡 多线路、多解析、自定义 UA / Referer / Cookie
- 🔄 JSON 接口配置数据源，支持自定义站点、直播源、解析规则
- 🐍 Python 爬虫支持（Chaquopy 引擎，需选择 `python` flavor）
- 🏠 投屏 / 推送播放

## 两个版本

| | TV 版 (影盒TV) | 手机版 (影盒) |
|---|---|---|
| **包名** | `yinghe.tv.jink` | `yinghe.mobile.jink` |
| **应用名** | 影盒TV | 影盒 |
| **当前版本** | 1.2.0 | 1.3.0 |
| **targetSdk** | 33 | 30 |
| **minSdk** | 19 (python: 24) | 21 |
| **播放器** | `player` (带弹幕) | `player_mobile` |
| **QuickJS** | `quickjs` (精简版) | `quickjs_mobile` (完整版) |
| **UI** | TV 遥控方向键导航 | 触屏 + 底部导航栏 |

两个版本使用 **同一签名**（YingheOSC），可共存于同一设备。

## 项目结构

```
Yinghe/
├── app/                  # TV 版主模块
├── app_mobile/           # 手机版主模块
├── player/               # TV 版播放器（IJK + EXO + 弹幕）
├── player_mobile/        # 手机版播放器（IJK + EXO）
├── quickjs/              # TV 版 QuickJS 引擎
├── quickjs_mobile/       # 手机版 QuickJS 引擎（完整版）
├── pyramid/              # Python 爬虫引擎（Chaquopy）
├── TabLayout/            # Kotlin DSL TabLayout（手机版依赖）
├── ViewPager1Delegate/   # ViewPager1 代理
├── crash/                # 崩溃捕获（CustomActivityOnCrash）
├── common_res/           # TV 和 Mobile 共用的 drawable/raw 资源
├── common_assets/        # TV 和 Mobile 共用的 assets（js/lib、ua.db 等）
├── .github/workflows/
│   ├── Yinghe.jks        # 签名文件（不入库，自行备份）
│   └── ...
└── outputs/              # 打包输出目录（不入库）
```

## 构建

### 环境要求

| 组件 | 版本 |
|---|---|
| **OS** | Linux / macOS / Windows |
| **JDK** | 21（Debian 13 默认） |
| **Gradle** | 8.5（Wrapper 自动下载） |
| **AGP** | 8.3.2 |
| **Android SDK** | platform-33 + build-tools 34.0.0 + NDK r25c |
| **Python** | ≥ 3.5（仅 python flavor 构建时需要，运行时不依赖） |

首次构建请在项目根目录执行一次 `export ANDROID_HOME=<你的SDK路径>`。

### 构建变体

**TV 版**（app 模块）：

| Variant | ABI | minSdk | 说明 |
|---|---|---|---|
| `java` | armeabi-v7a + arm64-v8a | 19 | 通用包，纯 Java 爬虫 |
| `java32` | armeabi-v7a | 19 | 仅 32 位 |
| `java64` | arm64-v8a | 21 | 仅 64 位 |
| `python` | armeabi-v7a + arm64-v8a | 24 | 带 Python 爬虫引擎 |
| `python32` | armeabi-v7a | 24 | Python 仅 32 位 |
| `python64` | arm64-v8a | 24 | Python 仅 64 位 |

**手机版**（app_mobile 模块）：仅有一个 default variant。

### 常用构建命令

```bash
# TV 版（通用包，双架构，推荐）
./gradlew :app:assembleJavaDebug
./gradlew :app:assembleJavaRelease

# 手机版
./gradlew :app_mobile:assembleDebug
./gradlew :app_mobile:assembleRelease

# 两个版本一起打 release 签名包
./gradlew :app:assembleJavaRelease :app_mobile:assembleRelease copyApks
```

### 一键打包并输出到 outputs/

```bash
./gradlew :app:assembleJavaRelease :app_mobile:assembleRelease copyApks
```

APK 自动复制到项目根目录的 `outputs/` 文件夹：

| 文件 | 说明 |
|---|---|
| `Yinghe_TV_latest.apk` | TV 版最新版 |
| `Yinghe_TV_Release_YYYYMMDD_HHMMSS.apk` | TV 版时间戳快照（自动清理，保留最近 2 个） |
| `Yinghe_Mobile_latest.apk` | 手机版最新版 |
| `Yinghe_Mobile_Release_YYYYMMDD_HHMMSS.apk` | 手机版时间戳快照（自动清理，保留最近 2 个） |

## 数据源配置

应用内 "设置 → 配置" 可填入远程 JSON 接口地址。

```json
{
  "spider": "./your.jar",
  "wallpaper": "./api/img",
  "sites": [],
  "parses": [],
  "hosts": [
    "cache.ott.ystenlive.itv.cmvideo.cn=base-v4-free-mghy.e.cdn.chinamobile.com"
  ],
  "lives": [],
  "rules": [],
  "doh": [
    { "name": "腾讯", "url": "https://doh.pub/dns-query" },
    { "name": "阿里", "url": "https://dns.alidns.com/dns-query" }
  ]
}
```

### 字段说明

| 字段 | 说明 |
|---|---|
| `spider` | 爬虫 jar 包路径 |
| `wallpaper` | 壁纸接口 |
| `sites` | 站点配置（`searchable`: 搜索开关 0/1；`filterable`: 首页可选 0/1；`playerType`: 播放器 系统/IJK/EXO） |
| `parses` | 解析接口（0 嗅探，1 返回直链） |
| `lives` | 直播配置（`ua`/`epg`/`logo`） |
| `rules` | 爬虫替换规则 |
| `doh` | DNS over HTTPS 服务器列表 |

## GitHub Release

最新版本请前往 [Releases](https://github.com/Jink92/Yinghe/releases) 页面下载。

## 免责声明

本项目仅供技术学习与交流使用，请勿用于任何非法用途。所有数据源均来自互联网，版权归原作者所有，使用者请遵守当地法律法规。

## 许可

基于开源协议，代码仅用于学习研究，请遵守相应开源协议及原作者约定。
