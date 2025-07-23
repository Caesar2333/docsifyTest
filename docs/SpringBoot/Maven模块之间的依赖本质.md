# Maven依赖的本质

Maven模块之间相互依赖时，实际“引入”的是依赖模块打包后的所有class文件（包括bean、工具类等），而不是它的依赖列表。

------

## 详细解释

### 1. Maven依赖的本质

- 当A模块依赖B模块时，A会把B模块编译后的所有class文件（包括bean、普通类、配置类等）都加入到自己的classpath中。

- 但是，B模块pom.xml里声明的依赖（比如Swagger），不会自动传递到A模块，除非A模块也在自己的pom.xml里声明这些依赖。

### 2. 依赖传递性

- 直接依赖：A依赖B，A能用B的所有class。

- 依赖的依赖（传递依赖）：如果B依赖了C，A默认也能用C的class（除非B用<scope>provided</scope>等特殊声明）。

- 三方依赖（如Swagger）：如果B依赖了Swagger，但A没依赖Swagger，A启动时不会自动把Swagger的jar包加进来，除非A也依赖Swagger。

### 3. Spring Boot Bean加载

- 只要class在classpath里，Spring Boot就能扫描到（比如你的Controller、Service等）。

- 但如果缺少Swagger依赖，相关的注解、配置、UI页面等就不会生效。

------

## 结论

- 模块依赖引入的是class文件（代码），不是依赖声明本身。

- 三方依赖（如Swagger）必须在启动模块的pom.xml里声明，否则不会被打包进最终的应用。

- Bean只要class在classpath里就能被Spring扫描，但三方依赖必须在启动模块声明。