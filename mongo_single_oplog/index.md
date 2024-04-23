# MongoDB 单节点开启 oplog


&lt;!--more--&gt;



&gt; MongoDB 单节点是没有 oplog 的, 但是在测试的时候或者在生产环境, 是有存在单节点的情况的. 这个文档介绍一下是怎么单节点开启 oplog 的.

## 修改配置文件

在 `mongodb.conf` 中添加一行

```conf
replSet=single
```

## 重启 MongoDB

### 停止 MongoDB

```shell
&gt; use admin
switched to db admin
&gt; db.shutdownServer()
```

如果报错这个错误的话, 需要添加权限: 

```shell
2023-10-16T11:15:16.895&#43;0800 E QUERY    [js] Error: shutdownServer failed: {
        &#34;ok&#34; : 0,
        &#34;errmsg&#34; : &#34;not authorized on admin to execute command { shutdown: 1.0, lsid: { id: UUID(\&#34;e5e69fbd-9e73-4e49-b43d-ffd71b82075a\&#34;) }, $db: \&#34;admin\&#34; }&#34;,
        &#34;code&#34; : 13,
        &#34;codeName&#34; : &#34;Unauthorized&#34;
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DB.prototype.shutdownServer@src/mongo/shell/db.js:454:1
@(shell):1:1

&gt; db.grantRolesToUser( &#34;mustard&#34; , [ { role: &#34;hostManager&#34;, db: &#34;admin&#34; } ])
```

### 启动 MongoDB

```shell
# ./mongod --config mongodb.conf --fork
```

## 初始化副本集的脚本

需要在 `local` 或者 `admin` 数据库中执行: 

```shell
&gt; rs.initiate({ _id: &#34;副本集名称&#34;, members: [{_id:0,host:&#34;服务器的IP:Mongo的端口号&#34;}]})
```

&gt; 如果报这个错误则, 需要添加权限: 
&gt;
&gt; ```shell
&gt; {
&gt; 		&#34;ok&#34; : 0,
&gt; 		&#34;errmsg&#34; : &#34;not authorized on admin to execute command { replSetInitiate: { _id: \&#34;single\&#34;, members: [ { _id: 0.0, host: \&#34;10.140.6.26:27017\&#34; } ] }, lsid: { id: UUID(\&#34;7fdda9e0-3a2b-469b-93b6-e5e92f2f4997\&#34;) }, $db: \&#34;admin\&#34; }&#34;,
&gt; 		&#34;code&#34; : 13,
&gt; 		&#34;codeName&#34; : &#34;Unauthorized&#34;
&gt; }
&gt; 
&gt; &gt; db.grantRolesToUser( &#34;mustard&#34; , [ { role: &#34;clusterManager&#34;, db: &#34;admin&#34; } ])
&gt; ```

初始完, 副本集中唯一的节点, 可能短时间显示为 `SECONDARY` 或 `OTHER`. 一般而言, 稍等一会, 就会自然恢复为 `PRIMARY`, 无需人工干预. 


---

> 作者:   
> URL: https://buli-home.cn/mongo_single_oplog/  

