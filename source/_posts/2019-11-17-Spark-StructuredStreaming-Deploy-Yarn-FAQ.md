---
title: Spark-StructuredStreaming-Deploy-Yarn-FAQ
date: 2019-11-17 11:09:39
categories:
    - Spark
tags:
    - Spark Structured Streaming
    - Spark Yarn Deploy
    - Spark FAQ
---

# 简介

这里主要整理`Spark`的`StructuredStreaming`程序包部署到`Yarn`上碰到的常见问题问题

# 问题列表

## 1.`java.lang.ClassNotFoundException: kafka.DefaultSource`

### 1.1 触发场景和原因

#### 1.1.1 场景一

`Maven` 或 `SBT` 没有自动把 `spark-sql-kafka` 包里的 `META-INF` 文件夹放进包里,但是那个文件很重要

### 1.2 解决方案

#### 1.2.1 方案一

`spark-submit`参数里添加`--packages org.apache.spark:spark-sql-kafka-xxxxx`参数,例如

```bash
spark-submit \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.11:2.1.0 \
  --class com.mygroup.myclass \
  --master local[*] \
  my-structured-streaming.jar
```

#### 1.2.2 方案二

在`Maven`或`SBT`里手动配置,[Maven配置方法](https://maven.apache.org/plugins-archives/maven-shade-plugin-2.0/examples/resource-transformers.html),例如

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>
                            META-INF/services/org.apache.spark.sql.sources.DataSourceRegister
                        </resource>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 2.`java.io.FileNotFoundException: File does not exist: hdfs://xxx/user/root/.sparkStaging/application_xxx/xxx.jar`

### 2.1 触发场景和原因

#### 2.1.1 场景一

`HDP`从2.x升级到3.x后,`yarn cluster`运行模式下,在`spark-submit`时候指定程序包为本地`jar`文件并不会像之前那样自动放入`HDFS`中让所有`worker`都能读到

### 2.2 解决方案

#### 2.2.1 方案一

手动把`jar`文件放入`HDFS`中,然后提交任务时,指定`HDFS`路径,例如
```bash
hdfs dfs -put xxxx.jar /
spark-submit \
    --class com.mygroup.myclass \
    --master yarn  \
    --deploy-mode cluster \
    hdfs://slave1:8020/xxxx.jar
```