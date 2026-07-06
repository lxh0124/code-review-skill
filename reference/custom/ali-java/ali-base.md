> 来源：[阿里巴巴 Java 开发手册 (P3C)](https://alibaba.github.io/p3c/) · Tier2 精简规范 · 违反【强制】项标记 blocking

## 命名风格

- 命名禁止以 `_`、`$` 开头或结尾；禁止拼音/中英混用/中文命名
- 类名 `UpperCamelCase`（例外：DO/BO/DTO/VO/AO）；方法/参数/成员/局部变量 `lowerCamelCase`
- 常量全大写下划线分隔；抽象类 `Abstract`/`Base` 开头，异常类 `Exception` 结尾，测试类 `*Test` 结尾
- 数组定义 `String[] args`，禁止 `String args[]`
- POJO 布尔属性禁止 `is` 前缀（避免 RPC/序列化解析错误）
- 包名全小写单数；禁止随意缩写（如 `AbsClass`、`condi`）
- Service/DAO 必须接口 + `Impl` 实现类

## 常量定义

- 禁止魔法值直接出现在代码中
- `long`/`Long` 字面量用大写 `L`，禁止小写 `l`

## 代码格式

- 大括号、空格、缩进（4 空格禁 tab）、单行 ≤120 字符按 P3C 换行规则
- 方法多参数逗号后必须空格
- 文件编码 UTF-8，换行符 Unix（LF）

## OOP 规约

- 静态成员用类名访问，禁止通过实例引用
- 覆写方法必须 `@Override`
- 可变参数放最后，禁止 `Object...`；外部依赖接口禁止改签名，废弃须 `@Deprecated`
- 禁止使用过时的类/方法
- `equals` 用常量或确定非 null 对象调用：`"x".equals(obj)`；包装类比较统一 `equals`
- 浮点比较禁止 `==`/`equals`，用误差范围或 `BigDecimal`；`BigDecimal` 等值用 `compareTo == 0`，不用 `equals`（嵩山版）
- POJO/RPC 参数返回值用包装类型；POJO 属性禁止设默认值
- 序列化类增属性不改 `serialVersionUID`（不兼容升级除外）
- 构造方法禁止业务逻辑；POJO 必须实现 `toString`（继承时先 `super.toString()`）

## 控制语句

- `switch` 每个 case 必须 `break`/`return`，必须有 `default`
- `if`/`else`/`for`/`do`/`while` 即使单行也必须加大括号
- 嵌套 `if-else` 不超过 3 层；条件表达式避免复杂逻辑直接内联

## 注释规约

- 类/类变量/方法用 Javadoc `/** */`，抽象/接口方法须写参数、返回值、异常
- 每个类注明作者与日期；枚举字段须 Javadoc 注释
- 单行注释放被注释代码上方；TODO/FIXME 须含作者与时间并及时清理

## 其他（基础）

- 正则 `Pattern` 须预编译，禁止在方法体内 `Pattern.compile`
- 获取毫秒用 `System.currentTimeMillis()`，禁止 `new Date().getTime()`（日期详见 [ali-datetime.md](ali-datetime.md)）
- 禁止 Apache `BeanUtils` 做属性 copy（性能差且浅拷贝）；可用 Spring BeanUtils / Cglib BeanCopier，注意浅拷贝风险
- 循环内字符串拼接用 `StringBuilder.append`，禁止 `+` 连接
- `Object.clone()` 默认浅拷贝，谨慎使用
