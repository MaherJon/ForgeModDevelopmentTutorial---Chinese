# 第 04 课：Java 进阶（二）——注解、反射与异常

> 本课目标：理解 `@Mod`、`@Override` 这些"@"符号背后的原理，初识反射，学会规范地处理异常。
>
> 前置知识：本模块第 03 课。

---

## 4.1 注解（Annotation）：贴在代码上的"标签"

### 注解是什么？

回看我们的主类：`@Mod(MyCoolMod.MODID)`。这个 `@Mod` 既不影响方法执行、也不是注释——它是**注解**：一种贴在类/方法/字段上的**元数据**（"描述数据的数据"），供框架和工具读取。

> 💡 **类比**：注解像行李上贴的托运标签。标签本身不改变行李的重量和内容，但机场系统（Forge）看到标签就知道"这件行李要送往哪里、怎么处理"。

### 你已经在用的注解

| 注解 | 贴在哪 | 含义 |
|---|---|---|
| `@Mod("id")` | 类 | 这是一个 Forge 模组 |
| `@Override` | 方法 | 此方法重写了父类方法（请编译器帮我检查） |
| `@Deprecated` | 类/方法 | 已过时，不建议继续使用 |
| `@Mod.EventBusSubscriber` | 类 | 自动把本类注册为事件监听者（第 05 课用） |
| `@SuppressWarnings("unchecked")` | 类/方法 | 让编译器闭嘴：别警告我了，我知道自己在做什么 |

### 自定义注解（看懂即可）

你也可以自己造标签：

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)   // 这个注解在运行时也能被读到
@Target(ElementType.METHOD)           // 只能贴在方法上
public @interface TestCase {
    String value();                   // 注解可以带参数
}

class MyTests {
    @TestCase("测试钻石剑伤害")
    void testSword() { ... }
}
```

- `@Retention(RUNTIME)`：注解活到什么时候（源码期 / 编译期 / 运行期）；
- `@Target(METHOD)`：允许贴在哪里。

### 注解自己不会动——关键认知

注解只是**死标签**，真正干活的是**读取标签的代码**。`@Mod` 之所以能让模组被加载，是因为 Forge 启动时会**扫描所有类、读取 `@Mod` 注解、然后 new 出你的模组对象**。怎么"扫描和读取"？靠反射。👇

---

## 4.2 反射（Reflection）：在运行时"透视"类

正常情况下，我们用 `new Sword()` 创建对象——这在**写代码时**就确定了类。而反射让程序在**运行时**动态地查看和操作任何类：

```java
// 普通方式：编译期就写死了
Sword s = new Sword();

// 反射方式：用一个字符串（类名）来操作类
Class<?> clazz = Class.forName("com.example.Sword");   // 拿到"类的说明书"
Object obj = clazz.getDeclaredConstructor().newInstance();  // 创建对象

// 甚至能调用私有方法、读取私有字段（"反射"之名由此而来：看到类的内部）
```

> 💡 **类比**：普通编程是"按图纸施工"；反射是"拿着探照镜检查一栋已建成的楼"——能看到每个房间（字段）、每部电梯（方法），甚至能进上锁的房间（private 成员）。

### 反射 + 注解 = 框架的魔法

Forge 启动时大致做了这些事（简化版）：

```java
// 伪代码：Forge 的模组发现逻辑
for (每个类 clazz : 扫描jar包里的所有类) {
    Mod annotation = clazz.getAnnotation(Mod.class);   // 反射读取注解
    if (annotation != null) {
        String modid = annotation.value();             // 拿到模组 ID
        Object modInstance = clazz.getDeclaredConstructor().newInstance(); // 创建模组对象
        模组列表.add(modInstance);
    }
}
```

这就是为什么你只要写个 `@Mod`，模组就"活"了——**你贴标签，Forge 用反射找到标签并执行相应动作**。模块三讲类加载时你会对这套机制有更深的理解。

> ⚠️ 反射强大但慢、且能破坏封装，只应在框架层使用。日常模组代码里几乎不需要自己写反射。

---

## 4.3 异常（Exception）：程序的"意外事故"处理

### 认识异常家族

程序运行中出错（文件不存在、网络断了、数组越界），Java 会抛出一个**异常对象**。不处理的话，程序直接崩溃。

```
Throwable
├── Error           ← 严重错误（内存溢出等），基本无法挽救，不用管
└── Exception
    ├── RuntimeException（运行时异常）   ← 程序员的 bug：NPE、数组越界
    │                                    编译器不强制处理
    └── 受检异常（checked exception）    ← 外界不可控因素：IOException 等
                                         编译器强制你处理
```

### try-catch-finally：处理异常的标准姿势

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public void saveData(String data) {
    try {
        // 可能出事的代码放这里
        Files.writeString(Path.of("config.txt"), data);
        System.out.println("保存成功");
    } catch (IOException e) {
        // 出事了怎么办：捕获异常，优雅处理而不是崩溃
        System.err.println("保存失败: " + e.getMessage());
    } finally {
        // 无论成功与否都执行（常用于释放资源）
        System.out.println("保存流程结束");
    }
}
```

- `try`：把"可能爆炸"的代码围起来；
- `catch`：爆炸后接住它，执行善后；
- `finally`：无论如何都执行（可做可不做的事，如关闭文件）。

### 主动抛出异常：throw

自己写方法时，遇到不合理的输入可以主动抛异常，把问题暴露给调用者：

```java
public static int divide(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("除数不能为 0！");
    }
    return a / b;
}
```

### 什么时候该 catch？

- 操作**外部资源**（文件、网络）时必须处理——你不知道玩家的磁盘是否满了；
- 纯内存中的逻辑错误（NPE、越界）**不要用 catch 掩盖**，那是 bug，应该修复代码本身。

> ⚠️ 模组开发大忌：`catch (Exception e) {}` ——空 catch 块。出了错一声不吭，以后排查问题会发现"什么都没报错但功能就是不正常"，等于把警报器拆了。

---

## 4.4 本课小结

- 注解是贴在代码上的元数据标签；`@Mod`、`@Override` 都是注解。注解本身不干活，读取注解的代码才干活。
- 反射让程序在运行时动态查看/操作类，是 Forge 扫描 `@Mod`、创建模组对象的底层手段；强大但慢，日常代码少用。
- 异常分受检异常（必须处理）和运行时异常（多为 bug）；用 `try-catch-finally` 优雅处理，绝不写空 catch。
- 主动 `throw` 异常可以让错误尽早暴露。

---

## 4.5 配套练习

> 答案见本模块 `答案/模块二答案与解析.md`。

**基础题**

1. `@Override` 注解的作用是什么？如果不写它，会发生什么？
2. 受检异常和运行时异常的区别是什么？各举一个例子。
3. 下面代码有什么问题？

```java
try {
    Files.writeString(Path.of("a.txt"), "hello");
} catch (IOException e) {
}
```

**提高题**

4. 写一个方法 `int parseLevel(String input)`：把字符串转成 int 等级。如果输入不是数字（`Integer.parseInt` 会抛 `NumberFormatException`），捕获异常并返回 `-1` 表示"无效输入"。
5. 写一个方法 `void setStackSize(Item.Properties props, int size)`：要求 size 必须在 1～64 之间，否则抛出 `IllegalArgumentException`，异常消息写清楚合法范围。
6. 用反射完成小实验：写任意一个带无参构造方法的类 `Hello`（含一个 `sayHi()` 方法），在另一个类里用 `Class.forName` + `newInstance` 创建它并调用 `sayHi()`。（提示：调用普通方法可以用 `clazz.getMethod("sayHi").invoke(obj)`。）
7. 思考题：Forge 为什么选择在"运行时扫描注解"而不是让你在某个配置文件中手动登记"我的模组主类是 xxx"？两种方式各有什么优缺点？
