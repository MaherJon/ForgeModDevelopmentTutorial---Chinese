# 第 02 课：Java 基础语法（一）——变量、数据类型与流程控制

> 本课目标：学会 Java 的"单词"和"句子"——变量、基本数据类型、运算符、if 判断和循环。
>
> 前置知识：第 01 课（环境已搭好）。

---

## 2.1 先写一个能跑的 Java 程序

学语法最好的方式是边看边敲。我们暂时不碰 Minecraft，先写一个纯 Java 小程序。

在 IDEA 里右键 `src/main/java` → **New → Java Class**，名字填 `HelloJava`，然后把内容改成：

```java
public class HelloJava {
    public static void main(String[] args) {
        System.out.println("你好，Java！");
    }
}
```

点 `main` 左边的绿色小三角 ▶ 运行，下方控制台会打印出 `你好，Java！`。

逐行解释：

- `public class HelloJava`：定义一个名叫 `HelloJava` 的**类**（class）。Java 里所有代码都必须写在类里。文件名必须和类名一致。
- `public static void main(String[] args)`：**main 方法**，是程序的入口。程序从这里开始执行。
- `System.out.println(...)`：向控制台打印一行文字。`println` = print line（打印并换行）。
- 每句代码末尾的**分号 `;`**：相当于中文的句号，表示"这句话说完了"。

> 💡 **类比**：类是"房子"，方法是"房间里的功能"（比如厨房用来做饭），而 `main` 是房子的"大门"——程序进门后从大门开始走。

---

## 2.2 变量：给数据起个名字

**变量**就是一个"贴了标签的盒子"，盒子里装着数据，标签就是变量名。

```java
int playerHealth = 20;        // 声明一个整数变量，装玩家生命值
playerHealth = 15;            // 可以换内容（所以叫"变"量）
```

格式：`数据类型 变量名 = 初始值;`

### 常用基本数据类型

| 类型 | 装什么 | 例子 | 在 Minecraft 里的用途 |
|---|---|---|---|
| `int` | 整数 | `int damage = 7;` | 伤害值、耐久度、数量 |
| `double` | 小数（双精度） | `double speed = 0.1;` | 移动速度、坐标 |
| `boolean` | 真/假 | `boolean isOnFire = true;` | 是否着火、是否潜行 |
| `char` | 单个字符 | `char grade = 'A';` | 用得较少 |
| `String` | 一串文字（不是基本类型，但极常用） | `String name = "钻石剑";` | 物品名、聊天信息 |

```java
String itemName = "超级苹果";
int stackSize = 64;
double attackDamage = 10.5;
boolean canFly = false;

System.out.println(itemName);        // 打印：超级苹果
System.out.println(stackSize + 1);   // 打印：65
```

### 变量命名规则（必须遵守）与规范（约定俗成）

- 规则：只能由字母、数字、下划线、`$` 组成，**不能以数字开头**，不能是 Java 的保留字（如 `class`、`int`）。
- 规范：使用**驼峰命名法**——第一个单词小写，后面每个单词首字母大写，如 `playerHealth`、`maxStackSize`。

---

## 2.3 运算符：对变量做计算

```java
int a = 10, b = 3;

System.out.println(a + b);   // 13  加
System.out.println(a - b);   // 7   减
System.out.println(a * b);   // 30  乘
System.out.println(a / b);   // 3   除（整数除法，小数部分被丢掉！）
System.out.println(a % b);   // 1   取余（10 除以 3 余 1）
```

> ⚠️ **新手大坑**：`10 / 3` 在 Java 里结果是 `3` 不是 `3.33...`，因为两个整数相除结果还是整数。想要小数结果，至少一边要写成小数：`10.0 / 3`。

还有两个超好用的简写：

```java
int health = 20;
health = health - 1;   // 掉 1 滴血，等价于下面这行
health -= 1;           // 简写形式
health--;              // 再简写：自减 1（还有 ++ 自增 1）
```

### 比较与逻辑运算（结果是 boolean）

```java
System.out.println(10 > 3);    // true   大于
System.out.println(10 == 3);   // false  等于（注意是两个等号！）
System.out.println(10 != 3);   // true   不等于

boolean hasKey = true;
boolean doorLocked = false;
System.out.println(hasKey && !doorLocked);  // true（有钥匙 且 门没锁）
```

- `&&`：并且（两个都真才真）
- `||`：或者（有一个真就真）
- `!`：取反（真变假）

> ⚠️ `=` 是赋值（把右边的值放进左边），`==` 是比较（判断左右是否相等）。写错是新手最常见的 bug 之一。

---

## 2.4 if 判断：让程序会"思考"

游戏里的逻辑："如果玩家生命值低于 5，就显示红屏警告"。翻译成 Java：

```java
int health = 4;

if (health < 5) {
    System.out.println("危险！生命值过低！");
} else if (health < 10) {
    System.out.println("受伤了，小心点。");
} else {
    System.out.println("状态良好。");
}
```

结构：

```
if (条件1) {
    条件1 为真时执行
} else if (条件2) {
    条件1 为假但条件2 为真时执行
} else {
    以上都为假时执行
}
```

- `else if` 可以有任意多个，也可以没有；`else` 最多一个，也可以没有。
- 花括号 `{}` 包住的是"这一块代码属于谁"。

### switch：多选一的场景

当需要根据一个变量的具体值分很多种情况时，`switch` 比一长串 `else if` 更清晰：

```java
String tool = "剑";

switch (tool) {
    case "剑":
        System.out.println("近战攻击！");
        break;                          // 别漏了 break，否则会"穿透"到下一个 case
    case "弓":
        System.out.println("远程射击！");
        break;
    default:                            // 以上都不匹配时执行
        System.out.println("徒手攻击。");
}
```

---

## 2.5 循环：让电脑做重复的事

游戏代码充满了重复：遍历背包里每一格、给周围每个怪物造成伤害……这些都靠循环。

### for 循环：知道要重复几次时用

```java
// 输出背包前 9 格的编号
for (int i = 0; i < 9; i++) {
    System.out.println("检查第 " + i + " 格");
}
```

拆解 `for (int i = 0; i < 9; i++)`：

1. `int i = 0`：循环开始前，先造一个计数器 `i`，从 0 开始；
2. `i < 9`：每次循环前检查这个条件，为真才继续；
3. `i++`：每循环完一圈，`i` 加 1。

所以循环体会执行 9 次（i 依次为 0～8）。

### while 循环：不知道次数、只看条件时用

```java
int hunger = 20;
while (hunger > 0) {
    System.out.println("当前饥饿值: " + hunger);
    hunger -= 4;   // 每圈扣 4 点
}
// 输出 20、16、12、8、4，hunger 变成 0 后条件不满足，循环结束
```

> ⚠️ 循环里一定要有让条件最终变为假的操作（比如 `hunger -= 4`），否则会变成**死循环**——程序永远卡住。

### 循环控制：break 与 continue

```java
for (int i = 0; i < 36; i++) {
    if (i == 5) {
        break;        // 直接结束整个循环（找到钻石了，不找了）
    }
    if (i % 2 == 0) {
        continue;     // 跳过本次，直接开始下一圈（偶数格子不检查）
    }
    System.out.println("认真检查第 " + i + " 格");
}
```

---

## 2.6 注释：写给人看的文字

```java
// 这是单行注释，两个斜杠后面的内容程序不会执行

/*
 * 这是多行注释
 * 可以写很多行
 */

int damage = 7; // 注释也可以写在代码后面
```

好的注释解释"为什么这么做"，而不是复述代码。模组开发中，几个月后的你会感谢现在写注释的自己。

---

## 2.7 本课小结

- 变量 = 贴标签的盒子；常用类型：`int`、`double`、`boolean`、`String`。
- 整数除法会丢掉小数部分；`=` 是赋值，`==` 是比较。
- `if / else if / else` 处理分支，`switch` 处理多选一。
- `for` 适合已知次数的循环，`while` 适合只看条件的循环；`break` 跳出循环，`continue` 跳过本轮。
- 分号是句号；花括号圈出代码块；`//` 开头是注释。

---

## 2.8 配套练习

> 答案见本模块 `答案/模块一答案与解析.md`。

**基础题**

1. 下面哪些变量名是合法的？哪些不合法？为什么？
   `player1`、`1player`、`player-name`、`playerName`、`class`、`max_stack_size`
2. 写出下列表达式的结果：
   - `17 / 5`
   - `17 % 5`
   - `17.0 / 5`
   - `(8 > 5) && (3 < 2)`
   - `!(10 == 10)`
3. 把这句话翻译成 if 语句："如果物品数量大于等于 64，打印'已满组'；否则打印'未满'。"（变量名自定）

**提高题**

4. 写一个 `for` 循环，打印 1 到 100 之间所有能被 3 整除的数。
5. 模拟经验升级：玩家经验从 0 开始，每次循环增加 7 点经验，每 10 点经验升 1 级（用 `exp / 10` 计算等级）。用 `while` 循环不断加经验，直到等级达到 5 级为止，每圈打印当前经验和等级。
6. 思考题：下面这段代码会打印什么？为什么？

```java
int x = 5;
if (x = 6) {       // 注意这里
    System.out.println("x 是 6");
}
```

（提示：先自己分析，再把它敲进 IDEA 看看编译器说什么。这道题揭示了一个 Java 和某些语言的重要区别。）
