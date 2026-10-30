# 第 05 课：Forge 事件系统——让模组"响应"游戏

> 本课目标：彻底掌握 Forge 事件总线，学会订阅游戏事件，做出"玩家受伤时…""方块被挖时…"这类真正的模组功能。
>
> 前置知识：模块一第 05 课（事件初识）、本模块第 03 课（Lambda）。

---

## 5.1 两条总线：先分清再上车

Forge 有**两条**事件总线，订阅错总线是新手高频 bug：

| 总线 | 广播什么 | 什么时候的事 | 获取方式 |
|---|---|---|---|
| **模组总线**（Mod Bus） | 生命周期事件：注册、配置、数据生成、创造栏填充…… | 游戏**启动加载**阶段 | `FMLJavaModLoadingContext.get().getModEventBus()` |
| **Forge 总线**（Forge Bus） | 游戏运行时事件：玩家登录、生物受伤、方块破坏…… | 游戏**运行中** | `MinecraftForge.EVENT_BUS` |

一句话记忆：**加载期找模组总线，游戏里的事找 Forge 总线**。

---

## 5.2 订阅事件的两种方式

### 方式一：addListener（手动订阅，适合单个事件）

第 05 课已经用过：

```java
IEventBus forgeBus = MinecraftForge.EVENT_BUS;
forgeBus.addListener(this::onPlayerLogin);

private void onPlayerLogin(PlayerEvent.PlayerLoggedInEvent event) {
    // 玩家登录时执行
}
```

### 方式二：@SubscribeEvent + 自动注册（推荐，适合事件多的类）

```java
import net.minecraftforge.event.entity.living.LivingHurtEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

// ① 类上加注解：自动把本类所有 @SubscribeEvent 方法挂到 Forge 总线
@Mod.EventBusSubscriber(modid = MyCoolMod.MODID)
public class GameEvents {

    // ② 方法上加注解：生物受伤时触发
    @SubscribeEvent
    public static void onLivingHurt(LivingHurtEvent event) {
        System.out.println(event.getEntity().getName().getString()
                + " 受到了 " + event.getAmount() + " 点伤害");
    }
}
```

注意：用 `@Mod.EventBusSubscriber` 自动注册时，事件处理方法必须是 **`public static`**。

> 💡 **默认挂哪条总线？** `@Mod.EventBusSubscriber` 默认挂 **Forge 总线**。要挂模组总线需写明 `bus = Mod.EventBusSubscriber.Bus.MOD`。

---

## 5.3 事件对象：信息的包裹

每个事件方法收到的参数（如 `LivingHurtEvent event`）是一个**事件对象**，里面装着事发时的信息，常用操作有三种：

1. **读取信息**：`event.getEntity()`（受害者）、`event.getAmount()`（伤害量）；
2. **修改结果**：`event.setAmount(0)`（把伤害改成 0 = 免伤）；
3. **取消事件**：有些事件支持 `event.setCanceled(true)`，直接阻止事情发生。

不是所有事件都可取消/可修改——看事件类里有没有 `setCanceled`、`setXxx` 方法即可（IDEA 里按住 Ctrl 点进事件类查看）。

---

## 5.4 常用游戏事件速查（Forge 总线）

| 事件类 | 触发时机 | 典型玩法 |
|---|---|---|
| `PlayerEvent.PlayerLoggedInEvent` | 玩家进入游戏 | 发欢迎消息、送新手礼包 |
| `LivingHurtEvent` | 生物受伤前 | 免伤、伤害加成 |
| `LivingDeathEvent` | 生物死亡时 | 死亡不掉落、击杀奖励 |
| `BlockEvent.BreakEvent` | 玩家破坏方块时 | 挖方块给经验、禁止挖掘 |
| `ItemTooltipEvent` | 显示物品悬浮提示时 | 给物品加自定义描述 |
| `PlayerInteractEvent.RightClickBlock` | 右键方块时 | 自定义交互 |
| `TickEvent.PlayerTickEvent` | 每个游戏刻（1秒=20刻） | 持续效果检测 |

### 实战示例 1：欢迎消息

```java
@Mod.EventBusSubscriber(modid = MyCoolMod.MODID)
public class WelcomeEvents {

    @SubscribeEvent
    public static void onLogin(PlayerEvent.PlayerLoggedInEvent event) {
        // 给玩家发一条聊天消息
        event.getEntity().sendSystemMessage(
                Component.literal("欢迎来到服务器，勇者！")
        );
    }
}
```

### 实战示例 2：穿红宝石盔甲免摔伤

```java
@SubscribeEvent
public static void onHurt(LivingHurtEvent event) {
    // 判断伤害来源是不是摔落
    if (event.getSource().equals(event.getEntity().damageSources().fall())) {
        event.setAmount(0);   // 摔落伤害清零
    }
}
```

### 实战示例 3：挖方块爆经验

```java
@SubscribeEvent
public static void onBreak(BlockEvent.BreakEvent event) {
    // event.getState() 是被挖方块的状态，可以判断方块种类
    if (event.getState().is(Blocks.DIAMOND_ORE)) {
        Player player = event.getPlayer();
        player.giveExperiencePoints(20);   // 额外奖励 20 点经验
    }
}
```

---

## 5.5 事件优先级与取消（了解）

多个模组可能监听同一事件，谁先谁后？`@SubscribeEvent(priority = EventPriority.HIGH)` 可以调整优先级（HIGHEST → LOWEST）。数值上先执行的有机会先改结果。普通模组一般用默认（NORMAL）即可。

如果一个事件被前面的模组 `setCanceled(true)` 了，后面默认不再收到——除非加 `receiveCanceled = true`。

---

## 5.6 排错指南

事件没反应？按顺序查：

1. 订阅到**正确的总线**了吗？（运行时事件用模组总线 = 永远不触发）
2. 用 `@Mod.EventBusSubscriber` 时方法是 `public static` 吗？
3. 方法的 `modid` 写对了吗？
4. 事件真的发生了吗？（加一行 `System.out.println` 验证）
5. 是否在正确的"端"？（如渲染相关事件只在客户端触发）

---

## 5.7 本课小结

- 两条总线：加载期事件走模组总线，运行时事件走 Forge 总线（`MinecraftForge.EVENT_BUS`）。
- 订阅方式：`addListener` 手动挂单个，或 `@Mod.EventBusSubscriber` + `@SubscribeEvent`（方法需 `public static`）批量挂。
- 事件对象可读取信息、修改结果、取消事件（视事件支持情况而定）。
- 记不住事件名没关系，记住"去 Forge 源码 event 包里翻"这个习惯。

---

## 5.8 配套练习

> 答案见本模块 `答案/模块二答案与解析.md`。

**基础题**

1. `BuildCreativeModeTabContentsEvent` 和 `LivingHurtEvent` 分别属于哪条总线？
2. 用 `@Mod.EventBusSubscriber` 自动注册时，事件处理方法必须满足什么签名要求？
3. 想让一个事件"完全不发生"（比如阻止方块被破坏），应该调用事件对象的什么方法？

**提高题**

4. 写一个事件处理：玩家死亡时在聊天栏发送一条消息 `"玩家 xxx 倒下了！"`（提示：`LivingDeathEvent`，玩家是 `Player` 类型，用 `instanceof` 判断）。
5. 写一个事件处理：给模组一的红宝石物品添加悬浮提示文字 `"一块闪耀着幸运光芒的宝石"`（提示：`ItemTooltipEvent`，用 `event.getItemStack().is(ModItems.RUBY.get())` 判断，`event.getToolTip().add(...)` 添加）。
6. 综合题：做一个简单的"砍树奖励"功能——玩家破坏任何原木方块（`BlockTags.LOGS` 标签）时，额外掉落一颗你的红宝石（提示：在 `BlockEvent.BreakEvent` 中用 `event.getLevel()` 拿到世界、`Block.popResource(...)` 或直接 `player.drop(...)` 生成掉落物；可以用 `state.is(BlockTags.LOGS)` 判断）。
7. 思考题：`TickEvent.PlayerTickEvent` 每秒触发 20 次。如果在这个事件里写很耗时的操作，会对游戏造成什么影响？这给你设计事件逻辑什么启示？
