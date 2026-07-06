> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 建表规约

- 表达布尔含义字段命名 `is_xxx`，类型 `unsigned tinyint`（1 是 / 0 否）；非负字段用 `unsigned`
- 表名/字段名小写字母、数字、下划线；禁止以数字开头、禁止连续下划线间仅数字
- 表名不用复数；禁用 MySQL 保留字（desc、range、match 等）
- 主键索引 `pk_`、唯一索引 `uk_`、普通索引 `idx_` 前缀
- 小数用 `decimal`，禁止 `float`/`double` 存金额
- 定长字符串用 `char`；`varchar` 长度不超过 5000，超长用 `text` 并考虑拆表
- 表必须含 `id`（`bigint unsigned` 自增）、`gmt_create`、`gmt_modified`

## 索引规约

- 业务上需要唯一的字段必须建唯一索引（应用层校验不能替代）
- 超过三表 `JOIN` 禁止；关联字段类型一致且须建索引
- `varchar` 索引须指定长度（按数据区分度）
- 分页禁止 `LIKE '%xxx'` 或 `LIKE '%xxx%'`（左模糊无法走索引）

## SQL 语句

- 统计行数用 `COUNT(*)`，不用 `COUNT(列名)` 代替
- `COUNT(distinct col1, col2)` 任一列全 null 则结果为 0
- `SUM(列)` 全 null 时返回 null，注意 NPE；判空用 `ISNULL()`
- NULL 比较用 `ISNULL()`，禁止 `= null` / `<> null`
- 分页前 count 为 0 须立即返回，禁止继续执行分页 SQL
- 禁止外键与级联更新（逻辑在应用层）
- 禁止存储过程
- 数据订正（delete/update）前必须先 `SELECT` 确认
- `IN` 集合元素控制在 1000 以内，能避免则避免
- 逻辑删除字段统一 `is_deleted`（嵩山版）
- 多表查询须为字段指定表别名，禁止 `select *` 且无别名（泰山版+）
- 禁止 `TRUNCATE` 做数据清理（无事务、不触发 trigger，事故风险）

## ORM 映射

- 查询禁止 `SELECT *`，须指定列名
- POJO 布尔属性不加 `is`，DB 列加 `is_`，`resultMap` 须显式映射
- 禁止 `resultClass` 作返回类型，须用 DO + `resultMap`
- MyBatis 参数用 `#{}`，**禁止** `${}`（SQL 注入）
- 禁止 `queryForList(statement, start, size)` 式内存分页（易 OOM），SQL 层 `LIMIT`
- 禁止用 `HashMap`/`Hashtable` 接收查询结果
- 更新记录时同步更新 `gmt_modified`
- 禁止「大而全」POJO 全字段 update；只更新变更列（推荐）
- `@Transactional` 勿滥用，须考虑缓存/搜索/消息回滚（参考）
