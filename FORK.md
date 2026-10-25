# Fork 说明 / About this fork

本仓库是 [XayahSuSuSu/Android-DataBackup](https://github.com/XayahSuSuSu/Android-DataBackup) 的**个人修改版（fork）**，
不是上游官方仓库，也未获得上游作者的背书或认可。

## 上游项目简介

| 项目 | 内容 |
| --- | --- |
| 名称 | DataBackup |
| 上游仓库 | <https://github.com/XayahSuSuSu/Android-DataBackup> |
| 作者 | XayahSuSuSu |
| 官方文档 | <https://DataBackupOfficial.github.io> |
| 上游发布渠道 | Telegram 频道 / 群组，见上游 README |
| 项目渊源 | 基于 YAWAsau 的 [speed-backup](https://github.com/YAWAsau/backup_script) 脚本，经原作者同意后发展为图形界面应用 |
| 核心能力 | 需要 Root（Magisk / KernelSU / APatch），支持多用户、云备份、100% 数据完整性 |
| 许可证 | GNU General Public License v3.0 |

## 本 fork 的改动

1. **补全「杀死应用选项」的实现说明**：`设置 -> 备份设置` 与 `设置 -> 恢复设置` 中的每个选项，在
   选择对话框内都附上了具体实现（调用了什么命令 / 什么接口），设置项下方也会实时显示当前选项的原理，
   不再只有「方案I / 方案II」这样的空名字。
2. **新增第 4 个选项「高级暂停」**：先强行停止应用，再用系统级 `pm suspend` 挂起——
   桌面、广播、服务、任务都无法再拉起它，图标变灰；该应用处理完成后立即解除挂起；
   系统应用只强行停止；任务异常中断时，下次启动会自动兜底解除所有遗留挂起。
3. **新增文档**：[docs/kill-app-options.md](docs/kill-app-options.md)（英文）、
   [docs/kill-app-options.zh-CN.md](docs/kill-app-options.zh-CN.md)（中文），逐项对照源码说明实现细节。

## 下载 / Downloads

构建产物发布在本仓库的 [Releases](https://github.com/FairyJJ/Android-DataBackup/releases) 页面，
命名与上游保持一致（`DataBackup-<版本>-<ABI>-<变体>-<构建类型>.apk`）：

| 资产 | 变体 | applicationId | 构建类型 | 体积 | 说明 |
| --- | --- | --- | --- | --- | --- |
| `DataBackup-2.0.12-arm64-v8a-foss-release.apk` | foss | `com.xayah.databackup.foss` | release | 13.9 MB | 无 Firebase 依赖，日常使用**推荐** |
| `DataBackup-2.0.12-arm64-v8a-premium-release.apk` | premium | `com.xayah.databackup.premium` | release | 14.5 MB | 含 Firebase（Analytics / Crashlytics） |
| `DataBackup-2.0.12-arm64-v8a-foss-debug.apk` | foss | `com.xayah.databackup.foss` | debug | 34.7 MB | 未混淆，用于抓日志排查 |

- release 构建开启 R8 代码混淆与资源压缩（`isMinifyEnabled` + `isShrinkResources`），`android:debuggable=false`；
  debug 构建关闭两者。
- `foss` 与 `premium` 的 `applicationId` 不同，可以**并存安装**。
- 签名：release 用 **fork 自有 keystore**，debug 用 **Android debug keystore**，两者签名不同，不能互相覆盖安装。
  与上游官方版本签名同样不同，从上游版本迁移过来需要先卸载（卸载会清空应用内的任务历史，存储目录里的备份文件不受影响）。
- 本仓库只提供 `arm64-v8a`（覆盖绝大多数 64 位 arm 设备）。其他 ABI 请自行按下面的步骤构建。

### 自行构建正式版

`release` 构建类型从环境变量读取签名信息（见 `build-logic/convention/.../ApplicationCommonConventionPlugin.kt`），
签名文件与口令不随仓库分发，请使用你自己的 keystore：

```bash
export STORE_FILE=/absolute/path/to/your.jks
export STORE_PASSWORD=你的store口令
export KEY_ALIAS=你的别名
export KEY_PASSWORD=你的key口令

cd source
./gradlew :app:assembleArm64-v8aFossRelease
./gradlew :app:assembleArm64-v8aPremiumRelease   # 含 Firebase，需 feature/flavor/premium/google-services.json
./gradlew :app:assembleArm64-v8aFossDebug        # 调试版，用内置 debug keystore
# 产物：source/app/build/outputs/apk/<变体>/<构建类型>/DataBackup-<版本>-arm64-v8a-<变体>-<构建类型>.apk
```

## 免责声明 / Disclaimer

- 本仓库为**第三方修改版**，与上游作者无关；上游作者不对本仓库的代码、构建产物或任何使用后果负责。
- 使用本软件**必须拥有 Root 权限**。刷机、备份、恢复均有风险，请自行承担由此产生的**一切后果**，
  包括但不限于应用数据丢失、备份损坏、系统无法启动、设备变砖。
- 「高级暂停」使用系统级 `pm suspend`，理论上不会持久影响应用：处理完成后自动解除，异常中断也会在下次任务开始时兜底解除。
  但仍建议**不要对系统核心应用**（SystemUI、桌面、输入法、电话等）使用该选项；如遇应用长期挂起（图标变灰、无法打开），
  可在 root shell 或 adb 中执行 `pm unsuspend --user 0 <包名>` 手动解除。
- 请务必**在已有可用备份的前提下**验证恢复功能，不要用唯一一份数据做实验。
- 本项目按「现状（AS IS）」提供，不附带任何明示或暗示的担保，包括但不限于适销性与特定用途适用性。
- 本仓库依据 **GNU GPL v3.0** 发布，完整源码随本仓库提供；上游版权归 XayahSuSuSu 及各位贡献者所有。
- 若您是上游作者并认为本仓库存在不妥之处，请通过 Issue 联系，我会立即调整或下线相关内容。
