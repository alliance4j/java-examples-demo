项目结构：
springboot-sql-tree/
├── src/main/java/com/example/sqltree/
│   ├── SqlTreeApplication.java        # 启动类
│   ├── SqlInterceptor.java            # MyBatis拦截器
│   ├── SqlCallTreeContext.java        # 调用树上下文管理
│   ├── SqlNode.java                   # SQL节点数据模型
│   ├── SqlTreeController.java         # REST API控制器
│   ├── DemoController.java            # 演示API
│   ├── UserService.java               # 用户服务（演示用）
│   ├── UserMapper.java                # 用户数据访问
│   └── OrderMapper.java               # 订单数据访问
├── src/main/resources/
│   ├── application.yml                # 应用配置
│   ├── schema.sql                     # 数据库表结构
│   ├── data.sql                       # 示例数据
│   └── static/
│       ├── index.html                 # 前端页面
│       └── sql-tree.js                # 前端JavaScript
└── pom.xml                            # Maven配置

---------------------------------------------------------
技术栈
 后端技术栈
  - Spring Boot 3.4.5： 应用框架
    - MyBatis 3.0.3： 数据访问层和拦截器
    - H2 Database： 内存数据库（演示用）
    - Lombok： 简化代码编写
    - Jackson： JSON序列化
 前端技术栈
    - HTML5 + Tailwind CSS： 页面结构和样式
    - D3.js v7： 数据可视化
    - Font Awesome： 图标库
    - 原生JavaScript： 前端交互逻辑
    - 
---------------------------------------------------------
核心实现详解：
- 1.MyBatis拦截器：零侵入的核心 - SqlInterceptor，这是整个系统的核心组件，通过MyBatis的插件机制实现SQL执行的无感知拦截
- 关键特性：
- 精准拦截： 同时拦截查询和更新操作
- 异常安全： 确保业务逻辑不受监控影响
- 丰富信息： 自动提取Service调用信息和执行统计

- 2.调用树上下文管理器：线程安全的数据管理 - SqlCallTreeContext，负责管理SQL调用树的构建和存储，采用线程安全的设计
- 线程安全： 使用ThreadLocal确保多线程环境下的数据隔离
- 智能建树： 自动识别父子关系，构建完整调用树
- 实时统计： 同步更新性能统计信息

- 3.数据模型：完整的SQL节点信息 - SqlNode，存储和传递SQL执行的详细信息，支持递归查询调用树

- 4.RESTFUL API：完整的数据接口 - SqlTreeController，提供完整的REST API接口，支持数据查询、配置管理和系统监控

- 5.前端可视化实现 - 前端使用D3.js实现交互式的SQL调用树可视化
- 核心特性：
- 树形布局： 清晰展示SQL调用层次关系
- 颜色编码： 绿色(正常)、红色(慢SQL)
- 交互操作： 点击节点查看详情，悬停显示提示
- 智能筛选： 支持按执行时间、SQL类型等条件筛选
- 实时刷新： 支持自动/手动刷新数据
--------------------------------------------------------

快速开始
- 环境要求
Java 21+
Maven 3.6+
现代浏览器（支持ES6+）

-访问系统
启动成功后，可以通过以下地址访问：
可视化界面：http://localhost:8080/index.html
H2数据库控制台：http://localhost:8080/h2-console

---------------------------------------------------------
实际应用场景
开发调试场景
场景1：复杂查询性能分析：/api/demo/user/1/detail
UserService.getUserDetailWithOrders()
├── SELECT * FROM users WHERE id = ? (2ms)
└── SELECT * FROM orders WHERE user_id = ? (15ms)
    └── SELECT * FROM order_items WHERE order_id IN (...) (45ms)

场景2：慢SQL识别
{
    "nodeId":"uuid-123",
    "sql":"SELECT * FROM orders o JOIN users u ON o.user_id = u.id WHERE o.status = ?",
    "executionTime":1250,
    "slowSql":true,
    "serviceName":"OrderService",
    "methodName":"getOrdersWithUserInfo"
}