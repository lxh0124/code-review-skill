> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 线程与线程池

- 单例及其中方法必须线程安全
- 创建线程/线程池须指定有意义线程名
- **禁止**在应用中显式 `new Thread()`；线程资源必须由线程池提供
- **禁止**用 `Executors` 创建线程池，必须用 `ThreadPoolExecutor` 明确核心/最大线程数、队列、拒绝策略

## 日期与 ThreadLocal

- `SimpleDateFormat` 线程安全与格式化规则见 [ali-datetime.md](ali-datetime.md)
- 线程池场景 `ThreadLocal` 用完必须 `remove()`（放 `finally`），防内存泄漏与脏数据

## 锁与并发更新

- 高并发优先评估锁开销：代码块锁优于方法锁，对象锁优于类锁
- 多资源/多表加锁顺序必须一致，防死锁
- `Lock.lock()` 必须在 `try` 外调用，`unlock()` 在 `finally`；`try` 与 `lock()` 之间不得有其他代码
- 并发更新同一记录须加锁：应用层锁 / 缓存锁 / DB 乐观锁（version）；冲突率 <20% 倾向乐观锁，重试 ≥3 次

## 定时与随机数

- 多 `TimerTask` 用 `ScheduledExecutorService`，禁止 `Timer`（未捕获异常会终止所有任务）
- 多线程共享 `Random` 影响性能，JDK7+ 用 `ThreadLocalRandom`

## 推荐（重要）

- `CountDownLatch`：子线程 `finally` 中必须 `countDown`，主线程须处理子线程异常
- 双重检查锁单例须将目标对象声明为 `volatile`
- `volatile` 解决可见性，不保证 `count++` 原子性；计数用 `AtomicInteger`/`LongAdder`
- 高并发下注意 `HashMap` 扩容死链风险，必要时用 `ConcurrentHashMap`
