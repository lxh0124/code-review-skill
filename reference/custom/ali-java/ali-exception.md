> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 异常处理

- 禁止 catch `RuntimeException`/`NullPointerException`/`IndexOutOfBoundsException` 等 JDK 运行时异常来替代前置校验（如 `if (obj != null)`）
- 禁止用异常做普通流程控制
- 禁止对大段代码 blanket try-catch；区分稳定/非稳定代码，catch 尽量具体
- 禁止吞掉异常；不处理则向上抛，最外层须处理并转为用户可理解信息
- 事务方法抛异常须确保回滚
- catch 类型须为抛出类型的同类或父类
- **禁止**在 `finally` 中 `return`（会覆盖 try/catch 返回值或吞异常）
- 禁止直接抛 `RuntimeException`/`Exception`/`Throwable`，用业务自定义异常（`DAOException`、`ServiceException`）

## 日志规范

- 禁止直接依赖 Log4j/Logback API，统一 SLF4J：`LoggerFactory.getLogger`
- 日志至少保留 15 天
- 扩展日志命名：`appName_logType_logName.log`（如 `mppserver_monitor_timeZoneConvert.log`）
- TRACE/DEBUG/INFO 须用占位符或 `isXxxEnabled()` 判断，禁止字符串拼接（无谓 `toString` 开销）
- Log4j logger 须设 `additivity=false`，避免重复输出
- 异常日志须含上下文参数 + 完整堆栈：`logger.error("ctx_{}", param, e)`
- 生产环境慎用 DEBUG；WARN 记无效参数，ERROR 记系统/逻辑错误
- 涉及敏感操作（资金、权限变更等）的日志须保留 ≥6 个月（嵩山版）

## 错误码

- 开放 API / 统一错误响应须遵循 [ali-error-code.md](ali-error-code.md)

## 分层异常

- 详见 [ali-project.md](ali-project.md)「分层异常」章节
