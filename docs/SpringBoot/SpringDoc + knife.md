# api文档的现状

### 🔧 4. 它们和 Swagger 有啥关系？

| 工具      | 协议规范             | 是否维护中 | 推荐使用 |
| --------- | -------------------- | ---------- | -------- |
| Swagger2  | OpenAPI 2.0          | ❌ 已废弃   | ❌        |
| Springdoc | OpenAPI 3.0          | ✅ 活跃维护 | ✅        |
| Knife4j   | Springdoc 的界面增强 | ✅ 活跃维护 | ✅        |

⚠️ 别再用 Swagger2 了，Spring Boot 3.x 根本不兼容，要么 Springdoc，要么 Springdoc + Knife4j



# 注解应该写在哪里？？

### 🎯 1. 注解写在哪？

最常写注解的位置如下：

| 位置                | 注解类型                      | 作用                                   |
| ------------------- | ----------------------------- | -------------------------------------- |
| Controller 类上     | `@Tag`、`@Api`                | 描述这个 Controller 的用途             |
| Controller 方法上   | `@Operation`、`@ApiOperation` | 描述这个方法做什么，接口说明           |
| 参数上              | `@Parameter`、`@RequestBody`  | 对参数进行说明，如说明字段、是否必传等 |
| 实体类（DTO）字段上 | `@Schema`                     | 给字段添加说明（字段名、是否必填等）   |

### ✅ 2. 注解用法示例（推荐 Springdoc 版本）

```java
@RestController
@RequestMapping("/user")
@Tag(name = "用户模块", description = "用户登录、注册等接口")
public class UserController {

    @PostMapping("/login")
    @Operation(summary = "用户登录", description = "根据用户名和密码进行登录校验")
    public Result<UserVO> login(
        @Parameter(description = "用户名", required = true) @RequestParam String username,
        @Parameter(description = "密码", required = true) @RequestParam String password
    ) {
        // ...
    }

    @PostMapping("/add")
    @Operation(summary = "添加用户")
    public Result<Void> addUser(@RequestBody @io.swagger.v3.oas.annotations.parameters.RequestBody(
        description = "用户信息", required = true
    ) UserDTO userDTO) {
        // ...
    }
}
```

------

### 💡 3. DTO 上的字段说明

如果你前端想知道参数结构中的每个字段什么意思，就要这样写👇：

```java
@Data
@Schema(description = "用户信息传输对象")
public class UserDTO {

    @Schema(description = "用户名", example = "zhengshao")
    private String username;

    @Schema(description = "密码", example = "123456")
    private String password;

    @Schema(description = "头像地址", example = "http://xxx.com/avatar.png")
    private String avatarUrl;
}
```



### 📌 推荐注解清单（记住这个组合就不会错）

| 用法位置       | 注解（Springdoc）                                     |
| -------------- | ----------------------------------------------------- |
| 类上描述控制器 | `@Tag(name="xxx", description="xxx")`                 |
| 方法上描述接口 | `@Operation(summary="xxx", description="xxx")`        |
| 请求参数       | `@Parameter(description="xxx", required=true)`        |
| 请求体         | `@RequestBody` + `@io.swagger.v3...@RequestBody(...)` |
| 返回字段       | `@Schema(description="xxx", example="xxx")`           |



# 需要引入哪些依赖呢？

#### ✅ 方式二：用 Springdoc + Knife4j（界面更好，功能更强）

```html
<!-- springdoc 核心 -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-api</artifactId>
    <version>2.5.0</version>
</dependency>

<!-- Knife4j 展示界面 -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.5.0</version>
</dependency>
```

Knife4j 默认访问路径是：

```
http://localhost:8080/doc.html
```

### 🔧 2. 配置文件 application.yaml 建议配置：

```yaml
springdoc:
  api-docs:
    enabled: true
    path: /v3/api-docs
  group-configs:
    - group: default
      paths-to-match: /**
      packages-to-scan: com.caesar # 扫描的controller包的名字
  
knife4j:
  enable: true
  setting:
    language: zh-CN
```

* #### 写成这样之后，直接访问`http://localhost:8080/doc.html`就可以了。

* #### 默认是访问`doc.html`的。

### ✅ 最后给你一个记忆口诀，巩固核心认知

```ymal
写接口要三件套：
@Tag → 模块分组
@Operation → 方法说明
@Schema/@Parameter → 字段说明
```

把这三步写好，你的接口文档就自动清晰、结构分明、可点击测试，任何人都能无痛对接前后端。















