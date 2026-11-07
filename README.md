# SlimeGlue

为[粘液科技](https://github.com/Slimefun/Slimefun4)接入王国、锁箱、商店等插件的保护系统，修复机器人破坏他人领地与上锁方块，自动识别已装插件。

SlimeGlue is a [Slimefun](https://github.com/Slimefun/Slimefun4) addon which improves compatibility between Slimefun and other plugins.

粘液胶是一个为提升粘液科技与其它插件兼容性的附属插件。

## 已修复的兼容问题 Fixed compatibilities

| 插件 | 修复内容 |
| --- | --- |
| [KingdomsX](https://github.com/CryptoMorin/KingdomsX/) | 修复 Slimefun 机器人破坏未授权领地内方块、领地结构/炮塔方块 [#3](https://github.com/Xzavier0722/SlimeGlue/issues/3) |
| [Magic](https://github.com/elBukkit/MagicPlugin/) | 修复机器人破坏法术生成的假方块导致刷物品 [MagicPlugin#1172](https://github.com/elBukkit/MagicPlugin/issues/1172) |
| [QuickShop-Reremake](https://github.com/PotatoCraft-Studio/QuickShop-Reremake) | 修复机器人破坏 QuickShop 箱子商店 |
| [QuickShop-Hikari](https://github.com/QuickShop-Community/QuickShop-Hikari) | 修复机器人破坏 QuickShop-Hikari 箱子商店 |
| [LockettePro](https://github.com/Soldier233/LockettePro) | 修复机器人破坏被上锁保护的方块 [#25](https://github.com/Xzavier0722/SlimeGlue/issues/25) |

## 前置要求 Requirements

- Java 21 及以上
- Spigot / Paper 及其衍生服务端（`api-version: 1.13`）
- [Slimefun4](https://github.com/Slimefun/Slimefun4)（硬依赖，必须安装）

## 安装 Installation

1. 从 [Releases](https://github.com/balugaq/SlimeGlue/releases) 下载最新的 jar；
2. 放入服务端 `plugins/` 目录，重启服务器。

无需任何配置或命令。模块按服务器已安装的插件自动启用与卸载（监听 `PluginEnableEvent` / `PluginDisableEvent`），装了对应插件即生效。

## 工作原理 How it works

SlimeGlue 向 Slimefun 的保护管理器（`ProtectionManager`）注册一个保护模块，把 Slimefun 机器人（矿物收集机、挖矿机等自动化设备）的方块破坏/放置/交互行为，转交给各保护插件（领地、锁箱、商店）判定权限，未授权时拦截。

## 反馈 Feedback

如果您发现任何不兼容的插件或功能，请在本仓库[提交 issue](https://github.com/balugaq/SlimeGlue/issues)。您可能需要协助验证您提交的 issue，非常感谢您的付出！

If you found any plugin or any function cannot work well with Slimefun, please post an issue under this repository. You may be requested to verify the issue you posted. Thanks for your efforts!

## 构建 Building

```bash
gradle build
```

产物位于 `build/libs/`。

## 许可 License

见 [LICENSE](./LICENSE)。
