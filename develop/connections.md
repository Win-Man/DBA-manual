# 连接池

## HikariCP 连接池

[HikariConfig 文档](https://www.javadoc.io/doc/com.zaxxer/HikariCP/4.0.3/com/zaxxer/hikari/HikariConfig.html)

## 常用属性

1. autoCommit 
这个属性控制连接凡辉池中前 auto-commit 是否自动进行。
默认值 true ，建议值 true

2. connectionTimeout
控制一个客户端等待从连接池中获取连接的最大时间。超过该时间获取不到连接则抛出 SQLEXception 异常。
默认值 30000ms ，建议值 30000ms

3. idleTimeout 
控制空闲连接在池中最大的空闲时间。该配置只有在配置了 minimumIdle 属性