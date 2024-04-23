# SeaTunnel 配置文件加解密


&lt;!--more--&gt;



## 简介
在大多数生产环境中, 密码等敏感配置项需要加密, 不能以明文存储, SeaTunnel 为此提供了便捷的一站式解决方案. 

## 使用方法
SeaTunnel 自带 base64 加解密功能, 但不建议用于生产使用, 建议用户实现自定义加解密逻辑, 可以参考本章 [如何实现用户自定义加解密](#如何实现用户自定义加解密) 获取更多关于它的详细信息. 

Base64 加密支持加密以下参数:
* username
* password
* auth

接下来展示怎么使用 `base64` 进行加密:
1. 在配置文件中添加新的配置项 `shade.identifier`, 这个配置项指示要使用的加密方法, 在这个示例中, 应该在配置中添加 `shade.identifier = base64`, 如下所示：
	```config
	#
	# Licensed to the Apache Software Foundation (ASF) under one or more
	# contributor license agreements.  See the NOTICE file distributed with
	# this work for additional information regarding copyright ownership.
	# The ASF licenses this file to You under the Apache License, Version 2.0
	# (the &#34;License&#34;); you may not use this file except in compliance with
	# the License.  You may obtain a copy of the License at
	#
	#     http://www.apache.org/licenses/LICENSE-2.0
	#
	# Unless required by applicable law or agreed to in writing, software
	# distributed under the License is distributed on an &#34;AS IS&#34; BASIS,
	# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	# See the License for the specific language governing permissions and
	# limitations under the License.
	#
	
	env {
	  execution.parallelism = 1
	  shade.identifier = &#34;base64&#34;
	}
	
	source {
	  MySQL-CDC {
		result_table_name = &#34;fake&#34;
		parallelism = 1
		server-id = 5656
		port = 56725
		hostname = &#34;127.0.0.1&#34;
		username = &#34;seatunnel&#34;
		password = &#34;seatunnel_password&#34;
		database-name = &#34;inventory_vwyw0n&#34;
		table-name = &#34;products&#34;
		base-url = &#34;jdbc:mysql://localhost:56725&#34;
	  }
	}
	
	transform {
	}
	
	sink {
	  # choose stdout output plugin to output data to console
	  Clickhouse {
		host = &#34;localhost:8123&#34;
		database = &#34;default&#34;
		table = &#34;fake_all&#34;
		username = &#34;seatunnel&#34;
		password = &#34;seatunnel_password&#34;
	
		# cdc options
		primary_key = &#34;id&#34;
		support_upsert = true
	  }
	}	
	```
2. 使用基于不同计算引擎的命令行来加密配置文件, 🌰中使用 Zeta (SeaTunnel 自研):
	```shell
	&gt; ${SEATUNNEL_HOME}/bin/seatunnel.sh --config config/v2.batch.template --encrypt
	```
	然后就可以在命令行中看到加密的配置:
	```json
	{
		&#34;env&#34; : {
			&#34;execution.parallelism&#34; : 1,
			&#34;shade.identifier&#34; : &#34;base64&#34;
		},
		&#34;source&#34; : [
			{
				&#34;base-url&#34; : &#34;jdbc:mysql://localhost:56725&#34;,
				&#34;hostname&#34; : &#34;127.0.0.1&#34;,
				&#34;password&#34; : &#34;c2VhdHVubmVsX3Bhc3N3b3Jk&#34;,
				&#34;port&#34; : 56725,
				&#34;database-name&#34; : &#34;inventory_vwyw0n&#34;,
				&#34;parallelism&#34; : 1,
				&#34;result_table_name&#34; : &#34;fake&#34;,
				&#34;table-name&#34; : &#34;products&#34;,
				&#34;plugin_name&#34; : &#34;MySQL-CDC&#34;,
				&#34;server-id&#34; : 5656,
				&#34;username&#34; : &#34;c2VhdHVubmVs&#34;
			}
		],
		&#34;transform&#34; : [],
		&#34;sink&#34; : [
			{
				&#34;database&#34; : &#34;default&#34;,
				&#34;password&#34; : &#34;c2VhdHVubmVsX3Bhc3N3b3Jk&#34;,
				&#34;support_upsert&#34; : true,
				&#34;host&#34; : &#34;localhost:8123&#34;,
				&#34;plugin_name&#34; : &#34;Clickhouse&#34;,
				&#34;primary_key&#34; : &#34;id&#34;,
				&#34;table&#34; : &#34;fake_all&#34;,
				&#34;username&#34; : &#34;c2VhdHVubmVs&#34;
			}
		]
	}
	```
3. 当然, 不仅支持加密配置文件, 如果用户想查看解密后的配置文件, 可以执行以下命令: 
	```shell
	&gt; ${SEATUNNEL_HOME}/bin/seatunnel.sh --config config/v2.batch.template --decrypt
	```
	
## 如何实现用户自定义加解密
如果想自定义加密方法和加密配置, 本节将帮助来解决问题. 

1. 创建一个 Java 的 maven 项目
2. 添加 `seatunnel-api` 模块在 `pom.xml`中:
	```xml
	&lt;dependency&gt;
		&lt;groupId&gt;org.apache.seatunnel&lt;/groupId&gt;
		&lt;artifactId&gt;seatunnel-api&lt;/artifactId&gt;
		&lt;version&gt;${seatunnel.version}&lt;/version&gt;
	&lt;/dependency&gt;
	```
3. 创建一个新的类, 并实现 `ConfigShade` 接口, 这个接口有下列方法:
	```java
	/**
	 * The interface that provides the ability to encrypt and decrypt {@link
	 * org.apache.seatunnel.shade.com.typesafe.config.Config}
	 */
	public interface ConfigShade {
	
		/**
		 * The unique identifier of the current interface, used it to select the correct {@link
		 * ConfigShade}
		 */
		String getIdentifier();
	
		/**
		 * Encrypt the content
		 *
		 * @param content The content to encrypt
		 */
		String encrypt(String content);
	
		/**
		 * Decrypt the content
		 *
		 * @param content The content to decrypt
		 */
		String decrypt(String content);
	
		/** To expand the options that user want to encrypt */
		default String[] sensitiveOptions() {
			return new String[0];
		}
	}
	```
4. 在 `resources/META-INF/services` 添加 `org.apache.seatunnel.api.configuration.ConfigShade`.
5. 将其打成 jar 包, 并添加到 `${SEATUNNEL_HOME}/lib`.
6. 将选项 `shade.identifier` 的值更改为上面定义在配置文件中的 `ConfigShade#getIdentifier` 的值. 
7. 完成! 


---

> 作者:   
> URL: https://buli-home.cn/seatunnel_encryption/  

