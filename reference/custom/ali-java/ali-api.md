> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · 嵩山版+ · 违反【强制】项标记 blocking

## API 定义

- 须明确：协议、域名、路径、方法、请求体、状态码、响应体
- 生产环境 **HTTPS**
- REST 路径：名词、推荐复数、小写+下划线；禁止 `.json`/`.xml` 后缀；动词由 HTTP 方法表达
- URL 参数无敏感信息；body 请求须设 `Content-Type`

## 响应约定

- 列表为空返回 `[]` 或 `{}`，禁止 `null`
- 错误响应含：HTTP 状态码 + `errorCode` + `errorMessage` + 用户友好提示
- JSON key 统一 **lowerCamelCase**（如 `errorCode`、`orderList`），禁止 `ERROR_CODE`/`msg` 等
- `errorMessage` 供前后端追踪，可放 hidden 字段或客户端日志
- 超大整数（订单号 ≥16 位等）**用 String 返回**，禁止 Long（JS Number 精度丢失）

## 请求限制

- URL 参数总长 ≤ 2048 字节
- body 须控制长度（注意 nginx/tomcat 默认限制）；翻页：参数 <1 返第一页，>总页数返最后一页

## 推荐

- 返回 JSON 而非 XML；响应头标明缓存策略（`Cache-Control`）
- 时间字段统一 `yyyy-MM-dd HH:mm:ss` GMT
- 版本号放 HTTP Header，不放 URL 路径（参考）

## 重定向

- 内部 `forward`；外部 URL 用统一代理/拼装工具（HTTPS 环境）
