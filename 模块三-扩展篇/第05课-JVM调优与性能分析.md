# 第 05 课：JVM 调优与性能分析——给 Minecraft 分配合适的内存

> 本课目标：看懂 JVM 启动参数，会给 Minecraft 合理分配内存、读懂 GC 日志，用性能观察定位卡顿来源。
>
> 前置知识：本模块第 03 课（内存与 GC）。

---

## 5.1 JVM 参数：启动时的"调节旋钮"

启动 Java 程序时可以传一堆 `-X` 开头的参数调教 JVM。看一条典型的 Minecraft 启动命令：

```powershell
java -Xms2G -Xmx4G -XX:+UseG1GC -jar game.jar
```

### 最常用的参数

| 参数 | 含义 | 建议 |
|---|---|---|
| `-Xms2G` | 堆**初始**大小 2GB | 一般和 Xmx 设成一样，避免堆扩容的抖动 |
| `-Xmx4G` | 堆**最大**大小 4GB | 见下面的"黄金区间" |
| `-XX:+UseG1GC` | 使用 G1 垃圾回收器 | MC 1.20 的默认推荐，停顿较短 |
| `-XX:MaxGCPauseMillis=200` | 告诉 G1 目标停顿时间 | G1 会尽量把单次停顿控制在 200ms 内 |
| `-verbose:gc` / `-Xlog:gc*` | 打印 GC 日志 | 排查卡顿来源时打开 |
| `-XX:+HeapDumpOnOutOfMemoryError` | OOM 时导出内存快照 | 排查内存泄漏的神器 |

### 内存分配的"黄金区间"（经验值）

| 场景 | 建议 -Xmx |
|---|---|
| 原版/少量模组 | 2～4 GB |
| 中型整合包（100+ 模组） | 4～6 GB |
| 大型整合包（200+ 模组） | 6～8 GB |
| 更多？ | **通常适得其反** |

> ⚠️ 第 03 课的结论重温：堆越大，Full GC 一次扫描的对象越多，停顿越长。给 MC 分配 32GB 的人往往获得"平时很流畅、定期大卡顿"的体验。4～8GB 是甜点区。

---

## 5.2 读懂 GC 日志

加参数 `-Xlog:gc*`（JDK 9+ 的统一日志语法）启动游戏，日志里会出现：

```
[gc] GC(125) Pause Young (Normal) (G1 Evacuation Pause) 512M->320M(4096M) 8.5ms
```

逐段解读：

- `Pause Young`：一次**年轻代 GC**（Minor GC）；
- `512M->320M`：GC 前堆占用 512MB，GC 后剩 320MB（回收了 192MB）；
- `(4096M)`：堆总容量 4GB；
- `8.5ms`：本次停顿 8.5 毫秒——毫无体感。

**需要警惕的信号**：

```
[gc] GC(300) Pause Full (G1 Compaction Pause) 3800M->3700M(4096M) 1200ms
```

- `Pause Full`：**Full GC**；
- 停顿 `1200ms` = 1.2 秒——玩家明显卡了一下；
- `3800M->3700M`：扫了整个堆只清出 100MB——堆快满了，GC 再努力也清不出空间。

看到频繁 Full GC 的对策：① 内存不够就适当加大 Xmx；② 内存够却频繁 Full GC，可能是某些模组在泄漏内存（对象只进不出），需要进一步分析。

---

## 5.3 实战观察工具

### JDK 自带工具（无需安装任何东西）

| 工具 | 用途 |
|---|---|
| `jps` | 列出当前所有 Java 进程（找到游戏的 PID） |
| `jstat -gc PID 1000` | 每秒打印一次该进程的 GC 统计（看各区占用和 GC 次数/耗时） |
| `jmap -dump:live,format=b,file=heap.bin PID` | 导出堆快照 |
| `jconsole` / `jvisualvm`（图形界面） | 实时看内存曲线、线程、CPU |

### 游戏内工具

- **F3 调试界面**：右上角实时显示内存占用百分比——玩着玩着百分比冲到 90% 以上又骤降，就是 GC 在干活；
- **F3 + Alt 性能饼图**：看每帧时间花在哪（渲染？实体？）。

---

## 5.4 给模组开发者的性能建议

作为模组作者，你的代码质量直接影响玩家的 GC 压力：

1. **别在 tick 事件里疯狂 new 对象**：每 tick 执行的代码里创建大量临时对象 = 给 GC 加班。能复用的对象做成字段复用。
2. **高频路径避免装箱**：`Integer` 是对象、`int` 不是。循环里优先用基本类型。
3. **字符串拼接用 StringBuilder**：循环里 `"a" + "b"` 每次都产生新 String 对象。
4. **事件处理早返回**：不满足条件立刻 return，别做无用功。

```java
// 差：每秒 new 20 次 StringBuilder 都救不了的浪费
@SubscribeEvent
public static void onTick(TickEvent.PlayerTickEvent event) {
    String msg = "";
    for (int i = 0; i < 100; i++) {
        msg += i;                 // 每次都新建 String 对象！
    }
}

// 好：
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++) {
    sb.append(i);
}
String msg = sb.toString();
```

---

## 5.5 本课小结

- 关键 JVM 参数：`-Xms/-Xmx`（堆大小）、`-XX:+UseG1GC`（回收器）、`-Xlog:gc*`（GC 日志）。
- MC 内存甜点区 4～8GB，过大反而加剧 Full GC 停顿。
- GC 日志关注：`Full` 字样、停顿毫秒数、回收前后占用。
- 自带工具：`jps`、`jstat`、`jconsole`；游戏内 F3 看内存曲线。
- 模组代码性能守则：高频路径少 new 对象、用基本类型、字符串拼接用 StringBuilder、事件早返回。

---

## 5.6 配套练习

> 答案见本模块 `答案/模块三答案与解析.md`。

**基础题**

1. `-Xms2G -Xmx4G` 中两个参数分别控制什么？为什么建议把它们设成相同的值？
2. GC 日志中出现什么字样说明发生了 Full GC？看到 `Pause Full ... 900ms` 玩家会有什么体感？
3. 为什么给 Minecraft 分配过大的内存反而可能导致卡顿？

**提高题**

4. 启动开发版游戏（runClient），进入世界后按 F3，观察右上角内存占用百分比的变化规律，描述你看到的"缓慢上升 → 突然下降"现象并用本课知识解释。
5. 用 `jps` 找到游戏进程 PID，再用 `jstat -gc PID 1000` 观察 30 秒，记录 YGC（年轻代 GC 次数）和 YGCT（年轻代 GC 总耗时）的变化。
6. 找出下面这段代码的性能问题并改写：

```java
@SubscribeEvent
public static void onPlayerTick(TickEvent.PlayerTickEvent event) {
    List<String> names = new ArrayList<>();
    for (int i = 0; i < 50; i++) {
        names.add("marker_" + i);
    }
    String label = "";
    for (String n : names) {
        label = label + n + ",";
    }
}
```

7. 思考题：一个整合包玩家抱怨"每过几分钟就卡一秒"。你作为排查者，按本课知识列出排查步骤（先做什么、再看什么、最后得出什么结论）。
