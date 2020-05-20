---
title: spark连接mysql
date: 2016-04-20 22:33:03
tags: [Spark]
---

QQ: 380800878, 微信: kittenll

# 一、下到驱动

官网载mysql的[java驱动包](https://www.mysql.com/)放到spark根目录。启动：

    SPARK_CLASSPATH=mysql-connector-java-x.x.x-bin.jar ./bin/spark-shell

# 二、连接mysql

```scala
val df = sqlContext.load("jdbc", Map("url" -> "jdbc:mysql://localhost:3306/newlaw?user=root&password=", "dbtable" -> "tax"))
df.count()
```

这里得到的`df`是一个DataFrame对象，可以通过这个对象选取表数据，map, reduce等等操作。
