> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 流与连接（可关闭资源）

- `InputStream`/`OutputStream`/`Reader`/`Writer`、DB 连接、Socket 等 **必须在 `finally` 关闭**，或 JDK7+ 用 try-with-resources
- **`finally` 块中禁止抛异常**（会覆盖 try 中原始异常）
- 禁止在 `finally` 中 `return`

## 文件与编码

- 源码/配置文件编码 **UTF-8**，换行符 **Unix (LF)**，禁止 Windows CRLF
- 字符集存储用 UTF-8；需 emoji 用 `utf8mb4`；注意 `LENGTH` vs `CHARACTER_LENGTH` 计数差异

## 日志文件 IO

- 业务日志与错误日志尽量分文件存储
- 扩展监控/访问日志按 `appName_logType_logName.log` 命名，便于检索与清理
- 避免大量无效日志写盘；WARN 级业务日志注意体积，防磁盘打满

## 其他 IO 相关

- 循环外声明对象/获取 DB 连接/大段 try-catch（控制语句推荐）
- 数据结构初始化尽量指定初始容量，防无界增长

## 服务器 / 重定向

- fd 上限、TCP `time_wait`、JVM OOM dump、forward/URL 拼装见 [ali-project.md](ali-project.md)
- REST API 重定向与 URL 规范见 [ali-api.md](ali-api.md)
