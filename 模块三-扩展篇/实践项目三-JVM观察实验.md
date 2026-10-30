# 实践项目三：JVM 观察实验——解剖一次 Minecraft 启动

> 模块三偏理论，本项目用"做实验"的方式把前三课知识落到实处。
> 完成后对照 `答案/模块三答案与解析.md` 检查你的观察结论。

---

## 实验目标

不写新功能，而是**用 JVM 的视角重新观察你熟悉的游戏**：看类加载、看 GC、看字节码。做完这个实验，你对"Java 程序"四个字的理解会超过大多数只会写业务代码的人。

## 实验一：类加载观察（对应第 02 课）

1. 写一个简单的 Java 程序（普通 main 类即可）：

```java
public class LoadTest {
    static {
        System.out.println(">>> LoadTest 类初始化了");
    }

    public static void main(String[] args) {
        System.out.println(">>> main 方法开始");
    }
}
```

2. 先直接运行，观察两条打印的顺序。
3. 再加参数 `-verbose:class` 运行（IDEA：运行配置 → Modify options → Add VM options），在输出中找到：
   - `LoadTest` 自己的加载记录出现在什么位置？
   - 统计 `[class,load]` 的总行数（感受数量级）；
   - 找出 `java.lang.String` 是哪一行被加载的——它为什么那么靠前？

**思考问题**：静态代码块和 main 方法谁先执行？为什么？

## 实验二：GC 日志观察（对应第 03、05 课）

1. 在 runClient 的运行配置（build.gradle 的 client 段或 IDEA 运行配置）中添加 JVM 参数：

```
-Xlog:gc*:file=gc.log
```

2. 启动游戏，进入一个世界玩 3 分钟（跑一跑、挖挖方块），退出游戏。
3. 打开工程目录下的 `gc.log`，回答：
   - 一共发生了多少次 GC？其中几次是 Full GC？
   - 找出耗时最长的一次 GC，它停顿了多少毫秒？
   - 挑一条 Young GC 记录，写出"回收前 → 回收后"的内存变化。

4. 把 `-Xmx` 改成 512M 再跑一次同样的操作，对比 GC 频率和 Full GC 次数有什么变化。

## 实验三：字节码解剖（对应第 04 课）

1. 写一个类：

```java
public class BytecodeDemo {
    public int pickTool(int hardness) {
        if (hardness > 3) {
            return 4;   // 钻石镐
        } else {
            return 2;   // 铁镐
        }
    }
}
```

2. 编译后执行 `javap -c BytecodeDemo`（在 IDEA 底部 Terminal 里进入 `build/classes/java/main` 相应目录执行）。
3. 把 `pickTool` 方法的字节码完整抄下来，给每一条指令写上中文注释（参考第 04 课的指令表）。
4. 找出哪条指令对应 Java 代码里的 `if` 判断。

## 实验四（选做）：模组启动全景图

打开一次正常的 runClient 启动日志（控制台或 `logs/latest.log`），按顺序找出以下"里程碑"并抄下对应日志行：

1. JVM 参数生效（搜索 `Java is` 或 JVM 版本信息）；
2. Forge 开始扫描模组（搜索 `Scanning` 或 `mod`）；
3. 你的模组被构造（你自己加的 LOGGER.info 那行）；
4. 注册阶段开始；
5. 游戏进入主界面（搜索 `Backend library` 或 `Narrator`）。

## 提交物

一份实验报告（手写或 md 均可），包含：

- [ ] 实验一：两次运行的输出截图/文字、静态块与 main 的顺序结论
- [ ] 实验二：gc.log 的统计结果（GC 次数、Full GC 次数、最长停顿），以及 512M 对比结论
- [ ] 实验三：带中文注释的完整字节码 + if 对应的指令
- [ ] 实验四（选做）：5 个里程碑的日志行
- [ ] 用 200 字总结："当我双击 runClient 后，JVM 层面依次发生了什么"
