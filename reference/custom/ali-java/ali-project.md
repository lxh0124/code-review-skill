> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 应用分层

- 默认上层依赖下层：开放接口 → Web → Service → Manager → DAO
- **DO**：表结构对象，DAO 向上传输
- **DTO**：Service/Manager 向外传输
- **BO**：封装业务逻辑
- **Query**：查询对象；**超过 2 个查询条件禁止用 Map 传参**
- **VO**：展示层对象，Web 向模板传输
- Service/DAO 须接口化，实现类 `*Impl` 后缀

## 分层异常（参考）

- DAO：`catch (Exception e)` → `throw new DAOException(e)`，不重复打日志
- Service/Manager：异常须落盘日志，尽量带参数
- Web 层禁止继续上抛；影响渲染则跳转友好错误页
- 开放接口：异常转为错误码 + 错误信息返回

## 二方库 / Maven（强制）

- GAV：`com.{公司/BU}.业务线[.子业务线]`（≤4 级）；ArtifactId `产品线-模块`；版本 `主.次.修订`，初始须 `1.0.0`
- 线上禁止依赖 `SNAPSHOT`（安全包除外）；RELEASE 版本不可覆盖
- 升级依赖须评估：`dependency:resolve` / `dependency:tree` 比对，不必要变更保持仲裁结果不变
- 二方库接口返回值**禁止**使用枚举或含枚举的 POJO；参数可用枚举
- 同组依赖用统一版本变量（如 `${spring.version}`）
- 子项目禁止相同 GAV 不同 Version 并存

## Maven 推荐

- 版本在父 POM `dependencyManagement` 声明，子模块 `dependencies` 引用
- 二方库尽量不自带配置项

## 服务器（运维向，推荐）

- 高并发可调小 `tcp_fin_timeout`（如 30s）
- 调大最大文件句柄 fd，防 `open too many files`
- JVM 开启 `-XX:+HeapDumpOnOutOfMemoryError`
- 内部重定向 `forward`；外部 URL 用统一工具生成
