# AGENTS.md — SlimeGlue 操作宪章

> 本文件是 AI 编程助手与本仓库开发者的**操作契约**。所有改动（代码、构建、文档）均须遵守本文约定；
> 当本文与代码冲突时，按末尾《文档更新契约》处理。

## TL;DR（太长了不看）

1. 本仓库是 **Minecraft Spigot/Paper + Slimefun4 附属插件**（Java 21 / Gradle Groovy DSL），核心功能是插件间兼容层：把 Slimefun 机器人的方块操作转交其他保护插件判定。所有改动必须通过 `gradle clean build` 验收后才能提交，禁止提交未编译验证的代码。
2. 本仓库**没有**玩家可见文案、命令与配置文件——不要发明 config.yml / 命令注册，也不要引入语言文件。日志一律走 `GlueLogger`，**禁止** `System.out.println`。
3. 本仓库是 `Xzavier0722/SlimeGlue` 的 fork，开发仓库为 `balugaq/SlimeGlue`，直接在 `master` 分支开发；提交、推送由开发者本人执行。

---

## 1. 项目身份与边界

| 维度 | 事实 |
| --- | --- |
| 定位 | Slimefun4 附属，纯兼容层：不新增物品/机器/研究/命令，只把 Slimefun 的保护判定接入其他插件 |
| 入口类 | `com.xzavier0722.mc.plugin.slimeglue.SlimeGlue`（`extends JavaPlugin implements SlimefunAddon`） |
| 版本目标 | Java 21（字节码 major=65）、spigot-api 1.19、Slimefun4 RC-37、`api-version: 1.13` |
| 仓库策略 | fork 仓库（origin = `balugaq/SlimeGlue`，upstream = `Xzavier0722/SlimeGlue`），`master` 分支开发；AI 不执行提交与推送 |
| AI 职责范围 | 编写/修改功能代码、Bug 修复、构建/CI 维护、文档维护 |
| AI 不负责 | 发布版本、提交、推送 |

### 1.1 代码包地图（改代码前先定位）

| 包 | 职责 |
| --- | --- |
| `...slimeglue` | `SlimeGlue` 主类（启动流程、模块注册、保护模块注册与重试）；`GlueLogger` 日志封装（i/w/e/v 分级，v 为 verbose） |
| `...slimeglue.api` | 插件扩展 API：`ACompatibilityModule`（兼容模块基类，持有 listeners 与 protectionHandlers 两个集合）；`api.protection` 的 `IBlockProtectionHandler` / `IPlayerProtectionHandler` / `IProtectionHandler`；`api.listener` 的 `IListener` / `ISlimefunBlockListener` |
| `...slimeglue.manager` | `CompatibilityModuleManager`：模块注册表，`load(name)` / `unload(name)` 按 Bukkit 插件名启用/卸载对应模块（模块是否可加载由其自身判断目标插件是否存在） |
| `...slimeglue.module` | 五个兼容模块实现：`KingdomsXModule`、`MagicModule`、`QuickShopModule`（Reremake）、`QuickShopHikariModule`、`LocketteProModule` |
| `...slimeglue.listener` | `PluginListener`（监听插件启停，动态 load/unload 模块）、`BlockListener`（Slimefun 方块的破坏/放置事件分发，LOWEST 优先级可取消）、`SlimefunListener`、`SlimefunCompListener`（仅兼容版 Slimefun fork 下启用，监听 `CargoOperationEvent` 做货运权限判定） |
| `...slimeglue.slimefun` | `GlueProtectionModule`：实现 Slimefun4 dough 的 `ProtectionModule`，把 `Interaction`（破坏/放置/攻击实体等）翻译成对各模块 protection handler 的遍历问询 |

---

## 2. 技术栈与项目结构（事实核对）

- **构建**：Gradle Groovy DSL（`build.gradle`）；版本号 `findProperty('version') ?: "1.0.3-<gitCommit>"`——`gradle.properties` 里的 `version=1.0.3` 优先（release CI 按它打 tag），本地开发回落到 commit 后缀；产物 `build/libs/SlimeGlue-exp-<commit>.jar`。无 shade/relocation，全部依赖 `compileOnly`。
- **依赖仓库**：spigot-api、JitPack（Slimefun4、Kingdoms、SlimefunCompLib）、CodeMC（QuickShop）、elmakers（MagicAPI）、ci.nyaacat.com（LockettePro）。改依赖前先确认对应仓库可解析。
- **启动顺序**（`SlimeGlue.onEnable()`，新增初始化逻辑须插入对应阶段，不要打乱顺序）：
  初始化 logger → 创建 `CompatibilityModuleManager` → `registerModules()` 注册五个模块 → `moduleManager.load()` 加载当前已装插件对应的模块 → 注册 `PluginListener` / `BlockListener` / `SlimefunListener` → 若检测到兼容版 Slimefun（`ICompatibleSlimefun`，来自 `SlimefunCompLib`）再注册 `SlimefunCompListener` → 向 `Slimefun.getProtectionManager()` 注册 `GlueProtectionModule`；失败时起 20 tick 间隔的重试任务（最多 10 次）。
- **无配置文件、无命令、无语言文件**：`plugin.yml` 无 `commands`/`permissions` 段，`src/main/resources` 只有 `plugin.yml`。
- **CI**（`.github/workflows/`）：
  - `build.yml`：JDK 21，push/PR 到 `master`，产物上传名为 `SlimeGlue`；提交信息以 `[ci skip]` 开头可跳过。
  - `release.yml`：从 `gradle.properties` 读 `version=`，tag 不存在时构建并发布 `v<version>`。
  - `test-compatibility.yml`：矩阵 `1.20.1,26.2`，启动 Paper 验证插件能 Enabling；Slimefun4 从鬼斩资源站下载（仓库 `master-271` 构建）。

---

## 3. 决策树：当遇到 A 时，执行 B

### 3.1 新增兼容插件支持

- **当新增一个兼容模块时**：在 `module/` 下新建 `XxxModule extends ACompatibilityModule`，在构造器里 `addProtectionHandler(...)` 注册 `IBlockProtectionHandler` / `IPlayerProtectionHandler` 实现；在 `SlimeGlue.registerModules()` 注册；`build.gradle` 加 `compileOnly` 依赖；`plugin.yml` 的 `softdepend` 加插件名；README「已修复的兼容问题」表加一行。
- **当保护判定需要区分"插件自己的方块"与"玩家操作"时**：实现 handler 的 `bypassCheck`（返回 true 的位置跳过判定，见 `GlueProtectionModule` 的遍历逻辑）。
- **当目标插件提供自己的事件（如货运访问）时**：参照 `SlimefunCompListener` 的模式新增监听器，LOWEST 优先级 + `ignoreCancelled = true`，遍历 protection handlers 命中不可访问即 cancel。
- **当模块需要感知 Slimefun 方块的破坏/放置时**：实现 `ISlimefunBlockListener`（返回 false 取消事件），由 `BlockListener` 按 Slimefun ID 分发。

### 3.2 日志

- **当需要输出日志时**：用 `SlimeGlue.logger().i/w/e(...)`；调试信息用 `v(...)`（verbose），保持现有模块里 `verbose(...)` 的用法一致。**禁止** `System.out.println` 与 `e.printStackTrace()`。

### 3.3 依赖与构建

- **当需要新增依赖时**：一律 `compileOnly`（本插件不打 fat jar）；新仓库加进 `repositories` 前先验证可解析性；改完跑 `gradle clean build`。
- **当改版本号发版时**：只改 `gradle.properties` 的 `version=`，release CI 会自动打 tag 发布；不要直接改 `build.gradle` 的回落值以外的逻辑。
- **当构建失败时**：先读报错定位（依赖解析 / 编译错误），修复后重跑；禁止 `-x test`、`--offline` 等绕过方式掩盖失败。

### 3.4 Java 版本

- 源码保持 Java 21（`sourceCompatibility`/`targetCompatibility` = 21）。降级前先确认依赖 class file 版本不高于目标；升级大版本用 `options.release` 写法一步到位，并验证字节码 major（61=17，65=21）。

---

## 4. 红线（绝对禁止操作）

1. **禁止 git push -f 或改写历史**：禁止 force push、rebase 改写、`git reset --hard` 后强推。
2. **禁止未构建验证就提交**：任何代码改动提交前必须通过 `gradle clean build`。
3. **禁止在生产路径使用 `System.out.println()`**：日志只能走 `GlueLogger`。
4. **禁止发明配置/命令/文案**：本插件是纯兼容层，不新增 config.yml、命令或玩家可见消息；拦截行为不应有 UI 反馈。
5. **禁止改动 `GlueProtectionModule` 的判定顺序语义**：handler 遍历中任一 handler 拒绝即拒绝（返回 false），这是全插件的安全语义核心；调整必须逐一验证五个模块的回归。
6. **禁止把上游 fork 关系弄丢**：`upstream` remote 指向 `Xzavier0722/SlimeGlue`，同步上游由开发者执行。

---

## 5. 质量规范

- **代码风格**：跟随现有代码风格（4 空格缩进、模块内匿名内部类实现 handler 的既有模式）。未配置 linter，不要擅自引入新格式化工具。
- **可编译性**：每个完成的改动都必须是"能编译"的完整状态；不留半成品/死代码/未使用 import。
- **兼容目标**：`api-version: 1.13`，不要在代码里使用高于 spigot-api 1.19 的 API 而不做版本判空。

---

## 6. 常用 Gradle 命令速查表

| 场景 | 命令 |
| --- | --- |
| 标准验收构建（必须通过） | `gradle clean build` |
| 增量编译（快速检查） | `gradle compileJava` |
| 查看依赖树 | `gradle dependencies` |
| 清理产物 | `gradle clean` |
| 产物位置 | `build/libs/SlimeGlue-exp-<commit>.jar` |

---

## 7. Git 工作流

- **分支**：本地 `master` 直接开发，不建长命分支；提交与推送由开发者本人执行，AI 把工作区留到可通过验收的状态并给出建议提交信息。
- **提交信息**：Conventional Commits（`feat:` / `fix:` / `docs:` / `refactor:` / `chore:`）。
- **gradlew 执行位**：Windows 提交时注意保持 `100755`（`git ls-files -s gradlew` 核对）；三个 workflow 已内置 `chmod +x gradlew` 兜底。

---

## 8. 文档更新契约

1. **冲突即提示**：当发现本文与代码不一致时，必须在回复中主动指出冲突，并说明应以代码为准还是更新本文。
2. **惯例沉淀**：产生新的可复用约定时，先提议、经开发者确认后补进本文。
3. **保持精简**：每一条规则都应能被"是否遵守"直接检查。
