# SeaTunnel 配置文件加解密


<!--more-->



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
	# (the "License"); you may not use this file except in compliance with
	# the License.  You may obtain a copy of the License at
	#
	#     http://www.apache.org/licenses/LICENSE-2.0
	#
	# Unless required by applicable law or agreed to in writing, software
	# distributed under the License is distributed on an "AS IS" BASIS,
	# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	# See the License for the specific language governing permissions and
	# limitations under the License.
	#
	
	env {
	  execution.parallelism = 1
	  shade.identifier = "base64"
	}
	
	source {
	  MySQL-CDC {
		result_table_name = "fake"
		parallelism = 1
		server-id = 5656
		port = 56725
		hostname = "127.0.0.1"
		username = "seatunnel"
		password = "seatunnel_password"
		database-name = "inventory_vwyw0n"
		table-name = "products"
		base-url = "jdbc:mysql://localhost:56725"
	  }
	}
	
	transform {
	}
	
	sink {
	  # choose stdout output plugin to output data to console
	  Clickhouse {
		host = "localhost:8123"
		database = "default"
		table = "fake_all"
		username = "seatunnel"
		password = "seatunnel_password"
	
		# cdc options
		primary_key = "id"
		support_upsert = true
	  }
	}	
	```
2. 使用基于不同计算引擎的命令行来加密配置文件, 🌰中使用 Zeta (SeaTunnel 自研):
	```shell
	> ${SEATUNNEL_HOME}/bin/seatunnel.sh --config config/v2.batch.template --encrypt
	```
	然后就可以在命令行中看到加密的配置:
	```json
	{
		"env" : {
			"execution.parallelism" : 1,
			"shade.identifier" : "base64"
		},
		"source" : [
			{
				"base-url" : "jdbc:mysql://localhost:56725",
				"hostname" : "127.0.0.1",
				"password" : "c2VhdHVubmVsX3Bhc3N3b3Jk",
				"port" : 56725,
				"database-name" : "inventory_vwyw0n",
				"parallelism" : 1,
				"result_table_name" : "fake",
				"table-name" : "products",
				"plugin_name" : "MySQL-CDC",
				"server-id" : 5656,
				"username" : "c2VhdHVubmVs"
			}
		],
		"transform" : [],
		"sink" : [
			{
				"database" : "default",
				"password" : "c2VhdHVubmVsX3Bhc3N3b3Jk",
				"support_upsert" : true,
				"host" : "localhost:8123",
				"plugin_name" : "Clickhouse",
				"primary_key" : "id",
				"table" : "fake_all",
				"username" : "c2VhdHVubmVs"
			}
		]
	}
	```
3. 当然, 不仅支持加密配置文件, 如果用户想查看解密后的配置文件, 可以执行以下命令: 
	```shell
	> ${SEATUNNEL_HOME}/bin/seatunnel.sh --config config/v2.batch.template --decrypt
	```
	
## 如何实现用户自定义加解密
如果想自定义加密方法和加密配置, 本节将帮助来解决问题. 

1. 创建一个 Java 的 maven 项目
2. 添加 `seatunnel-api` 模块在 `pom.xml`中:
	```xml
	<dependency>
		<groupId>org.apache.seatunnel</groupId>
		<artifactId>seatunnel-api</artifactId>
		<version>${seatunnel.version}</version>
	</dependency>
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

