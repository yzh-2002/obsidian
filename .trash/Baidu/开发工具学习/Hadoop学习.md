## 体系简述

Hadoop是一套工具的总称，主要包含三部分：
1. HDFS：分布式文件存储
2. Yarn：资源调度
3. MapReduce：计算

工作流程：Yarn调度资源，读取HDFS文件内容进行MR计算。

---
什么是Hive？
在Hadoop之上添加了SQL解析和优化器，写一段代码，解析为Java代码，然后执行MR，底层数据仍然在HDFS上。Hive表也只是虚拟的表。

---
什么是Spark？
动机：MR频繁读写文件，速度较慢。故产生了基于内存的Spark，用于替代MR。

--- 
HDFS存储的数据一般都是离线数据，**实时数据**如何处理呢？**消息队列**在其中起到什么作用呢？
.....

## MapReduce

### MR已被淘汰？
