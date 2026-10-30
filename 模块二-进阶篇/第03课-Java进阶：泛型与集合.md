# 第 03 课：Java 进阶（一）——泛型与集合框架

> 本课目标：看懂 `List<String>`、`Map<K, V>` 这类带尖括号的写法，掌握 ArrayList、HashMap 的使用，认识 Lambda 和 Stream。
>
> 前置知识：模块一第 03、04 课。

---

## 3.1 泛型（Generic）：给容器贴上"内容物标签"

### 痛点引入

数组能装成组数据，但**长度固定**。游戏里的背包物品数量会变，用数组很不方便。Java 提供了"会变长的数组"——`ArrayList`：

```java
ArrayList list = new ArrayList();   // 老写法：什么都能往里装
list.add("钻石剑");
list.add(123);                      // 混进了一个数字，编译器不拦你！
String s = (String) list.get(1);    // 运行时爆炸：数字不能当字符串用
```

问题：编译器不知道这个列表"应该装什么"，错误要到运行时才暴露。

**泛型**就是给容器贴标签："这个列表**只装 String**"：

```java
ArrayList<String> list = new ArrayList<>();   // <String> 就是泛型参数
list.add("钻石剑");
list.add(123);                // ❌ 编译期直接报错！错误被提前暴露
String s = list.get(0);       // ✅ 不用强制转换，拿出来的就是 String
```

> 💡 **类比**：泛型像快递箱上贴的"内含物品：易碎品"标签。贴了之后，分拣员（编译器）会帮你检查，谁也不能往"餐具箱"里塞哑铃。

你早就在用泛型了：`DeferredRegister<Item>`、`RegistryObject<Block>`——尖括号就是"这个注册器/注册对象，管理的是 Item/Block 类型"。

### 泛型写法规则

- 尖括号里只能写**类类型**，不能写基本类型：`<Integer>` 对，`<int>` 错。每个基本类型有对应的"包装类"：`int→Integer`、`double→Double`、`boolean→Boolean`。
- 创建时右侧可以写空的 `<>`（菱形语法），编译器自动推断。

---

## 3.2 集合框架：Java 的"容器家族"

### List：有序的列表（可重复）

```java
List<String> backpack = new ArrayList<>();

backpack.add("钻石剑");        // 末尾添加
backpack.add("面包");
backpack.add(0, "火把");       // 插到下标 0 处
backpack.get(1);              // 取下标 1 的元素 → "钻石剑"
backpack.remove("面包");       // 删除指定元素
backpack.size();              // 元素个数（相当于数组的 length）
backpack.contains("钻石剑");   // 是否包含 → true

for (String item : backpack) {   // 遍历，和数组一样用增强 for
    System.out.println(item);
}
```

`List` 是接口，`ArrayList` 是它的常用实现类（第 04 课接口的思想：`List<String> x = new ArrayList<>()`，用接口类型接收，以后想换实现类只改右边）。

### Map：键值对字典（按"键"查"值"）

最适合表达"根据 A 查 B"：根据玩家名查等级、根据物品注册名查物品……

```java
Map<String, Integer> playerLevels = new HashMap<>();   // <键类型, 值类型>

playerLevels.put("小明", 30);       // 存入：键"小明" → 值 30
playerLevels.put("小红", 12);
playerLevels.get("小明");          // 查 → 30
playerLevels.get("不存在的人");     // 查不到 → null（注意！）
playerLevels.containsKey("小红");  // 是否含某键 → true
playerLevels.remove("小明");       // 删除

// 遍历 Map 的标准写法：
for (Map.Entry<String, Integer> entry : playerLevels.entrySet()) {
    System.out.println(entry.getKey() + " 的等级是 " + entry.getValue());
}
```

> ⚠️ `get` 查不到键时返回 `null`（空值）。对 `null` 调用方法会抛出臭名昭著的 `NullPointerException`（空指针异常，NPE）。安全写法：
>
> ```java
> Integer level = playerLevels.get("小明");
> if (level != null) {
>     System.out.println(level);
> }
> ```

### Set：不重复的集合（了解）

```java
Set<String> modAuthors = new HashSet<>();
modAuthors.add("小明");
modAuthors.add("小明");   // 重复添加无效，Set 里还是只有一个
modAuthors.size();        // 1
```

### 怎么选？

| 需求 | 选择 |
|---|---|
| 按顺序存一串东西，可能要重复 | `List` |
| 按键查值，一对一映射 | `Map` |
| 只关心"有没有"，不许重复 | `Set` |

---

## 3.3 Lambda 表达式：把方法当参数传

第 06 课注册物品时见过 `() -> new Item(...)`，现在正式认识它。

**Lambda 是"匿名方法"的简写**——不需要起名字的临时方法：

```java
// 完整写法（匿名内部类，很啰嗦）
ITEMS.register("ruby", new Supplier<Item>() {
    @Override
    public Item get() {
        return new Item(new Item.Properties());
    }
});

// Lambda 简写：参数 → 方法体
ITEMS.register("ruby", () -> new Item(new Item.Properties()));
```

三种形态：

```java
() -> 表达式                    // 无参数
x -> 表达式                     // 一个参数（括号可省）
(x, y) -> { 多条语句; return 结果; }   // 多参数/多语句要用花括号和 return
```

再比如给列表排序：

```java
List<String> tools = new ArrayList<>(List.of("钻石剑", "木镐", "弓"));
tools.sort((a, b) -> a.length() - b.length());   // 按名字长度排序
System.out.println(tools);   // [弓, 木镐, 钻石剑]
```

---

## 3.4 Stream：流水线式处理集合（进阶了解）

Stream 让你用"流水线"的方式处理集合：过滤 → 变换 → 收集。

```java
List<String> backpack = List.of("钻石", "石头", "钻石剑", "木板", "钻石靴");

// 需求：找出所有名字含"钻石"的物品，转成大写，收进新列表
List<String> result = backpack.stream()           // 打开流水线
        .filter(item -> item.contains("钻石"))     // 第一道工序：过滤
        .map(item -> item.toUpperCase())          // 第二道工序：变换
        .toList();                                // 收集成品

System.out.println(result);   // [钻石, 钻石剑, 钻石靴]（英文才看得出大写效果，此处演示流程）
```

常用工序：

- `filter(条件)`：只留下满足条件的；
- `map(变换)`：把每个元素变成另一个东西；
- `count()`：数个数；`forEach(动作)`：对每个元素执行动作。

> 💡 不要求立刻熟练。Forge 代码和数据生成里经常看到 Stream，能看懂即可，写着写着就熟了。

---

## 3.5 本课小结

- 泛型 `<类型>` 给容器贴内容物标签，把类型错误从运行时提前到编译期。
- 三大集合：`List`（有序列表）、`Map`（键值字典）、`Set`（不重复集合）；Map 查不到键返回 `null`，要防空指针。
- Lambda `() -> ...` 是匿名方法简写，让"把行为当参数传"变得简洁。
- Stream 流水线：`filter` 过滤、`map` 变换、`toList/forEach` 收尾。

---

## 3.6 配套练习

> 答案见本模块 `答案/模块二答案与解析.md`。

**基础题**

1. 为什么 `List<int>` 这种写法不合法？应该怎么写？
2. `Map<String, Integer> map = new HashMap<>();` 执行 `map.get("不存在的键")` 返回什么？直接使用返回值有什么风险？
3. 要给"一组不可重复的玩家名字"选集合类型，应该选 List、Map 还是 Set？

**提高题**

4. 写一个方法，统计一组掉落物中每种物品的数量：

```java
// 输入: ["钻石", "木头", "钻石", "石头", "木头", "钻石"]
// 期望输出: {钻石=3, 木头=2, 石头=1}
static Map<String, Integer> countDrops(List<String> drops) { ... }
```

5. 用 Lambda 把一个 `List<Integer>`（玩家等级列表）按从高到低排序。
6. 用 Stream 完成：给定玩家等级列表 `List.of(5, 12, 30, 8, 25)`，筛出 10 级以上的玩家，计算共有几人。
7. 思考题：第 06 课的 `registerBlock` 方法签名是 `<T extends Block> RegistryObject<T> registerBlock(String name, Supplier<T> supplier)`。结合本课泛型知识解释：`<T extends Block>` 声明了什么？这样设计相比写死 `Block` 有什么好处？
