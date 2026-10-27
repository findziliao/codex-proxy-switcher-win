# Codex Proxy Switcher 项目介绍

Codex Proxy Switcher 是一个 Windows 小启动器，用来在启动 Codex 桌面端前选择网络模式：

- 原生启动：不注入代理环境变量，按 Codex 默认网络环境启动。
- VPN 启动：激活 Codex 前临时设置 `HTTP_PROXY`、`HTTPS_PROXY`、`ALL_PROXY` 和 `NO_PROXY`，启动后立即恢复。

project: [hloolx/codex-proxy-switcher-win](https://github.com/hloolx/codex-proxy-switcher-win)

它不会修改 Windows 全局代理，也不会写入系统级环境变量。为兼容 Windows 商店应用激活，程序会在启动窗口内短暂调整当前用户环境变量，激活 Codex 后恢复原值。

## 适配其他 Windows 电脑

这个启动器的便携逻辑是：

1. 第一次打开时要求输入本地代理端口，例如 `7897`、`7890` 或 `9000`；代理需要账号密码时，还可以填完整代理地址和用户名密码。
2. 程序会把端口保存到当前用户目录：
   `%APPDATA%\CodexProxySwitcher\settings.json`
3. 之后每次打开都会自动读取这个端口。代理密码会用 Windows DPAPI 加密后保存，只绑当前 Windows 用户。
4. 启动时自动查询这台电脑上的 Windows 商店版 Codex 安装位置和 AppUserModelID。
5. 优先通过系统应用激活方式启动 Codex，避免直接运行 WindowsApps 里的 Electron exe。
6. 首次配置完成后可选择创建桌面快捷方式，只提示一次。

因此，同一个 exe 拷到另一台 Windows 电脑后，第一次运行会让那台电脑的用户输入自己的代理端口，后续就不用重复输入。

## 自动查找 Codex 的方式

启动器会按顺序尝试：

1. 通过 PowerShell 的 `Get-AppxPackage -Name OpenAI.Codex` 查询安装目录和 Package Family Name。
2. 读取 `AppxManifest.xml` 获取 Application Id，生成类似 `OpenAI.Codex_2p2nqsd0c76g0!App` 的 AppUserModelID。
3. 优先通过 AppUserModelID 激活 Codex，避免绕过 Windows 商店应用启动链路。
4. VPN 模式会在激活前短暂设置当前用户级代理环境变量，激活后立即恢复原值。
5. 如果 Appx 查询失败，再在 `C:\Program Files\WindowsApps` 中查找 `OpenAI.Codex_*_x64__2p2nqsd0c76g0`。

如果 Codex 官方后续更改包名或目录结构，可能需要更新匹配规则。

## 解决的问题

这个项目用于缓解 Windows 桌面版 Codex 因网络没有正确走代理而出现的连接问题，例如：

- 启动后长时间显示 `Reconnecting`。
- 聊天或执行任务时反复断线。
- 代理软件可用，但 Codex 桌面端没有自动使用它。
- 新安装电脑启动 WindowsApps 里的 `Codex.exe` 时提示“拒绝访问”。
- 用户不想修改 Windows 全局代理，只想让 Codex 单独走代理。

它通过“临时环境变量 + 系统应用激活”实现，启动后会恢复原来的环境变量。

这样可以兼容更严格的新安装环境，也能避开直接启动 WindowsApps 中 `Codex.exe` 可能触发的 Electron 路径解析问题。

## API Key 登录说明

本项目不管理 API key，也不改变 Codex 登录方式。它只负责启动时的网络出口选择。

如果 API key 登录/使用失败是网络连接问题，VPN 启动模式可能有帮助；如果是 key 无效、额度不足、组织权限、模型权限或外部浏览器登录页打不开，则需要分别处理认证或浏览器网络问题。

## 为什么这样做

采用“临时环境变量”而不是修改 Windows 全局代理，主要是为了降低影响范围：

1. 只在 Codex 激活窗口内调整代理变量，并在启动后恢复。
2. 可以随时通过“原生启动”回到默认网络。
3. 不需要长期管理员权限。
4. 不影响浏览器、Git、包管理器或其他应用。
5. 端口配置保存在用户目录，方便在不同 Windows 电脑上首次配置后长期使用。

## 最佳使用方式

1. 先启动代理软件。
2. 确认本地 HTTP 代理端口。
3. 第一次打开启动器时输入端口号；代理需要账号密码时一起填好。
4. 日常使用优先点击“VPN 启动”；要改配置先点主界面左下角的 `Settings`（启动成功后本工具窗口会自动关闭）。
5. 网络不需要代理时点击“原生启动”。
6. 想让 VS Code 里的 Codex 扩展也走代理，点主界面的 `VS Code 代理`：可以写 `~/.codex/.env`（扩展、CLI、桌面版通吃），也可以写 VS Code 的 `http.proxy`（只影响 VS Code 侧），两个开关互相独立。
7. 首次配置后可以选择添加桌面快捷方式，任务栏也建议固定本启动器，而不是原始 Codex 图标。

## 项目文件

- `src\Program.cs`：启动器主程序源码。
- `src\CodexProxySwitcher.csproj`：.NET Windows Forms 项目文件。
- `release\Codex-Proxy-Switcher.exe`：可直接下载运行的自包含 exe。
- `Start-Codex-VPN.bat`：一键代理启动脚本。
- `Start-Codex-Native.bat`：一键原生启动脚本。
- `Start-Codex.ps1`：bat 背后的 PowerShell 启动逻辑。
- `proxy-url.txt`：旧版脚本使用的代理地址配置。

## 推荐分发方式

普通用户只需要下载仓库里的 `release\Codex-Proxy-Switcher.exe` 这一个文件即可使用。首次运行时让对方输入自己的代理端口，之后程序会自动保存配置并自动查找 Codex 安装位置。

当前推荐发布的是 self-contained 版本，对方通常不需要额外安装 .NET Runtime，但 exe 文件会明显更大。
