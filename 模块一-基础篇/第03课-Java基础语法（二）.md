# 第 03 课：Java 基础语法（二）——方法、数组与字符串

> 本课目标：学会把代码打包成可复用的"方法"，用"数组"管理成组的数据，以及操作游戏中最常见的文本（String）。
>
> 前置知识：第 02 课。

---

## 3.1 方法：把一段代码装进盒子

### 为什么需要方法？

想象你在模组里要计算伤害：玩家攻击时算一次、怪物攻击时算一次、陷阱触发时算一次……如果每处都抄一遍计算公式，改公式时要改十几处，迟早漏掉一处——bug 就是这么来的。

**方法（method）** 就是"给一段代码起个名字"，以后叫它的名字就能执行它。

### 定义和调用方法

```java
public class DamageDemo {

    public static void main(String[] args) {
        int result = calculateDamage(10, 3);   // 调用方法，传入 10 和 3
        System.out.println("造成的伤害: " + result);

        printLine();                           // 调用无参数的方法
    }

    // 定义一个方法：计算伤害 = 攻击力 - 防御力
    static int calculateDamage(int attack, int defense) {
        int damage = attack - defense;
        if (damage < 0) {
            damage = 0;      // 伤害不能为负
        }
        return damage;       // 把结果"还给"调用者
    }

    // 无参数、无返回值的方法
    static void printLine() {
        System.out.println("--------------------");
    }
}
```

拆解方法定义的五个部分：

```java
static   int      calculateDamage   (int attack, int defense)   { ... }
  ①       ②             ③                      ④                  ⑤
```

1. 修饰符：先固定写 `static`（第 04 课会解释为什么）；
2. **返回值类型**：方法执行完"还"给你什么类型的数据。`int` 表示还一个整数；如果什么都不还，写 `void`；
3. 方法名：驼峰命名，见名知意；
4. **参数列表**：括号里是方法需要的"原料"，调用时按顺序传入；
5. 方法体：花括号里的代码，`return` 语句负责把结果交回去。

> 💡 **类比**：方法像一台自动售货机。参数是你投的硬币和按的按钮，`return` 的值是掉出来的饮料。`void` 方法则像公告栏——只做事情（显示文字），不给你东西。

### return 的两个作用

1. 交回结果；
2. **立刻结束方法**——`return` 之后的代码不会执行：

```java
static int getExpForLevel(int level) {
    if (level < 0) {
        return 0;                 // 等级为负，直接返回，下面的代码不执行
    }
    return level * 100;
}
```

---

## 3.2 数组：一排连在一起的盒子

背包有 36 个格子，难道要写 36 个变量 `slot0, slot1, ..., slot35`？当然不。用**数组**：

```java
// 创建一个能装 5 个 int 的数组
int[] slots = new int[5];

slots[0] = 64;      // 给第 1 个格子赋值（注意：下标从 0 开始！）
slots[1] = 32;
System.out.println(slots[0]);    // 64
```

**重点：数组下标从 0 开始。** 长度为 5 的数组，合法下标是 0～4。访问 `slots[5]` 会直接报错（`ArrayIndexOutOfBoundsException`，新手老朋友）。

也可以在创建时直接装上数据：

```java
String[] tools = {"木剑", "石剑", "铁剑", "钻石剑", "下界合金剑"};
System.out.println(tools.length);   // 5（length 是数组的长度）
```

### 数组 + for 循环 = 黄金搭档

```java
String[] tools = {"木剑", "石剑", "铁剑", "钻石剑", "下界合金剑"};

for (int i = 0; i < tools.length; i++) {
    System.out.println("第 " + i + " 号工具: " + tools[i]);
}
```

还有一种更简洁的"增强 for 循环"（不需要下标时用）：

```java
for (String tool : tools) {          // 读作：对 tools 里的每一个 tool
    System.out.println(tool);
}
```

### 二维数组（了解即可）

箱子界面是 9 列 × 3 行的网格，可以用二维数组表示：

```java
int[][] chest = new int[3][9];     // 3 行 9 列
chest[0][0] = 64;                  // 第 1 行第 1 列放 64 个物品
```

---

## 3.3 字符串（String）：游戏中的文字处理

物品名、聊天消息、命令……Minecraft 里到处是字符串。

### 拼接

```java
String player = "小明";
String msg = "玩家 " + player + " 加入了游戏";
System.out.println(msg);   // 玩家 小明 加入了游戏
```

### 常用方法（String 自带的"功能"）

```java
String name = "Diamond Sword";

name.length();              // 12        长度
name.toLowerCase();         // "diamond sword"   全变小写
name.toUpperCase();         // "DIAMOND SWORD"   全变大写
name.contains("Sword");     // true      是否包含某段文字
name.startsWith("Diamond"); // true      是否以...开头
name.equals("diamond sword");         // false  内容是否相等（区分大小写）
name.equalsIgnoreCase("diamond sword"); // true  忽略大小写比较
name.substring(0, 7);       // "Diamond" 截取第 0 到第 6 个字符
```

> ⚠️ **判断字符串相等必须用 `.equals()`，不能用 `==`！**
>
> ```java
> String a = new String("hello");
> String b = new String("hello");
> System.out.println(a == b);        // false（比较的是"是不是同一个盒子"）
> System.out.println(a.equals(b));   // true（比较的是"盒子里的内容一不一样"）
> ```

### 字符串的"不可变"特性

String 一旦创建就不能修改，`toLowerCase()` 等方法其实是**生成了一个新字符串**，原字符串没变：

```java
String name = "Diamond";
name.toLowerCase();          // 生成新字符串，但没人接住它
System.out.println(name);    // 还是 "Diamond"

name = name.toLowerCase();   // 用变量接住新字符串，这才算"改了"
System.out.println(name);    // "diamond"
```

---

## 3.4 综合示例：一个简单的"背包统计"程序

把本课知识串起来——统计背包里某种物品的总数：

```java
public class BackpackDemo {

    public static void main(String[] args) {
        // 背包：每个元素是一个物品名，空格子用 "空" 表示
        String[] backpack = {"钻石", "木头", "钻石", "空", "钻石", "石头"};

        int count = countItem(backpack, "钻石");
        System.out.println("背包里共有 " + count + " 个钻石");
    }

    /**
     * 统计背包中指定物品的数量
     * @param backpack 背包数组
     * @param itemName 要找的物品名
     * @return 该物品的数量
     */
    static int countItem(String[] backpack, String itemName) {
        int count = 0;
        for (String slot : backpack) {
            if (slot.equals(itemName)) {   // 字符串比较用 equals
                count++;
            }
        }
        return count;
    }
}
```

运行结果：`背包里共有 3 个钻石`

注意 `countItem` 方法的设计：给它任何背包、任何物品名都能用——这就是方法的价值：**一次编写，到处调用**。

---

## 3.5 本课小结

- 方法 = 可复用的代码块；定义五要素：修饰符、返回值类型、方法名、参数、方法体；`return` 交回结果并结束方法。
- 数组 = 一排连续的同类型盒子；下标从 **0** 开始；`数组.length` 是长度；越界会报错。
- 字符串比较用 `.equals()` 不用 `==`；String 不可变，修改方法都返回新字符串。
- 方法上方的 `/** ... */` 是文档注释，是写注释的规范格式。

---

## 3.6 配套练习

> 答案见本模块 `答案/模块一答案与解析.md`。

**基础题**

1. 指出下面这个方法的：返回值类型、方法名、参数列表。

```java
static double getAverage(int sum, int count) {
    return (double) sum / count;
}
```

2. 数组 `int[] arr = new int[10];` 的合法下标范围是多少？`arr[10]` 会发生什么？
3. 字符串 `"Minecraft Forge"` 执行 `.toUpperCase()` 后是什么？执行 `.substring(0, 9)` 后是什么？

**提高题**

4. 写一个方法 `int countMaxStack(int[] stacks)`：传入一个"每组物品数量"的数组，返回其中数量等于 64 的组数。例如传入 `{64, 32, 64, 1}` 应返回 `2`。
5. 写一个方法 `String formatDamage(double damage)`：把伤害值格式化成统一格式的字符串返回，例如传入 `7.5` 返回 `"攻击力: 7.5 点"`。
6. 综合题：写一个方法 `String findStrongestTool(String[] tools, int[] damages)`，两个数组一一对应（`tools[i]` 的攻击力是 `damages[i]`），返回攻击力最高的工具的名字。例如 `{"木剑","钻石剑"}` 和 `{4, 7}` 应返回 `"钻石剑"`。如果数组为空，返回 `"无工具"`。
