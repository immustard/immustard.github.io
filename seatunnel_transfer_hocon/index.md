# SeaTunnel 配置文件从 hocon 解析为 Json 过程


<!--more-->



> SeaTunnel 使用的脚本的格式是 `hocon` 格式的, 通过观察日志, 可以看到启动之前会将 `hocon` 格式转为 `json` 格式. 但是发现会将 `source`、`transform`、`sink` 放在 JsonArray 中, 而不是 JsonObject 中. 带着这个问题, 去看一下源码中是如何将 `hocon` 转为 `json` 的. 

> SeaTunnel 版本为 v2.3.3

## `HOCON` 是什么

HOCON (Human-Optimized Config Object Notation) 是一种易于使用的配置格式. `hocon` 是 `json` 文件的超集, 也就是说, `json` 文件可用作 `hocon` 文件类型. `hocon` 语法等同于严格的 `json` 语法. HOCON 文件通常使用后缀 `.conf`. 

### 优点

* 轻量级
* 表示复杂的映射非常容易
* 可读写性强
* 使用任何文本编辑器都能轻松修改
* 适用于配置
* 合并简单
* 代换简单
* 可以声明复杂的配置

### 缺点

* 是一种新的格式, 存在学习曲线
* 不太流行


## SeaTunnel 转换 `hocon` -> `json`

1. 因为用 `Spark` 引擎比较多, 所以从 `Spark` 启动命令开始. 

	`org.apache.seatunnel.core.starter.spark.SeaTunnelSpark`: 
	```java
	public static void main(String[] args) throws CommandException {
		// 解析启动命令
		SparkCommandArgs sparkCommandArgs =
				CommandLineUtils.parse(
						args,
						new SparkCommandArgs(),
						EngineType.SPARK2.getStarterShellName(),
						true);
		// 任务启动
		SeaTunnel.run(sparkCommandArgs.buildCommand());
	}
	```

2. `org.apache.seatunnel.core.starter.SeaTunnel`: 
	```java
	/**
	 * 这个方法是 SeaTunnel 的入口.
	 *
	 * @param command 启动命令参数
	 * @param <T> 启动命令类型
	 */
	public static <T extends CommandArgs> void run(Command<T> command) throws CommandException {
		try {
			// 命令启动
			command.execute();
		} catch (ConfigRuntimeException e) {
			showConfigError(e);
			throw e;
		} catch (Exception e) {
			showFatalError(e);
			throw e;
		}
	}
	```

3. 接下来看 `Spark` 的解析过程, `org.apache.seatunnel.core.starter.spark.command.SparkTaskExecuteCommand`: 
	```java
	@Override
	public void execute() throws CommandExecuteException {
		// 读取配置文件地址
		Path configFile = FileUtils.getConfigPath(sparkCommandArgs);
		// 检查配置文件是否存在, 不存在抛出异常
		checkConfigExist(configFile);
		// 开始解析
		Config config = ConfigBuilder.of(configFile);
		
		...
	}
	```
	
4. `org.apache.seatunnel.core.starter.utils.ConfigBuilder`:
	```java
	public static Config of(@NonNull Path filePath) {
		log.info("Loading config file from path: {}", filePath);
		// 根据配置文件选择适配器
		Optional<ConfigAdapter> adapterSupplier = ConfigAdapterUtils.selectAdapter(filePath);
		// 根据适配器开始转换
		Config config =
				adapterSupplier
						.map(adapter -> of(adapter, filePath))
						.orElseGet(() -> ofInner(filePath));
		log.info("Parsed config file: {}", config.root().render(CONFIG_RENDER_OPTIONS));
		return config;
	}
	
	public static Config of(@NonNull ConfigAdapter configAdapter, @NonNull Path filePath) {
		log.info("With config adapter spi {}", configAdapter.getClass().getName());
		try {
			// 读取配置
			Map<String, Object> flattenedMap = configAdapter.loadConfig(filePath);
			// 解析配置文件转换的 map
			Config config = ConfigFactory.parseMap(flattenedMap);
			return ConfigShadeUtils.decryptConfig(config);
		} catch (Exception warn) {
			log.warn(
					"Loading config failed with spi {}, fallback to HOCON loader.",
					configAdapter.getClass().getName());
			return ofInner(filePath);
		}
	}
	```
	
5. 一路跳转, 会跳转到 `org.apache.seatunnel.shade.com.typesafe.config.impl.Parseable` 中, 从这个包名中就能看出来, 这个已经是 `hocon` 的解析包了: 
	```java
	// 最终会跳转到这个方法
	private AbstractConfigValue rawParseValue(Reader reader, ConfigOrigin origin,
			ConfigParseOptions finalOptions) throws IOException {
		if (finalOptions.getSyntax() == ConfigSyntax.PROPERTIES) {
			// 最终语法是 properties 的话会进入这个分支
			return PropertiesParser.parse(reader, origin);
		} else {
			Iterator<Token> tokens = Tokenizer.tokenize(origin, reader, finalOptions.getSyntax());
			ConfigNodeRoot document = ConfigDocumentParser.parse(tokens, origin, finalOptions);
			// 这个 ConfigParser 就是最终转换 hoconf 的类
			return ConfigParser.parse(document, origin, finalOptions, includeContext());
		}
	}	
	```
	
6. 最后跳转到这个类 `org.apache.seatunnel.shade.com.typesafe.config.impl.ConfigParser`, 因为这个类中的代码较多, 就不贴在这里了. 总体的思想就是按行读取配置, 通过对内容的收尾进行判断配置的层级, 从而进行转换. 
	``` java
	// 单独将这个方法拿出来是要指出一点, 为什么 SeaTunnel 将 hocon 转为 json 的时候, source、transform、sink 这三个模块是数组类型, 而不是对象. 如果用通用的 hocon 转 json 方法, 那么不能实现 SeaTunnel 的多源. 
	private AbstractConfigValue parseValue(AbstractConfigNodeValue n, List<String> comments) {
		AbstractConfigValue v;
	
		int startingArrayCount = arrayCount;
	
		if (n instanceof ConfigNodeSimpleValue) {
			v = ((ConfigNodeSimpleValue) n).value();
		} else if (n instanceof ConfigNodeObject) {
	
			Path path = pathStack.peekFirst();
			
			// 从这个分支的判断条件就能看出来, 这三个块是特殊处理的, 是将这三个转换为数组而不是对象
			if (path != null
					&& !ConfigSyntax.JSON.equals(flavor)
					&& ("source".equals(path.first())
							|| "transform".equals(path.first())
							|| "sink".equals(path.first()))) {
				v = parseObjectForSeaTunnel((ConfigNodeObject) n);
			} else {
				v = parseObject((ConfigNodeObject) n);
			}
	
		} else if (n instanceof ConfigNodeArray) {
			v = parseArray((ConfigNodeArray) n);
		} else if (n instanceof ConfigNodeConcatenation) {
			v = parseConcatenation((ConfigNodeConcatenation) n);
		} else {
			throw parseError("Expecting a value but got wrong node type: " + n.getClass());
		}
	
		if (comments != null && !comments.isEmpty()) {
			v = v.withOrigin(v.origin().prependComments(new ArrayList<>(comments)));
			comments.clear();
		}
	
		if (arrayCount != startingArrayCount) {
			throw new ConfigException.BugOrBroken(
					"Bug in config parser: unbalanced array count");
		}
	
		return v;
	}	
	```


---

> 作者: [Mustard](https://github.com/immustard)  
> URL: https://buli-home.cn/seatunnel_transfer_hocon/  

