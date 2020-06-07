---
title: Jedis工具类和Jackson工具类
date: 2020-05-26 11:54:46
categories: 后端学习
tags:
- [代码模板]
- [工具类]
---

Jackson工具类和Jedis工具类

<!-- more -->

## Jackson工具类

```java
public class JedisUtils {

	private static JedisPool jedisPool;
	static {
        // 将外部配置文件转为字节输入流
		InputStream in = JedisUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
		Properties p = new Properties();

		try {
			p.load(in);
            // 配置Jedis配置类
			JedisPoolConfig config = new JedisPoolConfig();
            // 设置maxIdle和maxTotal
			config.setMaxIdle(Integer.parseInt(p.getProperty("maxIdle")));
			config.setMaxTotal(Integer.parseInt(p.getProperty("maxTotal")));
			jedisPool = new JedisPool(config, p.getProperty("host"), Integer.parseInt(p.getProperty("port")));
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public static Jedis getJedis () {
        // 从jedisPool里拿出Jedis
		return jedisPool.getResource();
	}
}
```

### 外部的配置文件

```properties
host=ip地址
port=6379
maxIdle=10
maxTotal=50
```

除了用静态方法调用外，还可将Jedis工具类添加到容器中，将读取配置文件的代码放到bean的init方法中……

## Jackson工具类

将jackson进行简单的封装，可用于字符串和指定泛型和指定类型的转换

```java
public class JacksonUtils {
	private static ObjectMapper objectMapper = new ObjectMapper();

	/**
	 * 对象转成字符串的封装
	 * @param obj 要转的对象
	 * @return 转完的字符串
	 */
	public static String Obj2String (Object obj) {
		if (obj == null) {
			throw new RuntimeException("传入为null");
		}
		if (obj instanceof String) {
			return (String) obj;
		}

		try {
			return objectMapper.writeValueAsString(obj);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 * 将json字符串转为之指定类型的对象
	 * @param json 传入的json字符串
	 * @param clazz 要转成的类型
	 * @param <T> 泛型T
	 * @return 泛型T
	 */
	public static<T> T Json2T (String json, Class<T> clazz) {
		try {
			T t = objectMapper.readValue(json, clazz);
			return t;
		} catch (IOException e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 * 将json转为指定好泛型的List集合
	 * @param json json字符串
	 * @param clazz List集合的泛型
	 * @param <T> 泛型T
	 * @return 指定好泛型的List集合
	 */
	public static<T> List<T> Json2ListT (String json, Class<T> clazz) {
		CollectionType collectionType = objectMapper.getTypeFactory().constructCollectionType(List.class, clazz);
		try {
			List<T> list = objectMapper.readValue(json, collectionType);
			return list;
		} catch (IOException e) {
			e.printStackTrace();
		}
		return null;
	}

}
```





