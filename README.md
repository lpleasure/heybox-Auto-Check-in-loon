# 小黑盒每日签到 · Loon 插件

在 iOS 上用 Loon 每天自动完成小黑盒签到。开启 MitM 后打开一次小黑盒 App，插件会自动录入账号，之后每天 8:00 定时签到并推送结果。

## 安装

1. Loon → 配置 → 插件 → 添加，填入：

   ```
   https://raw.githubusercontent.com/lpleasure/heybox-Auto-Check-in-loon/main/heybox.plugin
   ```

2. 确认 MitM 已开启，并且 CA 证书已安装、已信任（设置 → 通用 → 关于本机 → 证书信任设置）。
3. 打开小黑盒 App 随便刷几下。收到「已录入账号 xxx」的通知就说明录好了。
4. 录入后可以关掉 MitM 里的 `api.xiaoheihe.cn`，签到不需要它；Cookie 失效后再打开重新录一次即可。

想立即测试：Loon → 脚本 → 找到「小黑盒签到」→ 运行。

## 工作方式

| 脚本 | 触发 | 作用 |
|---|---|---|
| 小黑盒抓包 | 请求 `api/data.xiaoheihe.cn` | 从请求里取 `heybox_id` 和 Cookie 中的 `pkey` / `x_xhh_tokenid`，存进 Loon 持久化存储。仅首次录入时通知，之后 Cookie 刷新静默更新 |
| 小黑盒签到 | cron `0 8 * * *` | 查询签到状态 → 未签到则签到 → 再次确认 → 读取 H 币，推送通知 |

支持多账号：用不同账号各登录抓一次即可，存储格式为 `heybox_id#pkey=..;x_xhh_tokenid=..`，以 `&` 分隔。

### 存储键

| 键 | 内容 |
|---|---|
| `HEYBOX_ACCOUNTS` | 账号与 Cookie |
| `HEYBOX_STATE` | 最近一次运行结果 (JSON) |
| `HEYBOX_HKEY_API` | 可选，自定义 hkey 服务地址 |

## 修改签到时间

把插件 fork 一份，或在 Loon 里把插件转为本地后，修改 `cron "0 8 * * *"` 一行（分 时 日 月 周）。

## 注意事项

- **签名依赖第三方服务。** 小黑盒接口要求 `hkey` 签名，算法未公开，本插件调用第三方服务 `https://hkey.qcciii.com/hkey` 获取。该服务会收到 `heybox_id`、`imei`、`os_version`、请求路径和时间戳，**不会**收到 Cookie。服务返回的 `hkey` 和 `_rnd` 会随请求一起发给小黑盒。服务不可用时签到会失败。
- **请求以 Android 客户端身份发出。** hkey 服务按 Android App 签名，所以签到请求固定使用 Android 设备参数，而不是 iOS App 里抓到的参数。
- **Cron 依赖 Loon 在运行。** 定时任务只在 Loon 的 VPN 连接着时执行。
- **未在真机验证。** 脚本已用模拟的 Loon API 跑通完整流程，但真实接口返回值（特别是已签到状态的取值）以实际运行为准。
- **风险自负。** 自动化签到可能违反小黑盒用户协议，存在账号被风控的可能。Cookie 等同于登录凭证，不要分享。

## 致谢

改编自 [MinamiHashiRun0/heybox-checkin](https://github.com/MinamiHashiRun0/heybox-checkin)（Surge 模块，MIT License）。Loon 版改动：`$httpClient` 超时改为毫秒单位，去掉 Surge 专用的信息面板，改为 Loon 插件格式。
