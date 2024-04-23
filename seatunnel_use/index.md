# Seatunnel使用手册


&lt;!--more--&gt;



&gt; 官方文档: [传送门](https://seatunnel.apache.org/)

&gt; 本文档针对 SeaTunnel v2.1.2 编写

---

## SeaTunnel是什么

### 概述

先来看一下官方文档上是怎么说的:

SeaTunnel is a very easy-to-use ultra-high-performance distributed data integration platform that supports real-time synchronization of massive data. It can synchronize tens of billions of data stably and efficiently every day, and has been used in the production of nearly 100 companies.

&gt; 翻译一下: 
&gt;
&gt; SeaTunnel 是一个使用起来非常简单, 性能非常高效的分布式数据集成平台. 它支持海量数据的实时同步. 它可以每天稳定高效的同步数百亿的数据, 并且已经用于近百个公司的生产中. 

可以从上面提炼出几个关键词:

1. very easy-to-use: 使用非常简单. 其实 SeaTunnel 并不是对 Flink 或是 Spark 或是以后支持的其他技术的二次开发, 而是在其之上封装了一层, 使得这些技术使用起来会更简便. 
2. ultra-high-performance: 超高性能. 
3. real-time synchronization of massive data: 海量数据的实时同步. 



&gt; 以下是从官方文档摘抄: 

&gt; ### 为什么我们需要 SeaTunnel
&gt;
&gt; SeaTunnel 尽所能为您解决海量数据同步中可能遇到的问题：
&gt;
&gt; - 数据丢失与重复
&gt; - 任务堆积与延迟
&gt; - 吞吐量低
&gt; - 应用到生产环境周期长
&gt; - 缺少应用运行状态监控

&gt; ### SeaTunnel 使用场景
&gt;
&gt; - 海量数据同步
&gt; - 海量数据集成
&gt; - 海量数据的 ETL
&gt; - 海量数据聚合
&gt; - 多源数据处理



## 环境依赖

| 组件  | 版本 | 备注                                                         |
| :---: | ---- | ------------------------------------------------------------ |
| Java  | &gt;=8  |                                                              |
| Flink | 1.13 | 因为当前版本SeaTunnel还未适配Flink1.14, 所以使用Flink1.13版本进行文档编写 |
| Spark | 2.x  | 如果是集群方式的话, Yarn/Standalone 都是支持的               |



### 环境配置

`config`目录下`seatunnel-env.sh`中可以配置Spark和Flink的环境

```
# Home directory of spark distribution.
SPARK_HOME=${SPARK_HOME:-/opt/spark}
# Home directory of flink distribution.
FLINK_HOME=${FLINK_HOME:-/opt/flink}
```

&gt; `:-`代表: 若未找到之前的地址, 则用之后的地址



## 官方示例

首先来进行一个官方示例简单的运行一下. 

在`config`目录下创建文件`example.conf`: 

```
# 配置 Spark 或 Flink 的参数
env {
    # You can set flink configuration here
    execution.parallelism = 1
    #execution.checkpoint.interval = 10000
    #execution.checkpoint.data-uri = &#34;hdfs://hadoop102:9092/checkpoint&#34;
}

# 在 source 所属的块中配置数据源
# 默认端口: 9999
source {
    SocketStream{
        host = hadoop102
        result_table_name = &#34;fake&#34;
        field_name = &#34;info&#34;
    }
}

# 在 transform 的块中声明转换插件
# 这里需要说明的是: Split是不会立即生效的, 只有当sql插件中的sql语句中调用了split函数才会真正的作用在数据上
transform {
    Split{
        separator = &#34;#&#34;
        fields = [&#34;name&#34;,&#34;age&#34;]
    }
    sql {
        sql = &#34;select info, split(info) as info_row from fake&#34;
    }
}
# 在 sink 块中声明要输出到哪
sink {
     ConsoleSink {}
}
```

然后`cd`到seatunnel目录在shell中执行:

```
./bin/start-seatunnel-flink.sh --config config/example.conf
```

&gt; 用`nc -lk 9999`模拟一下socket连接



### sql执行顺序

1. 在`source`块中, 利用`SocketStream`插件读取出数据, 命名为`fake`表, 字段名为`info`
2. 拿到`info`字段, 利用`Split`插件进行切分



### 启动命令

```
./bin/start-seatunnel-flink.sh -h
Usage: start-seatunnel-flink.sh [options]
  Options:
    -t, --check    check config (default: false)
  * -c, --config   Config file
    -h, --help     Show the usage message
    -r, --run-mode job run mode, run or run-application (default: RUN) 
                   (values: [RUN, APPLICATION_RUN])
    -i, --variable variable substitution, such as -i city=beijing, or -i 
                   date=20190318 (default: [])
```

**其中, `--config`是必填参数**

- `-i`

  当其中上面的sql改为:

  ```
  sql = &#34;select * from (select info, split(info) as info_row from fake) where age &gt; &#34;${age}&#34;&#34;
  ```

  启动命令改为: 

  ```
  ./bin/start-seatunnel-flink.sh --config config/example02.conf -i age=18
  ```

- `-r`

  执行 flink 自带的命令参数, 可以`cd`到 flink 下面 -&gt; `./bin/flink run -h`查看



## 应用配置的4个基本组件

一个完整的SeaTunnel配置文件应包含四个配置组件: 

```
env{}`  `source{}` --&gt; `transform{}` --&gt; `sink{}
```



### env块

env块中可以直接写 spark 或 flink 支持的配置项. 比如并行度, 检查点时间, 检查点 hdfs 路径等. 以 Flink 为例, 在 SeaTunnel 源码的`ConfigKeyName`类中声明了所有可用的key:

```
package org.apache.seatunnel.flink.util;

public class ConfigKeyName {

    private ConfigKeyName() {
        throw new IllegalStateException(&#34;Utility class&#34;);
    }

    public static final String TIME_CHARACTERISTIC = &#34;execution.time-characteristic&#34;;
    public static final String BUFFER_TIMEOUT_MILLIS = &#34;execution.buffer.timeout&#34;;
    public static final String PARALLELISM = &#34;execution.parallelism&#34;;
    public static final String MAX_PARALLELISM = &#34;execution.max-parallelism&#34;;
    public static final String CHECKPOINT_INTERVAL = &#34;execution.checkpoint.interval&#34;;
    public static final String CHECKPOINT_MODE = &#34;execution.checkpoint.mode&#34;;
    public static final String CHECKPOINT_TIMEOUT = &#34;execution.checkpoint.timeout&#34;;
    public static final String CHECKPOINT_DATA_URI = &#34;execution.checkpoint.data-uri&#34;;
    public static final String MAX_CONCURRENT_CHECKPOINTS = &#34;execution.max-concurrent-checkpoints&#34;;
    public static final String CHECKPOINT_CLEANUP_MODE = &#34;execution.checkpoint.cleanup-mode&#34;;
    public static final String MIN_PAUSE_BETWEEN_CHECKPOINTS = &#34;execution.checkpoint.min-pause&#34;;
    public static final String FAIL_ON_CHECKPOINTING_ERRORS = &#34;execution.checkpoint.fail-on-error&#34;;
    public static final String RESTART_STRATEGY = &#34;execution.restart.strategy&#34;;
    public static final String RESTART_ATTEMPTS = &#34;execution.restart.attempts&#34;;
    public static final String RESTART_DELAY_BETWEEN_ATTEMPTS = &#34;execution.restart.delayBetweenAttempts&#34;;
    public static final String RESTART_FAILURE_INTERVAL = &#34;execution.restart.failureInterval&#34;;
    public static final String RESTART_FAILURE_RATE = &#34;execution.restart.failureRate&#34;;
    public static final String RESTART_DELAY_INTERVAL = &#34;execution.restart.delayInterval&#34;;
    public static final String MAX_STATE_RETENTION_TIME = &#34;execution.query.state.max-retention&#34;;
    public static final String MIN_STATE_RETENTION_TIME = &#34;execution.query.state.min-retention&#34;;
    public static final String STATE_BACKEND = &#34;execution.state.backend&#34;;
    public static final String PLANNER = &#34;execution.planner&#34;;
}
```



### Row

在说明`source块`, `transform块`和`sink块`之前, 需要先了解一下 SeaTunnel 中的核心数据结构: `Row`

Row 是 SeaTunnel 中数据传递的核心数据结构. 对来 Flink 说, source 插件需要给下游的转换插件返回一个 `DataStream&lt;Row&gt;`, 转换插件接到上游的 `DataStream&lt;Row&gt;`进行处理后需要再给下游返回一个 `DataStream&lt;Row&gt;`. 最后 Sink 插件将转换插件处理好的`DataStream&lt;Row&gt;`输出到外部的数据系统. 

&gt; 因为 `DataStream`可以很方便地和 Table 进行互转, 所以将 Row 当作核心数据结构可以让转换插件同时具有使用代码 (命令式) 和 sql (声明式) 处理数据的能力. 

可以看一下上面示例中, 读取数据的源码: 

```
package org.apache.seatunnel.flink.socket.source;

import ...

@AutoService(BaseFlinkSource.class)
public class SocketStream implements FlinkStreamSource {

    ...

    @Override
    public DataStream&lt;Row&gt; getData(FlinkEnvironment env) {
        final StreamExecutionEnvironment environment = env.getStreamExecutionEnvironment();
        return environment.socketTextStream(host, port)
                .map((MapFunction&lt;String, Row&gt;) value -&gt; {
                    Row row = new Row(1);
                    row.setField(0, value);
                    return row;
                }).returns(new RowTypeInfo(Types.STRING()));
    }
}
```

感兴趣的话, 也可以看到源码中的 Split, sql, sink 都是用`DataStream&lt;Row&gt;`进行数据传递的



### source块

`source{}`是可以配置多个 source 插件的

```
# 伪代码
env {
    ...
}

source {
    hdfs { ... }
    jdbc { ... }
    elasticsearch { ... }
}

transform {
    sql {
        sql = &#34;&#34;&#34;
            select ... from hdfs_table
            join es_table on
            hdfs_table.uid = es_table.uid
            where ...&#34;&#34;&#34;
    }
}

sink {
    elasticsearch { ... }
}
```

&gt; 需要注意的是: 所有的 source 插件中都可以声明`result_table_name`. 如果声明了`result_table_name`. SeaTunnel 会将 source 插件输出的`DataStream&lt;Row&gt;`转换为 Table 并注册在 Table 环境中. 当指定了`result_table_name`那么还可以指定`field_name`, 在注册时, 给 Table重设字段名. 

&gt; 因为每个 source 所需要的配置是不一致的 (`result_table_name`和`field_name`为共有非必填参数), 所以**配置的时候查找官方文档会好一点**



#### 当前支持的source

| source            | 支持Spark | 支持Flink | 备注                                                         | 文档地址                                                     |
| ----------------- | --------- | --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Druid**         | ❌         | ✔️         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Druid) |
| **Elasticsearch** | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Elasticsearch) |
| **Fake**          | ✔️         | ✔️         | 改类型主要是用于方便生成指定的数据, 用作 SeaTunnel 的功能验证, 测试和性能测试. | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Fake) |
| **Feishu Sheet**  | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/FeishuSheet) |
| **File**          | ✔️         | ✔️         | 从本地或者 hdfs 中读取.                                      | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/File) |
| **HBase**         | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Hbase) |
| **Hive**          | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Hive) |
| **Http**          | ✔️         | ✔️         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Http) |
| **Hudi**          | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Hudi) |
| **Iceberg**       | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Iceberg) |
| **InfluxDb**      | ❌         | ✔️         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/InfluxDb) |
| **Jdbc**          | ✔️         | ✔️         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Jdbc) |
| **Kafka**         | ✔️         | ✔️         | Kafka 版本 &gt;= 0.10.0, ~~目前源码中发现 Schema 的解析有问题~~(原因为社区把 fastjson 换成 Jackon 引起的). | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Kafka) |
| **Kudu**          | ✔️         | ❌         | 兼容 Kerberos 认证                                           | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Kudu) |
| **MongoDb**       | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/MongoDB) |
| **Neo4j**         | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/neo4j) |
| **Phoenix**       | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Phoenix) |
| **Redis**         | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Redis) |
| **Socket**        | ✔️         | ✔️         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Redis) |
| **Tidb**          | ✔️         | ❌         |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Redis) |
| **WebhookStream** | ✔️         | ❌         | 提供 http 接口推送数据, 仅支持 **POST** 请求                 | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/source/Redis) |



### transform块

目前社区对 transform 插件做了很多规划, 但截至 `v2.1.2` 版本, 可用的插件有3个: `Split`, `Sql`和`Json`. 其中`Json`只适配 Spark 可用. 

`transform{}`中可以声明多个转换插件. 所有的转换插件都可以使用`source_table_name`, 和`result_table_name`. 同样, 如果声明了`result_table_name`, 那么就能声明`field_name`. 



#### Split插件

这里着重说一下 Split 插件: 

```
    @Override
    public DataSet&lt;Row&gt; processBatch(FlinkEnvironment env, DataSet&lt;Row&gt; data) {
        return data;
    }

    @Override
    public DataStream&lt;Row&gt; processStream(FlinkEnvironment env, DataStream&lt;Row&gt; dataStream) {
        return dataStream;
    }

    @Override
    public void registerFunction(FlinkEnvironment flinkEnvironment) {
        if (flinkEnvironment.isStreaming()) {
            flinkEnvironment
                    .getStreamTableEnvironment()
                    .registerFunction(&#34;split&#34;, new ScalarSplit(rowTypeInfo, num, separator));
        } else {
            flinkEnvironment
                    .getBatchTableEnvironment()
                    .registerFunction(&#34;split&#34;, new ScalarSplit(rowTypeInfo, num, separator));
        }
    }
```

从源码中可以发现 Split 插件并没有对数据流进行任何的处理, 而是将它直接`return`了. 反之, 它向表环境中注册了一个名为 **split** 的 UDF(用户自定义函数). 而且, 函数名是写死的. 这意味着, **如果声明了多个 Split 后面的 UDF 还会把前面的覆盖**.

&gt; 从 Split 插件中就能看出了, 这个插件其实是通过注册方法的方式来调用的. 但是, transform 接口其实是预留了直接操作数据的能力的 (比如 Sql 插件中的处理方式), 也就是`processStream`方法. **那么, 一个 transform 插件其实同时履行了 process 和 UDF 的职责, 这是违背单一职责原则的. 所以要判断一个 transform 插件在做什么就只能从源码和文档的方面来加以区分了**. 



#### Sql 插件

sql插件中需要特别说明的是, 指定`source_table_name`对于 sql 插件的意义不大, 因为可以通过`from`子句来决定从哪个表里抽取数据. 



### Sink块

sink块里可以声明多个 sink 插件, 每个 sink 插件都可以指定`source_table_name`. 



#### 当前支持的sink

| source             | 支持Spark | 支持Flink | 备注                                                         | 文档地址                                                     |
| ------------------ | :-------: | :-------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Clickhouse**     |     ✔️     |     ✔️     | 使用 Clickhouse-jdbc 根据字段名称对应数据源, 并将其写入. 使用前需创建对应的数据表. | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Clickhouse) |
| **ClickhouseFile** |     ✔️     |     ✔️     | 通过 clickhouse-local 程序生成 Clickhouse 数据文件, 然后将其发送到 Clickhouse 服务器, 也称为是 bulk load (批量加载). | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/ClickhouseFile) |
| **Console**        |     ✔️     |     ✔️     | 将数据输出到标准终端或 Flink 的 TaskManager. 通常用于调试和易于观察的数据. | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Console) |
| **Doris**          |     ✔️     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Doris) |
| **Druid**          |     ❌     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Druid) |
| **Elasticsearch**  |     ✔️     |     ✔️     | Spark 支持的 Elasticsearch &gt;= 2.0 并且 &lt; 7.0.0; Flink 支持的 Elasticsearch = 7.x, 如果要用 Elasticsearch 6.x, 需用源码执行命令 `mvn clean package -Delasticsearch=6`重新打包. | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Elasticsearch) |
| **Email**          |     ✔️     |     ❌     | 支持通过 email 附件输出数据.                                 | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Email) |
| **File**           |     ✔️     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/File) |
| **Hbase**          |     ✔️     |     ✔️     | 使用 hbase-connectors 将数据输出到 Hbase(&gt;=2.1.0) 和 Spark(&gt;=2.0.0) 版本兼容性取决于 hbase-connectors. | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Hbase) |
| **Hive**           |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Hive) |
| **Hudi**           |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Hudi) |
| **Iceberg**        |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Iceberg) |
| **InfluxDB**       |     ❌️     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/InfluxDb) |
| **Jdbc**           |     ✔️     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Jdbc) |
| **Kafka**          |     ✔️     |     ✔️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Kafka) |
| **Kudu**           |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Kudu) |
| **MongoDB**        |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/MongoDB) |
| **Phoenix**        |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Phoenix) |
| **Redis**          |     ✔️     |     ❌     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Redis) |
| **TiDb**           |     ✔️     |     ❌️     |                                                              | [传送门](https://seatunnel.apache.org/docs/2.1.2/connector/sink/Tidb) |



---

> 作者:   
> URL: https://buli-home.cn/seatunnel_use/  

