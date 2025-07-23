# 怎么查看redis是否链接

* ### ✅ 一、判断 Redis 是否已经启动

  #### 方法1：**查看后台进程**

  打开命令提示符（`cmd`）或 PowerShell，输入：

  ```
  tasklist | findstr redis
  ```

  如果有看到 `redis-server.exe` 或类似名字，说明 Redis 正在运行。

  ```
  redis-server.exe              9896 Services                   0      9,636 K
  ```

  * #### 表示其进程号是`9896`的

  ------

  #### 方法2：**尝试连接 Redis**

  使用 redis-cli 客户端连接看看：

  ```
  redis-cli -h 127.0.0.1 -p 6379
  ```

  如果连接成功会进入交互式界面：

  ```
  127.0.0.1:6379>
  ```

  说明 Redis 正常运行，默认端口是 `6379`。

  ⚠️ 如果提示 `Could not connect to Redis`，说明 Redis 没有启动或端口不对。

  ------

  ### ✅ 二、查看 Redis 使用的端口号

  #### 方法1：查看配置文件 `redis.windows.conf` 或 `redis.conf`

  - 找到你 Redis 安装目录下的配置文件。
  - 搜索：

  ```
  port 6379
  ```

  这个就是配置的端口号。

  ------

  #### 方法2：查看正在监听的端口（系统层）

  用 PowerShell 或 cmd：

  ```
  netstat -ano | findstr :6379
  ```

  如果你看到如下类似内容：

  ```
  TCP    127.0.0.1:6379     0.0.0.0:0    LISTENING     12345
  ```

  说明端口 `6379` 正在被监听，PID 是 `12345`，你可以再查这个 PID 是哪个程序：

  ```
  tasklist | findstr 12345
  ```

  ------

  ### ✅ 三、Redis 日志查看（如果有日志路径）

  如果你启动 Redis 是通过手动命令，比如：

  ```
  redis-server.exe redis.windows.conf
  ```

  那么日志会输出到控制台或者你配置的文件（配置中有类似）：

  ```
  logfile "redis.log"
  ```

  你也可以查找一下日志路径或标准输出中是否有提示“Server started”之类信息。

  ------

  ### ✅ 四、如果你用的是 Docker 方式安装 Redis

  运行：

  ```
  docker ps
  ```

  可以看到类似：

  ```
  CONTAINER ID  IMAGE       PORTS                     NAMES
  abc123456     redis       0.0.0.0:6379->6379/tcp    redis
  ```

  说明 Redis 正在运行，并且暴露了 `6379` 端口。

  ------

  ### ✅ 小结

  | 目的       | 方法                                           |
  | ---------- | ---------------------------------------------- |
  | 是否运行   | `tasklist                                      |
  | 查看端口   | `redis.conf` 中 `port` 配置项 或 `netstat -ano |
  | 查看日志   | 配置文件中 `logfile` 位置 或 Docker 日志       |
  | Docker方式 | `docker ps` 查看容器和端口映射                 |



# 怎么启动redis





