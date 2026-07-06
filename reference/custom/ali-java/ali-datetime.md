> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · 嵩山版+ · 违反【强制】项标记 blocking

## 格式化

- 年份用小写 `y`（`yyyy`），禁止大写 `YYYY`（周-based year，跨年周会错年）
- 分清 `M`(月) / `m`(分)、`H`(24h) / `h`(12h)
- 推荐格式：`yyyy-MM-dd HH:mm:ss`；JDK8+ 统计场景用 `Instant`/`LocalDateTime`/`DateTimeFormatter`

## 类型与 API

- 获取毫秒：`System.currentTimeMillis()`，禁止 `new Date().getTime()`
- **禁止**使用 `java.sql.Date` / `Time` / `Timestamp` 参与业务比较（与 `java.util.Date` 混用有 JDK 历史 BUG）
- `SimpleDateFormat` 禁止 static 共享；线程池场景配合 `ThreadLocal` 或换 `DateTimeFormatter`

## 闰年与历法

- 禁止写死一年 365 天（会员有效期、数组长度等用 `LocalDate.lengthOfYear()` 或 API 计算）
- 注意闰年 2 月 29 日：一年后不可能是 2/29
- 月份优先用枚举；`Calendar.MONTH` 为 0–11

## 并发（交叉）

日期格式化线程安全要求见 [ali-concurrent.md](ali-concurrent.md) 中 `SimpleDateFormat` / `ThreadLocal.remove()` 条目。
