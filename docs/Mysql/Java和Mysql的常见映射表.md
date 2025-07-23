### 📦 三、常见映射对照表（实用）

| MySQL 类型            | Java 类型（推荐）                                    | 字节数/说明                    |
| --------------------- | ---------------------------------------------------- | ------------------------------ |
| `TINYINT`             | `Byte` / `Integer`                                   | 1字节，-128~~127 或 0~~255     |
| `TINYINT(1)`          | `Boolean`                                            | ⚠️ 框架默认解析为布尔值         |
| `SMALLINT`            | `Short` / `Integer`                                  | 2字节                          |
| `INT` / `INTEGER`     | `Integer`                                            | 4字节                          |
| `BIGINT`              | `Long`                                               | 8字节                          |
| `DECIMAL` / `NUMERIC` | `BigDecimal`                                         | 精度高，不能用 float 替代      |
| `FLOAT`               | `Float`                                              | 4字节，可能精度丢失            |
| `DOUBLE`              | `Double`                                             | 8字节                          |
| `CHAR` / `VARCHAR`    | `String`                                             | 字符串                         |
| `TEXT` / `LONGTEXT`   | `String`                                             | 长文本                         |
| `DATE` / `DATETIME`   | `java.time.LocalDateTime`（推荐）或 `java.util.Date` | 时间字段建议用 `LocalDateTime` |