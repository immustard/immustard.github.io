# 关于剥离 Taier 中 DatasourceX 组件


<!--more-->



### 前提

之前项目里用的数据源连接都是 [DatasourceX ](https://github.com/DTStack/DatasourceX) , 但是这个项目从2022年5月之后就没有维护过了, 其中也有一些bug都没有修复. 所以想着要不要自己写一个, 或者再看看有没有其他的. 

前两天在ChatGPT上询问了一下有没有类似的工具的时候, 推荐了同样是袋鼠云的 [Taier](https://github.com/DTStack/Taier) , 突然发现从 Taier 的 1.3.0 版本之后 DatasourceX 就移到这个项目中了. 所以就想着从中剥离出来 DatasourceX 放到自己的项目中去. 现在记录一下这个过程. 



### 开始

1. 第一步肯定是先把代码下载下来: (这里下载的是`1.3.0-RELEASE`)

   ```bash
   > git clone https://github.com/DTStack/Taier.git -b 1.3.0-RELEASE
   ```

2. 把代码下载下来之后找到 DatasoruceX 的目录: `taier-datasource`

   这个模块中有两个子模块: `taier-datasource-api` 和 `taier-datasource-plugin`. 其中: 

   * `taier-datasource-api` 这个就是需要剥离出来的接口. 
   * `taier-datasource-plugin` 这个里面就是各种数据源的依赖, 现在就不要这个了, 因为 GitHub 上已经下载好了现成的: [plugins 传送门](https://github.com/DTStack/Taier/releases/download/v1.3.0/taier.tar.gz)

3. 在打包之前先修改一下 `taier-datasource.pom.xml`:

   1. 将 `slf4j-api` 的 `<scope>` 去掉, 否则打完包会报找不到 `slf4j` 的错:

      ```xml
                  <dependency>
                      <groupId>org.slf4j</groupId>
                      <artifactId>slf4j-api</artifactId>
                      <version>1.7.21</version>
                  </dependency>
      ```

   2. 在 `<build>` 中加入 `maven-assembly-plugin` 插件, 目的是将 `taier-datasource-api` 的依赖也打到包里面去:

      ```xml
                  <plugin>
                      <groupId>org.apache.maven.plugins</groupId>
                      <artifactId>maven-assembly-plugin</artifactId>
                      <version>2.4.1</version>
                      <configuration>
                          <descriptorRefs>
                              <descriptorRef>jar-with-dependencies</descriptorRef>
                          </descriptorRefs>
                      </configuration>
                      <executions>
                          <execution>
                              <id>make-assembly</id>
                              <phase>package</phase>
                              <goals>
                                  <goal>single</goal>
                              </goals>
                          </execution>
                      </executions>
                  </plugin>
      ```

      这样打包之后会同时生成一个后面带 `jar-with-dependencies` 的包, 这个就是我们需要的了. 

   3. `mvn clean install` 或者 `mvn clean package`. 这样就打包成功了. 



### 导入

这样就生成了一个本地的 `jar`  包, 这里我用的导入方法是先将这个 `jar` 包导入到本地的 `maven` 仓库中. 

1. 导入到 `maven` 本地仓库:

   ```bash
   > mvn install:install-file -Dfile=上面打包的jar包路径 -DgroupId=com.dtstack.taier -DartifactId=taier.datasource.x -Dversion=1.0.0 -Dpackaging=jar
   ```

2. 在项目的 `pom` 中引入:

   ```xml
           <dependency>
               <groupId>com.dtstack.taier</groupId>
               <artifactId>taier.datasource.x</artifactId>
               <version>1.0.0</version>
           </dependency>
   ```



### 配置

这里需要对插件的载入重写一下: 

```java
import com.dtstack.taier.datasource.api.base.ClientCache;
import com.dtstack.taier.datasource.api.constant.ConfigConstants;
import com.dtstack.taier.datasource.api.context.ClientEnvironment;
import com.dtstack.taier.datasource.api.manager.list.ClientManager;
import com.dtstack.taier.datasource.api.config.Configuration;

private void p_initDatasourcePluginsDir() {
    String pluginsDir = 这里是插件路径

    Map<String, Object> config = new HashMap<>();
    config.put(ConfigConstants.PLUGIN_DIR, pluginsDir);

    Configuration configuration = new Configuration(config);

    ClientEnvironment clientEnvironment = new ClientEnvironment(configuration);
    clientEnvironment.start();
    ClientCache.setEnv(clientEnvironment.getManagerFactory().getManager(ClientManager.class));
}
```



至此, 这个就已经把 Taier 中的 DatasourceX 移进来了. 

