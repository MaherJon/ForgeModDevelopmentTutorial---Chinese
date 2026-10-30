# 第 05 课：Forge 入门与模组结构

> 本课目标：看懂 Forge 模组工程的目录结构，理解模组的加载流程，写出属于你自己的模组主类。
>
> 前置知识：第 01～04 课。

---

## 5.1 模组工程的"地图"

打开第 01 课解压的工程，完整的目录结构是这样的：

```
MyFirstMod/
├── build.gradle                ← 构建配置（模块二详解，现在不用动）
├── settings.gradle
├── gradlew / gradlew.bat       ← Gradle 启动器
└── src/
    └── main/
        ├── java/               ← 【你的 Java 代码】放这里
        │   └── com/example/examplemod/
        │       └── ExampleMod.java
        └── resources/          ← 【资源文件】贴图、模型、语言文件放这里
            ├── META-INF/
            │   └── mods.toml   ← 模组"档案表"（名字、版本、介绍）
            └── pack.mcmeta     ← 资源包信息
```

**两个最重要的文件夹，一句话记住：**

- `src/main/java` → 放**代码**（决定模组"做什么"）
- `src/main/resources` → 放**资源**（决定模组"长什么样"）

---

## 5.2 打造你自己的模组主类

我们来把示例模组改造成自己的模组。假设你的模组叫 **MyCoolMod**。

### 第一步：建包（package）

"包"就是 Java 代码的文件夹，用来防止重名冲突。规范是倒过来的域名，比如 `com.yourname.mycoolmod`。没有域名？用 `top.你的拼音.mycoolmod` 也行。

在 IDEA 中右键 `src/main/java` → **New → Package**，输入：

```
com.mycooldev.mycoolmod
```

### 第二步：创建主类

右键新建的包 → **New → Java Class**，名字 `MyCoolMod`，写入：

```java
package com.mycooldev.mycoolmod;   // 声明这个类属于哪个包（必须和文件夹对应）

import net.minecraftforge.fml.common.Mod;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Mod(MyCoolMod.MODID)   // 关键注解：告诉 Forge "我是模组，我的ID在下面"
public class MyCoolMod {

    // 模组 ID：全小写英文+数字+下划线，是模组的唯一身份证，之后到处要用
    public static final String MODID = "mycoolmod";

    // 日志器：往游戏日志里输出信息，调试全靠它
    private static final Logger LOGGER = LoggerFactory.getLogger(MyCoolMod.class);

    public MyCoolMod() {
        LOGGER.info("我的第一个模组加载啦！");
    }
}
```

逐块解释：

1. `package com.mycooldev.mycoolmod;` —— 必须是文件第一行，声明"本类住在哪个文件夹"。
2. `import ...` —— 引入别人写好的类。想用 `Mod` 这个注解，就要先 import 它，否则 Java 不知道它是谁。（IDEA 可以按 `Alt + Enter` 自动补 import。）
3. `@Mod("mycoolmod")` —— **注解**（Annotation），以 `@` 开头，是贴在代码上的"标签"。Forge 启动时扫描所有类，看到 `@Mod` 标签就知道"这是个模组，要加载它"。
4. `public static final String MODID = "mycoolmod"` —— 模组 ID 常量。规范：**全小写**，不要有空格和大写字母。第 02 课说过 `static final` 表示全局唯一的常量。
5. `LOGGER.info(...)` —— 日志输出。运行后能在控制台和游戏日志文件里看到，是你排查问题的好帮手。

### 第三步：同步修改 mods.toml

打开 `src/main/resources/META-INF/mods.toml`，找到这两处改掉（其他地方可以先保持默认）：

```toml
modLoader="javafml"
loaderVersion="[47,)"
license="MIT"

[[mods]]
modId="mycoolmod"               # ← 必须和 Java 里的 MODID 完全一致！
version="1.0.0"
displayName="My Cool Mod"       # ← 游戏里显示的名字
description='''
这是我写的第一个模组！
'''
authors="你的名字"
```

> ⚠️ `modId` 必须和 Java 代码里的 MODID **一字不差**，否则 Forge 找不到你的模组，报错 "missing mod"。

删掉旧的 `ExampleMod.java` 和 `com.example.examplemod` 包（右键 → Delete），然后运行 `runClient`——在 Mods 列表里找到 **My Cool Mod**，就大功告成了！

---

## 5.3 模组的生命周期：Forge 是怎么加载你的模组的

启动游戏时，Forge 会按固定顺序经历几个**生命周期阶段**。理解它，才知道代码该写在哪。

```
游戏启动
   │
   ▼
① 构造模组对象      ← 你的主类构造方法执行（new MyCoolMod()）
   │
   ▼
② 注册（Registry）   ← 所有模组把自己的物品/方块登记进"花名册"
   │
   ▼
③ 通用设置（commonSetup） ← 适合做初始化逻辑
   │
   ▼
④ 进入游戏主界面     ← 玩家可以开始玩了
```

对应到代码，你目前需要认识两个东西：

### 模组事件总线（Mod Event Bus）

Forge 内部有一套"广播系统"：游戏进行到哪一步，就通过"总线"广播对应的**事件**。你想在哪个阶段做事，就"收听"哪个事件。

```java
import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.fml.event.lifecycle.FMLCommonSetupEvent;
import net.minecraftforge.fml.javafmlmod.FMLJavaModLoadingContext;
import net.minecraftforge.eventbus.api.IEventBus;

@Mod(MyCoolMod.MODID)
public class MyCoolMod {

    public static final String MODID = "mycoolmod";

    public MyCoolMod() {
        // 拿到"模组事件总线"（可以把它理解为本模组专用的广播收音机）
        IEventBus modEventBus = FMLJavaModLoadingContext.get().getModEventBus();

        // 订阅"通用设置"事件：到了这个阶段，就执行我们的 commonSetup 方法
        modEventBus.addListener(this::commonSetup);
    }

    private void commonSetup(final FMLCommonSetupEvent event) {
        // 这里放初始化逻辑，例如：
        System.out.println("通用设置阶段完成！");
    }
}
```

关键概念（现在只需建立印象，模块二会深挖）：

- **事件（Event）**：游戏过程中发生的事（"玩家受伤了""方块被挖了""注册开始了"）；
- **事件总线（Event Bus）**：事件的广播站；
- **监听（Listen / Subscribe）**：告诉总线"某事件发生时请叫我"，并提供处理方法。
- `this::commonSetup` 是"方法引用"，意思是"把 commonSetup 这个方法交给总线，事件发生时帮我调用它"。

---

## 5.4 本课小结

- 代码放 `src/main/java`，资源放 `src/main/resources`。
- 模组主类三要素：`@Mod` 注解、MODID 常量、构造方法。
- `mods.toml` 里的 `modId` 必须和代码里的 MODID 一致。
- Forge 按生命周期加载模组；想介入某个阶段，就通过事件总线订阅对应事件。
- 遇到不认识的类，用 IDEA 的 `Alt + Enter` 自动 import。

---

## 5.5 配套练习

> 答案见本模块 `答案/模块一答案与解析.md`。

**基础题**

1. Java 代码和资源文件分别放在工程的哪两个目录下？
2. 模组 ID 的命名规范是什么？`"MyCoolMod"`、`"my cool mod"`、`"mycoolmod"` 三个写法哪个合法？
3. `mods.toml` 中哪个字段必须和 Java 代码里的 MODID 保持一致？不一致会发生什么？

**提高题**

4. 按照本课步骤，把示例模组改造成名为 `adventure`（冒险）的模组：包名 `com.adventure.adventuremod`，显示名 "Adventure Mod"，作者填你的名字。运行 runClient 并在 Mods 列表确认显示正确，描述你改了哪几个文件。
5. 在主类构造方法里加一行日志 `"冒险模组正在加载，版本 1.0.0"`，运行后在控制台找到这行日志，描述它大概在哪些日志信息附近出现（这能帮你理解模组在游戏启动的哪个阶段执行）。
6. 思考题：为什么 Forge 要用"事件总线"这种设计，而不是直接让你在 main 方法里从头写到尾？（提示：想想模组和游戏本体、以及模组与模组之间的关系。）
