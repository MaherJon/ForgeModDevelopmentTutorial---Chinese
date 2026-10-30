# 第 01 课：Gradle 构建系统详解

> 本课目标：看懂 `build.gradle`，理解构建系统到底替你做了什么，会给工程添加第三方依赖。
>
> 前置知识：模块一全部。

---

## 1.1 构建系统是干什么的？

回想一下：你的模组从一堆 `.java` 文件，变成玩家能装进游戏的 `.jar` 文件，中间发生了什么？

```
.java 源代码 → 编译 → .class 字节码 → 打包 → mycoolmod-1.0.0.jar
                  ↑ 还要下载 Minecraft、Forge、各种库，
                    还要把资源文件一起塞进去……
```

这一整套流程如果手动做，每改一行代码都要敲十几条命令。**构建系统**就是帮你自动化这一切的管家，而 Gradle 是 Java 世界最流行的构建系统之一。

> 💡 **类比**：Gradle 像一位任劳任怨的大厨。你只要在"菜谱"（build.gradle）里写明"要什么食材、怎么做"，喊一声"开饭"（运行任务），它就自动完成采购（下载依赖）、烹饪（编译）、装盘（打包）。

---

## 1.2 Gradle 的几个核心概念

| 概念 | 说明 | 类比 |
|---|---|---|
| **构建脚本** | `build.gradle`，用 Groovy 语法写的配置文件 | 菜谱 |
| **插件（Plugin）** | 给 Gradle 增加新能力的扩展 | 大厨学会的"菜系" |
| **依赖（Dependency）** | 你的工程需要用到的别人的代码库 | 食材 |
| **任务（Task）** | 一个具体的构建动作（编译、运行游戏、打包……） | 一道工序 |
| **Gradle Wrapper** | `gradlew`，自带固定版本 Gradle 的启动器 | 大厨本人随叫随到 |

> 💡 **为什么用 `gradlew` 而不是自己装 Gradle？** Wrapper 会自动下载工程指定的 Gradle 版本，保证所有人用完全一致的构建环境。你永远不会遇到"我电脑上能构建，你电脑上不行"的问题。

---

## 1.3 逐段看懂 build.gradle

打开工程根目录的 `build.gradle`，我们挑关键部分讲（不同 MDK 版本细节略有差异，思路相同）：

### 插件声明

```groovy
plugins {
    id 'java'                                    // Java 编译能力
    id 'idea'                                    // 与 IDEA 集成
    id 'net.minecraftforge.gradle' version '[6.0,6.2)'   // Forge 官方插件
}
```

`net.minecraftforge.gradle`（简称 ForgeGradle）是灵魂插件——下载 Minecraft、反混淆游戏代码、提供 `runClient` 任务，全靠它。

### 工程坐标与版本

```groovy
group = 'com.mycooldev'    // 你的"组织名"（通常是包名前缀）
version = '1.0.0'          // 模组版本号，打包出的 jar 文件名会带上它
```

### Java 版本

```groovy
java {
    toolchain.languageVersion = JavaLanguageVersion.of(17)  // 用 Java 17 编译
}
```

### 依赖

```groovy
dependencies {
    minecraft 'net.minecraftforge:forge:1.20.1-47.2.0'   // 依赖 Minecraft + Forge
}
```

格式 `'组:名:版本'`。Gradle 会自动从网上（Maven 仓库）把它下载下来。

### 运行配置

```groovy
runs {
    client {                     // 定义名为 client 的运行配置 → 生成 runClient 任务
        workingDirectory project.file('run')   // 游戏的"存档/配置"目录
        mods {
            mycoolmod {          // 把你的模组放进这个运行实例
                source sourceSets.main
            }
        }
    }
}
```

想启动服务端？复制一段改名 `server`，就会多出一个 `runServer` 任务。

---

## 1.4 常用 Gradle 任务速查

在 IDEA 右侧 Gradle 面板双击，或在工程目录用命令行执行 `gradlew 任务名`：

| 任务 | 作用 |
|---|---|
| `runClient` | 启动开发版游戏客户端 |
| `runServer` | 启动开发版服务端 |
| `build` | 完整构建：编译 + 测试 + 打包出 `.jar`（产物在 `build/libs/` 下） |
| `runData` | 运行数据生成（第 07 课讲） |
| `clean` | 清除构建产物（`build/` 目录） |

> 💡 打出最终发布给玩家的 jar：运行 `build`，然后到 `build/libs/` 里找 `mycoolmod-1.0.0.jar`。把它发给玩家，放进他们的 `mods` 文件夹即可使用。

---

## 1.5 添加第三方依赖：用别人的轮子

想在你的模组里使用一个现成的 Java 库（比如 JSON 处理库）？往 `dependencies` 里加一行：

```groovy
dependencies {
    minecraft 'net.minecraftforge:forge:1.20.1-47.2.0'

    // 例子：引入一个常用工具库 Guava（Google 出品）
    implementation 'com.google.guava:guava:33.0.0-jre'
}
```

改完 `build.gradle` 后，点击 IDEA 右上角弹出的 **Load Gradle Changes**（大象刷新图标），Gradle 就会下载新依赖。

### 配置国内镜像加速（可选但强烈推荐）

首次构建慢，多半是卡在从国外仓库下载。打开 `settings.gradle`（或 `build.gradle` 顶部的仓库配置），把国内镜像加到 `repositories` 的**最前面**：

```groovy
repositories {
    maven { url = 'https://maven.aliyun.com/repository/public' }   // 阿里云镜像
    // ...原有的仓库保持不动，放在下面作为后备
}
```

> ⚠️ 镜像只能加速"普通依赖"；Minecraft/Forge 本体仍需从官方源下载，首建慢是正常现象。

---

## 1.6 settings.gradle 与 gradle.properties

- `settings.gradle`：声明工程名、插件仓库地址等全局设置，一般不用动。
- `gradle.properties`：MDK 用它集中存放版本号等属性，`build.gradle` 里用 `${属性名}` 引用。把版本号集中在这里管理，升级版本时只需改一处——这就是"配置文件"的意义。

---

## 1.7 本课小结

- Gradle 是自动化构建管家：下载依赖、编译、运行游戏、打包 jar 一条龙。
- `build.gradle` 是核心配置：插件、坐标、Java 版本、依赖、运行配置。
- `gradlew` 保证所有人用同一版本 Gradle；常用任务：`runClient`、`build`。
- 加依赖只需在 `dependencies` 里加一行 `'组:名:版本'`，然后刷新 Gradle。
- 国内网络可配置阿里云镜像加速。

---

## 1.8 配套练习

> 答案见本模块 `答案/模块二答案与解析.md`。

**基础题**

1. `build.gradle` 中 `plugins`、`dependencies`、`runs` 三个段落分别负责什么？
2. 用 Wrapper 执行构建的完整命令（Windows 命令行）是什么？执行后产物 jar 在哪个目录？
3. 依赖声明 `'com.google.guava:guava:33.0.0-jre'` 的三个部分分别是什么？

**提高题**

4. 把你的模组版本号改成 `1.1.0`，执行 `build` 任务，找到并确认生成的 jar 文件名。
5. 在 `runs` 段落里参照 `client` 配置新增一个 `server` 配置，刷新 Gradle 后确认出现 `runServer` 任务，并启动一次服务端（看到 "Done" 字样即成功，然后输入 `stop` 关闭）。
6. 思考题：为什么 Minecraft 模组工程不用"手动下载各种 jar 再拖进 lib 文件夹"的方式管理依赖？Gradle 的方式解决了哪些痛点？（提示：想想版本升级、多人协作、依赖的依赖。）
