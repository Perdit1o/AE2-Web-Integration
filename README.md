# AE2 Web Integration 简体中文包（GTNH 2.9.0）

> Chinese localization for **AE2 Web Integration 2.0.0** on Minecraft 1.7.10 / Forge, including a small
> server-side mod that makes a **dedicated server** resolve item names in Chinese.
>
> 适用于 Minecraft 1.7.10 + Forge + GTNH 2.9.0 的网页终端汉化包：网页界面中文 + 服务端物品名中文。
>
> 本分支（`AE2-Web-Integration-汉化版本`）存放 AE2 Web Integration 的简体中文产物与说明。
> 上游项目本身的说明文档保留在 `README.upstream.md`。

---

## 这个仓库里有什么

| 文件 | 放哪里 | 作用 |
|---|---|---|
| `ae2webintegration-2.0.0-forge-1.7.10-zh_CN.jar` | `mods/`（**替换**原版同名 mod） | 网页界面简体中文化。原版只有英文 / 波兰语，本包补充了完整的中文界面文案 |
| `ae2wi-namelang-1.3.1.jar` | `mods/`（**新增**） | 让**专用服务器**把物品 / 方块名解析成中文，专治"单机看是中文、服务端网页看是英文" |

两个文件解决的是两件事，需要一起装才完整：

- 只装第一个 → 界面是中文了，但物品名还是 `Rod of the Highlands`、`Aluminium Ingot`；
- 只装第二个 → 网页界面还是英文，物品名却是中文。

### 为什么专用服务器上的物品名会变英文

AE2 Web Integration 的物品名是**服务端**算好再发给浏览器的。而 1.7.10 的专用服务器有两道坎：

1. `FMLServerHandler.getCurrentLanguage()` 硬编码返回 `en_US`，服务器从不读 `assets/*/lang/zh_CN.lang`；
2. GT 自己的一套语言表（`GregTech.lang` / `GregTech_zh_CN.lang`）走的是客户端 `Minecraft` 类的加载路径，服务端启动时直接跳过；
3. GT 的机器名还走 `LanguageRegistry`，它只查自己的表，根本不看 `StringTranslate`。

所以客户端装了汉化也没用——网页拿到的仍然是服务器那份英文表。`ae2wi-namelang` 就是在服务端启动后，把这些中文表反射注入进 `StringTranslate` 和 `LanguageRegistry(en_US)`。

---

## 环境要求

| 项 | 要求 |
|---|---|
| Minecraft | 1.7.10 |
| Forge | 10.13.4.1614（GTNH 自带版本即可） |
| AE2 Web Integration | **2.0.0**（网页包是替换版，版本必须对得上） |
| 整合包 | GTNH 2.9.0（其他整合包也能用，但要自备 `config/txloader` 中文包） |
| Java | 8 及以上（本包 class 目标版本 52；服务器实跑 Java 17 已验证） |
| 端 | **仅服务端**。客户端不需要装 `ae2wi-namelang`（`acceptableRemoteVersions="*"`） |

---

## 安装

> 全程**先关服**：`mods/` 里的 jar 在服务器运行时被锁，覆盖会失败。

### 1. 网页界面包

把 `ae2webintegration-2.0.0-forge-1.7.10-zh_CN.jar` 覆盖到 `mods/`，替换原版同名文件。

### 2. 名字模组

把 `ae2wi-namelang-1.3.1.jar` 放进 `mods/`（这是新增文件，不是替换）。

### 3. 翻译数据

名字模组本身**不含译文**，它读的是服务器上已有的中文语言文件。GTNH 用户只需保证下面这些在位：

```
config/txloader/{load,forceload}/<包名>/lang/zh_CN.lang   官方汉化包 v2.9.0（约 296 个文件）
GregTech_zh_CN.lang                                       服务器根目录，GT 材质 / 机器名
```

### 4. 启动并刷新

启动后在日志里找这一行（英文，故意不用中文以免乱码）：

```
[AE2WI-Lang] [postInit] injected 528 lang file(s) / 88282 keys into the server translation table
for zh_CN (...) and 88282 into LanguageRegistry(en_US) for machine names ...
```

看到它，浏览器 **Ctrl+F5** 硬刷新即可。

---

## 配置

`config/ae2webintegration/name_language.txt`（首次启动自动生成）

```ini
zh_CN            # 第一行：目标语言。留空或写 en_US 等于关闭本功能
force=true       # 可选：客户端也强制生效（默认只在专用服务器上工作）
traditional=off  # 可选：不再用 zh_TW / zh_HK 给缺失的简体键兜底
```

`config/ae2webintegration/name_override.lang` —— 人工修正表，**优先级最高**，改完不用重编 jar：

```properties
# 键=想要的显示名
gt.metatool.01.16.name=扳手
item.botania:skyDirtRod.name=高原法杖
```

---

## 名字模组的读取顺序

后面的覆盖前面的（同一键以最后读到的为准）：

| 顺序 | 来源 | 说明 |
|---|---|---|
| 1 | `GregTech.lang`（根目录） | GT 英文表，只补空缺 |
| 2 | `mods/**/*.jar` 内的 `*/lang/zh_CN.lang` | 各 mod 自带翻译 |
| 3 | `nestedmods/` | 嵌套 mod 包 |
| 4 | `config/txloader/{load,forceload}` | GTNH 官方汉化包，**主力**，递归最多 4 层 |
| 5 | `GregTech_<locale>.lang`（根目录） | GT 材质名 / 机器名，Forge Configuration 格式 `S:"键"=值` |
| 6 | `config/ae2webintegration/name_override.lang` | 人工修正 |
| 兜底 | `zh_TW` / `zh_HK` | 只填简体缺失的键，绝不覆盖已有简体 |

---

## 已知限制

- **NBT 自定义名不翻译**：玩家用铁砧改过名的物品，网页显示的就是那个自定义名。
- **按 meta 拼键的物品**可能仍是键名：个别 mod（如 Amazing Trophies 的奖杯）用 `tile.xxx.<meta>.name` 这种运行时拼出来的键，语言表里本来就没有对应条目，英文环境同样显示键名。
- **原版物品未包含**：本包只处理 mod 物品，`minecraft:*` 仍按服务端默认（英文）。
- **服务端拼好的系统提示**也会跟着变中文，这是同一次注入的副作用，无害。
- 网页包里 `Damage` 一栏被改成「**子 ID（metadata）**」并去掉了千分位：这个字段是 `ItemStack.getItemDamage()`，是 metadata / 子 ID，不是耐久损耗。

---

## 更新日志

### `ae2wi-namelang` 1.3.1

- **修复带冒号的键被吞掉**：以前用 `java.util.Properties` 读 `.lang`，而 Properties 把 `:` 和空格也当分隔符，
  `item.botania:skyDirtRod.name=高原法杖` 被读成键 `item.botania`。所有 modid 前缀形式的键（汉化包里 4934 条）
  都受影响，表现为植物魔法的高原法杖、Thaumic Exploration 的寄生杖端等一直是英文。
  改为手写逐行解析（只认第一个 `=`，跳过 `#`，处理 `\n \r \t \" \\ \uXXXX`，剥 BOM）：键数 **83000 → 88282**。
- 顺带修掉把 `--Sojourner's Wand Cap` 这种无等号行当成条目的问题。

### `ae2wi-namelang` 1.3.0

- 翻译数据整包换成 GTNH 官方汉化包 **v2.9.0**；GT 中文表改用运行时 dump + 官方 2612 条覆盖（2875 条）。
- 新增 `GregTech_US.lang` 作为英文基准；旧包独有的 244 条补到 `forceload/____legacy_retained`。
- 支持任意含 `lang/` 段的路径（书籍、图鉴、资源包覆盖）、txloader 递归 4 层、`zh_TW/zh_HK` 兜底。

### `ae2wi-namelang` 1.2.x

- 1.2.3：修 `Aluminium锭`（GT 的 `S:key=value` 配置格式被 Properties 读成带 `S:` 的废键）；机器名注入 `LanguageRegistry(en_US)`，解决 GT 机器全英文。
- 1.2.2：注入量 6.5 万键。
- 1.2.1 / 1.2.0：语言表多来源合并、反射注入。

### 1.0.0（已废弃）

- 曾做成 Forge coremod，在 `injectData` 阶段触达 `cpw.mods.fml.common.Loader`，早于 AccessTransformer 安装，
  导致 `IllegalAccessError: LoaderV2 ... Loader.<init>()`，服务器直接起不来。
  **结论：这个功能不需要 coremod**，1.1.0 起改为最普通的 `@Mod`。

### 网页包 `zh_CN`

- 补全简体中文界面（`i18n.js` 324 条 + 主题 `translations.js` 345 条，共约 670 条文案）。
- 修正 `Damage` → `子 ID（metadata）`、`Stored` → `库存`、`Not craftable` → `不可合成(缺失样板)`。
- 只改动 `assets/web/` 下的 JS 文案，**class 与依赖库未动**，jar 仍是 1074 个条目。

---

## 文件校验

| 文件 | 大小 | MD5 |
|---|---|---|
| `ae2webintegration-2.0.0-forge-1.7.10-zh_CN.jar` | 14,025,135 字节 | `00f26d56ee8c95b9041edb3b9d8829aa` |
| `ae2wi-namelang-1.3.1.jar` | 25,666 字节 | `851499b675d02c90314f711b5278b1cb` |

---

## 卸载

- 网页包：换回原版 `ae2webintegration-2.0.0-forge-1.7.10.jar` 即可。
- 名字模组：删掉 `ae2wi-namelang-1.3.1.jar` 即可，**不留痕迹、不改存档**；`config/txloader` 等翻译数据留着无害。

---

## 致谢与许可

- 上游模组：**AE2 Web Integration** 2.0.0，作者 kuba6000 —— 本仓库仅分发修改后的 jar，版权归上游所有。
- 中文译文来源：GTNH 官方汉化包 **Translation-of-GTNH v2.9.0**，以及各 mod 自带 `zh_CN.lang` 的译者。
- `ae2wi-namelang` 为本项目自行编写（`javac --release 8` 直接编译，无需 ForgeGradle），源码暂未随本分支发布。

> 如果上游作者不接受修改版 jar 的再分发，请自行按下文从原版包重建，或联系本分支维护者索取源码。
