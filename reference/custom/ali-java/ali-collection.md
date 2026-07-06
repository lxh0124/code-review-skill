> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 集合基础

- 覆写 `equals` 必须覆写 `hashCode`；`Set` 元素、`Map` 的 key 同理
- 判断集合空用 `isEmpty()`，禁止 `size() == 0`
- `keySet()`/`values()`/`entrySet()` 返回集合禁止 `add`/`remove`
- `Collections.emptyList()`/`singletonList()` 等不可变集合禁止增删

## 转换与视图

- 禁止将 `ArrayList.subList()` 强转为 `ArrayList`（`RandomAccessSubList` 是视图）
- 修改原列表大小会导致 `subList` 遍历/增删抛 `ConcurrentModificationException`
- 集合转数组用 `toArray(T[] array)`，数组类型与大小一致；禁止无参 `toArray()` 再强转
- `Arrays.asList()` 后禁止 `add`/`remove`/`clear`；注意修改原数组会反映到 list

## 泛型与遍历

- 禁止 `List<?>` 上调用 `add`，禁止 `List<? super T>` 上调用 `get`（PECS 原则）
- **禁止**在 `foreach` 中 `remove`/`add`，用 `Iterator.remove()`；并发遍历须对 `Iterator` 加锁

## Comparator

- JDK7+ 自定义 `Comparator` 须满足：反对称、传递性、相等一致性，否则 `sort` 抛 `IllegalArgumentException`
- 相等元素比较不能简单返回 `-1`（须处理 `compare(a,a)==0`）

## 推荐（重要）

- 集合初始化指定容量（如 `new ArrayList<>(initialCapacity)`）
- 遍历 `Map` 用 `entrySet` 或 JDK8 `forEach`，避免 `keySet` 二次查找
- 注意各 Map 是否允许 null key/value：`ConcurrentHashMap`/`Hashtable` 均不允许 null

## Stream / Collectors（嵩山版+）

- `Collectors.toMap()` 必须提供 `mergeFunction`，否则 duplicate key 抛 `IllegalStateException`
- `toMap()` 的 value 为 null 会 NPE（`HashMap.merge` 限制）
