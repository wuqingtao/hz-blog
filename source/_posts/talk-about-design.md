---
title: 浅谈设计和架构过程
date: 2016-12-05 21:39:35
tags:
---

# 前言

程序员大部分的时间都集中在做某个具体功能的实现，大部分的博客和技术论坛都只是谈论着怎么实现一个具体的功能，怎么使用一个新的框架，很少涉及涉及和架构。

事实上是，越来越丰富的框架、第三方模块、开源库，让我们实现一个具体功能变得容易，但是，软件的整体性能和迭代性并没有实质性的改善。

原因很简单，我们没有做好设计和架构，甚至从来做过什么单元测试。

我的意思是，如果我们想持续把程序员作为一个职业或者兴趣，让我们现在就开始时刻关注和提高自己的设计和架构能力。

好的架构应该达到以下目的：
* 有效缩短项目开发周期。
* 最大可能减少各种bug。
* 低成本的版本迭代。
* 很好的优化开发团队结构。

一个快速完整的架构历程应该是：
* 目录结构设计。
* 模块切分。
* 模块接口定义。
* 模块单元测试设计。
* 编译自动化。
* 部署自动化。

本文将以一个简单的web应用的例子，串联整个软件的开始周期，描述自己对设计、架构及单元测试的理解。

# 需求分析

假如需求是这样的：做一个web页面，功能类似于一个心情记事本，包括消息展示、发布和获取，点击[紅中随笔](http://120.55.185.245/message/)查看。

# 工程名称

工程名称将是整个项目生命周期最多被引用到的名词，所以多花点时间琢磨工程名称很要必要。

合适的工程名称应该满足：
* 简洁，不要过于冗长。
* 又不能太过简洁，能够大致描述项目目的。
* 不要用拼音或简拼。
* 也不要过分纠结。

我们把这个工程命名为messag，当然msg也不错，但是message还是太泛，我们再给他加点限制，message of hongzhong，也就是hz-message，至于是-还是_，都可以。
```bash
hz-message
```

# 目录结构

所有项目的源文件都将位于各个目录中，各个文件之间互相依赖调用，我们不希望在开发过程中发现目录结构或是目录名称定义不合理再来修改，而致代码日志混乱。

目录结构的定义应该满足以下原则：
* 各个子系统分开，如前端（web）和后端（server）。
* 每个子系统的不同组件分开，如源码（src）、测试（test）、配置（conf）和库目录（libs）。
* 每个组件的不同类型归类，如js、css，utils等。

目录名称的命名应该：
* 遵循惯例，如html、js、css、src、test、conf、libs、utils、bin、obj、tmp等。
* 惯例以外的名称不要过分简洁，亦不要太臃肿。
* 忌拼音或简拼，很容易不知道是什么意思。

对于hz-message，前端我们命令为web，www看上去更像是一个部署目录，html不足以涵盖前端的概念。当然，在这里，更好的做法应该是app/web，app意味前端，而前端可能会有web，也可能有android和ios。测试目录是必须的，因此web具备有两个子目录src和test。src分为html、js和css不同类型，hz-message把html文件直接至于src目录下也OK，毕竟js和css都归html调用。对于test，也应该分为html、js和css，hz-message虽然仅做了部分js相关的测试，考虑后续也许会做html相关测试，所以仅有js目录。

后端命名为server，srv好像也不错，不过我个人觉得太缩写。对于后端，虽然实际的hz-message实现了多个语言版本，这里只说java，server理所当然的分为src、test、conf和libs，conf和libs最好不要放在src下面，因为test中测试程序也可能调用conf和libs，同时也方便后续编译和打包。对于，src和test，我们增加层级域名的结构hz/message。

最后看上去是这样的：
```bash
hz-message
├── server
│   ├── conf
│   ├── src
│   │   └── hz
│   │       └── message
│   └── test
│       └── hz
│           └── message
└── web
    ├── src
    │   ├── css
    │   └── js
    └── test
        └── js
```

# 模块切分

我们先说server。

## Server

既然是服务，我们首先需要一个模块来实现服务入口。结构的搭建是hz-message演示的重点，所以我们不会使用tomcat来帮忙，当然也不会重新从底层开始写一个服务。Servt是java服务最合适的标准，我们使用Jetty嵌入式组件来实现hz-message java服务。

我们把这个模块命名为ServletServer，直接命名为Server也没问题，不过我想明确他是一个Servlet服务，而不是fcgi服务。
```bash
hz-message
└── server
    └── src
        └── hz
            └── message
                └── ServletServer.java
```

### ServletServer

接下来，我们按照Jetty Servlet规范写框架：
```java
/**
 * Servlet服务
 */
public class ServletServer {
	/**
	 * HTTP请求处理Handler
	 */
	private static class Handler extends AbstractHandler {
		@Override
		public void handle(String target, Request baseRequest,
				HttpServletRequest request, HttpServletResponse response) throws IOException,
				ServletException {
	}
}
```

从现在开始，我们的代码将持续注意java代码规范：
* 注释
* 各种类命名，类、函数、成员变量、参数、临时变量等
* 各种格式化，空格、换行等

OK，接下来我们要在hanlde函数中处理HTTP请求了：
* POST还是GET
* 如果是POST，POST的编码和参数规范又是什么，都得定义
* 我们应该把参数从字符串解析成一个对象模型
* 这个参数对象谁来处理
* 处理的结果的对象又是什么模型
* 接着要把结果对象转换成字符串返回
* 我们也要处理好各种异常
* 最后，作为整个Server的，也需要一个main函数来启动Server

对于ServletServer，我们进一步演化：
```java
/**
 * Servlet服务
 */
public class ServletServer {
	/**
	 * HTTP请求处理Handler
	 */
	private static class Handler extends AbstractHandler {
		@Override
		public void handle(String target, Request baseRequest,
				HttpServletRequest request, HttpServletResponse response) throws IOException,
				ServletException {
			try {
				// TODO: [1]处理POST以外的不支持的method, HTTP 501返回
				
				// TODO: [2]获取POST请求字符串参数
				String paramStr;
				
				// TODO: [3]将字符串参数转换为参数模型
				JSONObject param = Converter.str2json(paramStr);
				
				// TODO: [4]将参数模型传递给内部处理对象处理
				Message message;
				JSONObject result = message.request(param);
				
				// TODO: [5]返回结果模型
				String resultStr = Converter.json2str(result);
				
				// TODO: [6]将返回结果模型转换为字符串，HTTP返回
			} catch (Exception e) {
				// TODO: [7]内部异常，HTTP 500返回
			}
		}
	}

	public static void main(String[] args) throws Exception {
		// 创建Servlet Server，监听指定端口
		// TODO
		
		// 设置HTTP请求处理Handler
		// TODO
		
		// 启动服务
		// TODO
		
		// 等待退出
		// TODO
	}
}
```

对于[1]没什么好说的了，待实现。

对于[2]和[6]需要的字符串，我们要定义输入和输出字符串的的协议，这里，我们定义输入和返回的协议格式为标准json字符串，具体业务规格后续体现。

对于[3]和[5]涉及的参数和返回模型类，我们直接使用org.json.JSONObject，当然具体定义输入和返回的模型类，效率会更好，这里忽略。

但是，我们需要一个转换模块，用于在字符串参数、字符串结果和JSONObject之间做转换，我们我们命名整个转换模块为Converter，他有两个静态函数str2json和json2str来实现转换。

对于[4]，这个内部处理对象应该输入JSONObject参数，处理之后，返回一个JSONObject结果，整个内部对象将处理来自前端的所有请求，这里命名为Message，具备一个request函数。

再考虑异常捕获[7]，对于期望的异常，我们应该各个角度具体实现的时候捕获他，而对于不期望的异常，意味着内部错误，比如mysqldown了，或者是一个bug，但是我们不能把这些异常抛给前端用户，因此，合适的做法在最外层捕获，同时做好日志记录（包括详细的调用栈信息），返回HTTP 500。

### Converter

ServletServer的设计暂时到此，接下来，我们要明确把Convert的定义出来：
```bash
/**
 * 处理字符串和json之间的转换
 */
public class Converter {
	/**
	 * 字符串转json
	 * 
	 * @param str 字符串
	 * @return {json对象, {"status": "<错误状态>", "message": "<错误描述>"}}
	 */
	public static Object[] str2json(String str) {
		// TODO
	}

	/**
	 * json转字符串
	 * 
	 * @param json json对象
	 * @return 字符串
	 */
	public static String json2str(JSONObject obj) {
		// TODO
	}
}

```

### Message

轮到Message模块了，最初的Message对象是：
```java
/**
 * 消息类
 * <p>消息类用于处理用户的消息请求，包括添加、查询、修改和删除操作
 */
public class Message {
	/**
	 * 处理请求
	 * 
	 * @param param {"type", "<type值>", ...}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": <结果>}
	 * @throws Exception 错误异常
	 */
    public JSONObject request(JSONObject param) throws Exception {
		// TODO
	}
}
```

接下来的问题是，我们要怎么实现request，hz-message使用mysql管理后台数据，那我们这个Message实现的时候应该需要mysql相关的连接对象和数据表的操作等。但是，也许我们要提前考虑一些扩展性的事，也许我们以后想换成mongodb来实现，这像是一个典型的多态设计，既然是多态，我们要把数据访问逻辑抽象化，我们不希望以后把mysql换成mongodb时，还要修改Message，颠覆Message的测试代码。

我们先简单定义一下mysql数据库名称（message）和业务唯一相关的消息表（post）。

考虑后续我们也许会给hz-message加上用户的概念，我们最好是把数据库的连接和消息的处理逻辑分为两个模块：
* Holder，抽象数据库连接
* Post，抽象消息表的操作

Holder具备一个inst接口，用于返回一个Object对象，对于mysql，他实际是一个mysql连接对象。

Post根据功能业务，定义getCount、getAll、getById、add、modify、remove接口。

至此，Message应该看上去是这样的：
```java
/**
 * 消息类
 * <p>
 * 消息类用于处理用户的消息请求，包括添加、查询、修改和删除操作
 */
public class Message {
	public Message(Holder holder, Post post) {
		// TODO
	}

	/**
	 * 处理请求
	 * 
	 * @param param {"type", "<type值>", ...}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": <结果>}
	 * @throws Exception 错误异常
	 */
    public JSONObject request(JSONObject param) throws Exception {
		String type = param["type"];
	
		// TODO: 根据不同的type值来分发调用Post的对应接口
	}
}
```

### Checker

等等，也许param不存`type`，或者`type`的值根本是String，我必须正确处理，最好我们需要一个具体的参数校验模块Checker，而且这块Checker将不仅仅是校验`type`参数。
```java
/**
 * 校验请求参数是否合法
 */
public class Checker {
	/**
	 * 校验type参数是否合法
	 * 
	 * @param param {"type": "<type值>", ...}
	 * @return {"<type值>", {"status": "<错误状态>", "message", "<错误描述>:}}
	 */
	public static Object[] checkParamType(JSONObject param) {
		// TODO: 考虑各种type异常值的处理
	}
}
```

我们干脆把各种type值直接定义了，这时Message将看起来是这样的：
```java
/**
 * 消息类
 * <p>
 * 消息类用于处理用户的消息请求，包括添加、查询、修改和删除操作
 */
public class Message {
	public Message(Holder holder, Post post) {
		// TODO:
	}

	/**
	 * 处理请求
	 * 
	 * @param param {"type", "<type值>", ...}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": <结果>}
	 * @throws Exception 错误异常
	 */
	public JSONObject request(JSONObject param) throws Exception {
		// 校验type参数
		Object[] typeErr = Checker.checkParamType(param);
		String type = (String) typeErr[0];
		
		// 根据不同type值调用post对象处理
		if (type.equals("get_post_count")) {
			return _post.getCount();
		} else if (type.equals("get_all_post")) {
			return _post.getAll();
		} else if (type.equals("get_post_by_id")) {
			return _post.getById(param);
		} else if (type.equals("add_post")) {
			return _post.add(param);
		} else if (type.equals("modify_post")) {
			return _post.modify(param);
		} else if (type.equals("remove_post")) {
			return _post.remove(param);
		} else {
			return new JSONObject("{\"status\": \"invalid_parameter\", \"message\": \"`type` is invalid.\"}");
		}
	}
}
```

另外一个问题是，Message对象的创建过程，需要实例化具体的Holder和Post，考虑mysql和mongodb两种实现方式，显示创建Message过程是一个典型工厂模式，我们将需要一个MessageCreator，MessageCreator的设计将涉及具体的Holder和Post的实现，我们接下来先深入Holder、MysqlHolder和Post、MysqlPost。

### Holder

Java是相对其他脚本语言（js、php、python等）的一种强类型语言，对于强类型语言的多态，我们需要抽象类，Holder来定义了Holder需要的接口：
```java
/**
 * 实体类接口
 */
public interface Holder {
	/**
	 * 获取实体内部对象
	 * 
	 * @return 实体内部对象
	 */
	public Object inst();
	
	/**
	 * 销毁数据实体
	 * 
	 * @throws Exception 错误异常
	 */
	public void destroy() throws Exception;
	
	/**
	 * 关闭数据实体
	 * 
	 * @throws Exception 错误异常
	 */
	public void close() throws Exception;
}
```
其中，inst的返回，对于mysql来说就是一个mysql连接对象，但是对于mongodb来说则是mongodb的连接对象，我们直接用Object抽象。

### Post

同样的，Holder来定义了Holder需要的接口：
```java

/**
 * 信息类接口
 */
public interface Post {
	/**
	 * 获取post个数
	 * 
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": {"count": <post个数>}}
	 * @throws Exception 错误异常
	 */
	public JSONObject getCount() throws Exception;

	/**
	 * 获取所有的post
	 * 
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": [{"id": <post ID>, "timestamp": <post时间戳，单位：s>, "content": "<post内容>"}, ...]}
	 * @throws Exception 错误异常
	 */
	public JSONObject getAll() throws Exception;

	/**
	 * 根据ID获取post
	 * 
	 * @param param {"id": <post ID>}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": {"id": <post ID>, "timestamp": <post时间戳，单位：s>, "content": "<post内容>"}}
	 * @throws Exception 错误异常
	 */
	public JSONObject getById(JSONObject param) throws Exception;

	/**
	 * 添加post
	 * 
	 * @param param {"content": "<post内容>"}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": {"id": <post ID>, "timestamp": <post时间戳，单位：s>, "content": "<post内容>"}}
	 * @throws Exception 错误异常
	 */
	public JSONObject add(JSONObject param) throws Exception;

	/**
	 * 修改post
	 * 
	 * @param param {"id": <post ID>, "content": "<post内容>"}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": {"id": <post ID>, "timestamp": <post时间戳，单位：s>, "content": "<post内容>"}}
	 * @throws Exception 错误异常
	 */
	public JSONObject modify(JSONObject param) throws Exception;

	/**
	 * 删除post
	 * 
	 * @param param {"id": <post ID>}
	 * @return {"status": "<状态>", "message": "<错误描述>", "data": {"id": <post ID>}}
	 * @throws Exception 错误异常
	 */
	public JSONObject remove(JSONObject param) throws Exception;
}
```

### MysqlHolder

接下来，我们来具体说MysqlHolder如何遵循Holder接口规范来实现Mysql版Holder：
```java

/**
 * mysql实体类
 * <p>
 * mysql实体类实现mysql连接和关闭，及数据库创建和销毁
 */
public class MysqlHolder implements Holder {
	/** 数据库连接 */
	private Connection _conn;
	/** 数据库名 */
	private String _database;
	
	/**
	 * 构造函数
	 * 
	 * @param user mysql用户名
	 * @param password mysql用户密码
	 * @param database mysql数据库名
	 * @throws Exception 错误异常
	 */
	public MysqlHolder(String user, String password, String database) throws Exception {
		// 定义数据库接入类和连接地址
		final String driver = "com.mysql.cj.jdbc.Driver";
		final String url = "jdbc:mysql://localhost:3306/?characterEncoding=utf8&useSSL=false";
		
		// 连接数据库
		Class.forName(driver);
		_conn = DriverManager.getConnection(url, user, password);
		
		// 保存数据库名
		_database = database;
		
		// 创建数据库
		Statement statement = _conn.createStatement();
		statement.execute(String.format("CREATE DATABASE IF NOT EXISTS `%s` default character set utf8 COLLATE utf8_general_ci", _database));
		statement.execute(String.format("USE `%s`", _database));
		statement.close();
	}
	
	@Override
	public Object inst() {
		return _conn;
	}
	
	@Override
	public void destroy() throws Exception {
		Statement statement = _conn.createStatement();
		statement.execute(String.format("DROP DATABASE IF EXISTS `%s`", _database));
		statement.close();
	}
	
	@Override
	public void close() throws Exception {
		_conn.close();
	}
}
```

其中的构造函数是MysqlHolder需要注意设计的，把数据库连接需要的user和password及database作为参数传递过来是更好的设计，我们期望数据库user、password和database再整个应用中仅保留一份，显然不是这里，而且测试相关的数据库信息和正式的一定不一样，更说明这样设计的必要性。

说说这里边的注释，对于重载的函数，就不用再注释函数了，因为接口类已经详细注释了，必要的注释应该体现在函数内部的实现。

### MysqlPost

```java

/**
 * mysql信息类
 * <p>
 * mysql信息类实现对数据库表post的创建、添加、查询、修改和删除
 */
public class MysqlPost implements Post {
	/** 数据库表名 */
	private final String _table = "post";

	/** 数据库连接 */
	private Connection _conn;
	
	/**
	 * 构造函数
	 * 
	 * @param conn 数据库连接
	 * @throws Exception 错误异常
	 */
	public MysqlPost(Connection conn) throws Exception {
		// 保存数据库连接
		_conn = conn;
		
		// 创建数据库表
		Statement statement = _conn.createStatement();
		statement.execute(String.format("CREATE TABLE IF NOT EXISTS `%s` (`id` INT NOT NULL PRIMARY KEY AUTO_INCREMENT, `timestamp` INT NOT NULL, `content` TEXT NOT NULL)", _table));
		statement.close();
	}
	
	@Override
	public JSONObject getCount() throws Exception {
		// 查询数据库
		Statement statement = _conn.createStatement();
		ResultSet resultSet = statement.executeQuery(String.format("SELECT COUNT(`id`) AS count FROM `%s`", _table));
		resultSet.next();
		int count = resultSet.getInt("count");
		statement.close();
		String resultStr = String.format("{\"status\": \"ok\", \"data\": {\"count\": %d}}", count);
		return new JSONObject(resultStr);
	}
	
	@Override
	public JSONObject getAll() throws Exception {
		// 查询数据库
		Statement statement = _conn.createStatement();
		ResultSet resultSet = statement.executeQuery(String.format("SELECT `id`,`timestamp`,`content` FROM `%s` ORDER BY `id` DESC", _table));
		String cellStrs = "";
		while (resultSet.next()) {
			int id = resultSet.getInt("id");
			int timestamp = resultSet.getInt("timestamp");
			String content = resultSet.getString("content");
			String cellStr = String.format("{\"id\": %d, \"timestamp\": %d, \"content\": \"%s\"}", id, timestamp, content);
			cellStrs += (cellStrs.isEmpty() ? "" : ", ") + cellStr;
		}
		statement.close();
		String resultStr = String.format("{\"status\": \"ok\", \"data\": [%s]}", cellStrs);
		return new JSONObject(resultStr);
	}

	@Override
    public JSONObject getById(JSONObject param) throws Exception {
		// 校验id参数
		Object[] idErr = Checker.checkParamId(param);
		int id = (Integer) idErr[0];
		JSONObject err = (JSONObject) idErr[1];
		if (err != null) {
			return err;
		}
		
		// 查询数据库，如果没有数据返回错误
		Statement statement = _conn.createStatement();
		ResultSet resultSet = statement.executeQuery(String.format("SELECT `id`,`timestamp`,`content` FROM `%s` WHERE `id`=%d LIMIT 1", _table, id));
		if (!resultSet.next()) {
			statement.close();
			return new JSONObject("{\"status\": \"none_target\", \"message\": \"`id` does not exist.\"}");
		}
		int timestamp = resultSet.getInt("timestamp");
		String content = resultSet.getString("content");
		statement.close();
		String resultStr = String.format("{\"status\": \"ok\", \"data\": {\"id\": %d, \"timestamp\": %d, \"content\": \"%s\"}}", id, timestamp, content);
		return new JSONObject(resultStr);
	}

	@Override
    public JSONObject add(JSONObject param) throws Exception {
		// 校验content参数
		Object[] contentErr = Checker.checkParamContent(param);
		String content = (String) contentErr[0];
		JSONObject err = (JSONObject) contentErr[1];
		if (err != null) {
			return err;
		}

		// 获取当前系统时间
		int timestamp = (int) (new Date().getTime() / 1000);

		// 查询数据库
		Statement statement = _conn.createStatement();
		statement.executeUpdate(String.format("INSERT INTO `%s` SET `timestamp`=%d,`content`=\"%s\"", _table, timestamp, content), Statement.RETURN_GENERATED_KEYS);
		ResultSet resultSet = statement.getGeneratedKeys();
		resultSet.next();
		int id = resultSet.getInt(1);
		statement.close();
		String resultStr = String.format("{\"status\": \"ok\", \"data\": {\"id\": %d, \"timestamp\": %d, \"content\": \"%s\"}}", id, timestamp, content);
		return new JSONObject(resultStr);
	}

	@Override
    public JSONObject modify(JSONObject param) throws Exception {
		// 校验id参数
		Object[] idErr = Checker.checkParamId(param);
		int id = (Integer) idErr[0];
		JSONObject err = (JSONObject) idErr[1];
		if (err != null) {
			return err;
		}

		// 校验content参数
		Object[] contentErr = Checker.checkParamContent(param);
		String content = (String) contentErr[0];
		err = (JSONObject) contentErr[1];
		if (err != null) {
			return err;
		}
		
		// 查询数据库，如果没有数据修改返回错误
		Statement statement = _conn.createStatement();
		int rowCount = statement.executeUpdate(String.format("UPDATE `%s` SET `content`=\"%s\" WHERE `id`=%d", _table, content, id));
		statement.close();
		if (rowCount != 1) {
			return new JSONObject("{\"status\": \"none_target\", \"message\": \"`id` does not exist.\"}");
		}
		return getById(new JSONObject(String.format("{\"id\": %d}", id)));
	}

	@Override
    public JSONObject remove(JSONObject param) throws Exception {
		// 校验id参数
		Object[] idErr = Checker.checkParamId(param);
		int id = (Integer) idErr[0];
		JSONObject err = (JSONObject) idErr[1];
		if (err != null) {
			return err;
		}
		
		// 查询数据库，如果没有数据修改返回错误
		Statement statement = _conn.createStatement();
		int rowCount = statement.executeUpdate(String.format("DELETE FROM `%s` WHERE `id`=%d", _table, id));
		statement.close();
		if (rowCount != 1) {
			return new JSONObject("{\"status\": \"none_target\", \"message\": \"`id` does not exist.\"}");
		}
		String resultStr = String.format("{\"status\": \"ok\", \"data\": {\"id\": %d}}", id);
		return new JSONObject(resultStr);
	}
}
```

MysqlPost实际上是对应mysql的post表，既然Holder我们已经定义了，MysqlPost应当仅关注post相关的实现，数据库的连接对象将作为参数传入。

前面说到，数据库的参数应该传入，对于MysqlPost，post的表名直接定义常量即可，不管是正式还是测试版，这个名称没有必要改变。

对于函数内注释，我们应该有必要的注释，但是没有必要逐行注释，原则上，函数内如逻辑上分为多个功能块，我么应该对各个功能块注释，格式化的角度，各功能块换行。

### MessageCreator

现在改说MessageCreator，MessageCreator模式上是工厂，使用者需要明确告诉MessageCeator是想创建mysql的Message还是其他。

这里衍生一下，为什么不是抽象工厂模式。抽象工厂模式的应用场景对于hz-message应该是，hz-message运行过程中，需要动态切换mysql message和moongodb message，显示对于hz-message没有这样的需求，所有不是抽象工厂模式。

```java

/**
 * message对象创建类
 */
public class MessageCreator {
	/**
	 * 创建mysql类message
	 * 
	 * @param user mysql用户名
	 * @param password mysql用户密码
	 * @param database mysql数据库名
	 * @return message对象
	 * @throws Exception 错误异常
	 */
	public static Message createMessageByMysql(String user, String password, String database) throws Exception {
		Holder holder = new MysqlHolder(user, password, database);
		Post post = new MysqlPost((Connection) holder.inst());
		return new Message(holder, post);
	}
}
```

## Server测试

到此，我们把hz-messag的设计多做的七七八八了，有些都顺手实现了，接下来，我们在具体功能实现的时候，应该并行单元测试了。

单元测试和QA做的事是两回事，单元测试是开发分内的事。

单元测试不仅确保每个模块的功能实现正确没有bug，同时还检验整个工程的架构的设计是否合理，如果单元测试的覆盖做不到每个功能的全面覆盖及唯一覆盖，那我们应该反过来具体检查设计是否合理，具体的，模块切分是否合理，模块间是否功能交叉，模块的接口设计是否具备好的扩展性。

单元测试的覆盖分两个维度：
* 是否都覆盖
* 是否存在交叉覆盖

不想设计自顶而下，单元测试则是自下而上的实现。

### ConverteTest

```java
/**
 * 测试Converter
 */
public class ConverterTest {
	@Test
	public void test_str2json() {
		// 测试输入空的参数（null和""），返回期望值
		// TODO
		
		// 测试输入非法参数（非json字符串），返回期望值
		// TODO
		
		// 测试输入合法参数（json字符串），返回期望值
		// TODO
	}
	
	@Test
	public void test_json2str() {
		// 测试输入空参数（null），返回期望值
		// TODO

		// 测试输入合法参数，返回期望值
		// TODO
	}
}
```

### CheckerTest

```java

/**
 * 测试Checker
 */
public class CheckerTest {
	@Test
	public void test_checkParamType() {
		// 测试空参数
		Object[] res = Checker.checkParamType(new JSONObject("{}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		
		// 测试非法key参数
		res = Checker.checkParamType(new JSONObject("{\"abcd\": \"1234\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		res = Checker.checkParamType(new JSONObject("{\"Type\": \"1234\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		
		// 测试非法value参数
		res = Checker.checkParamType(new JSONObject("{\"type\": 1234}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("invalid_parameter"));
		res = Checker.checkParamType(new JSONObject("{\"type\": \"\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("invalid_parameter"));
		
		// 测试合法参数
		res = Checker.checkParamType(new JSONObject("{\"type\": \"1234\"}"));
		assertTrue(res[0].equals("1234") && res[1] == null);
	}
	
	@Test
	public void test_checkParamId() {
		// 测试空参数
		Object[] res = Checker.checkParamId(new JSONObject("{}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		
		// 测试非法key参数
		res = Checker.checkParamId(new JSONObject("{\"abcd\": 1234}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		res = Checker.checkParamId(new JSONObject("{\"Id\": 1234}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));

		// 测试非法value参数
		res = Checker.checkParamId(new JSONObject("{\"id\": \"1234\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("invalid_parameter"));
		
		// 测试合法参数
		res = Checker.checkParamId(new JSONObject("{\"id\": 1234}"));
		assertTrue(res[0].equals(1234) && res[1] == null);
	}
	
	@Test
	public void test_checkParamContent() {
		// 测试空参数
		Object[] res = Checker.checkParamContent(new JSONObject("{}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));

		// 测试非法key参数
		res = Checker.checkParamContent(new JSONObject("{\"abcd\": \"1234\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));
		res = Checker.checkParamContent(new JSONObject("{\"Content\": \"1234\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("lost_parameter"));

		// 测试非法value参数
		res = Checker.checkParamContent(new JSONObject("{\"content\": 1234}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("invalid_parameter"));
		res = Checker.checkParamContent(new JSONObject("{\"content\": \"\"}"));
		assertTrue(res[0] == null && ((JSONObject) res[1]).getString("status").equals("invalid_parameter"));

		// 测试合法参数
		res = Checker.checkParamContent(new JSONObject("{\"content\": \"1234\"}"));
		assertTrue(res[0].equals("1234") && res[1] == null);
	}
}
```

### MysqlHolderTest

我们测试的主要是接口实现，所有HolderTest不用测试了。

测试，原则上我们仅覆盖接口，或者说是public方法。

Java单元测试，我们使用org.junit。

MysqlHolderTest的框架看上去是这样的：
```java
/**
 * 测试MysqlHolder
 */
public class MysqlHolderTest {
	@Test
	public void test_inst() throws Exception {
		// 创建MysqlHolder对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO
	
		// 测试MysqlHolder.inst返回的对象是否为mysql连接对象
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}
	
	@Test
	public void test_destroy() throws Exception {		
		// 创建MysqlHolder对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

		// 获取数据库连接（MysqlHolder.inst()），用于访问数据库验证detroy方法是否正确执行
		// TODO

		// 测试数据库没有销毁，通过数据连接conn直接查询数据库，比如"SHOW TABLES"，看有没有异常
		// TODO

		// 执行MysqlHolder.destroy
		// TODO
		
		// 测试数据库已经销毁，通过数据连接conn直接查询数据库，比如"SHOW TABLES"，看有没有异常
		// TODO
	}
	
	@Test
	public void test_close() throws Exception {
		// 创建MysqlHolder对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

		// 测试数据库连接没有关闭
		// TODO
		
		// 执行MysqlHolder.close
		// TODO
		
		// 测试数据库连接已经关闭
		// TODO
	}
}
```

### MysqlPostTest

MysqlPostTest用来测试MysqlPost的接口函数，看起来是这样的。

```java

/**
 * 测试MysqlPost类
 */
public class MysqlPostTest {
	@Test
    public void test_getCount() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO
	
		// 执行MysqlPost.getCount()，测试初始post个数为0
		// TODO
		
		// 添加一组测试post
		// TODO
		
		// 执行MysqlPost.getCount()，测试post个数为添加的post个数
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}

	@Test
    public void test_getAll() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

		// 执行MysqlPost.getAll()，测试初始的post为空
		// TODO

		// 添加一组测试post
		// TODO

		// 执行MysqlPost.getAll()
		// 测试获取的post数组：content和添加的post数组一致，但是时间顺序依次递减，且和添加的post数组顺序相反
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}

	@Test
    public void test_getById() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

        // 指定任意一个post id执行MysqlPost.getById()，测试返回期望的错误码
		// TODO
		
        // 添加一组测试post，并获取所有的post（包含post id）
		// TODO

        // 枚举每一个post，获取post id，分别执行MysqlPost.getById()，测试返回的post数据和枚举的post一致
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}
	
	@Test	
    public void test_add() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

        // 执行MysqlPost.add()，添加一个测试post
		// 测试返回的状态ok
		// 测试返回的id合法
		// 测试返回的timestamp合法
		// 测试返回的content和添加的一致
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}
	
	@Test
    public void test_modify() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TODO

        // 添加一个测试post，并获取id和timestamp
		// TODO

        // 根据获取的id，执行MysqlPost.modify()
		// 测试返回的status为ok
		// 测试后返回的timestamp和添加的timestamp一致
		// 测试返回的content正确
		// TODO
		
		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}

	@Test
    public void test_remove() throws Exception {
		// 创建MysqlHolder对象
		// 创建MysqlPost对象
		// 注意：要使用测试数据库，和线网数据库分开
		// TOD
		
		// 指定任意一个id，执行MysqlPost.remove()
		// 测试返回的状态none_target
		// TODO

        // 添加一个测试post，获取post id
		// TODO

		// 获取添加的post id
		// TODO

		// 根据获取的post id，执行MysqlPost.remove()
		// 测试返回的状态ok
		// 测试返回的id和删除的id一致
		// TODO

		// 销毁MysqlHolder，确保测试数据库销毁，不影响其他测试
		// TODO
	}
}
```

### MessageTest

```java
/**
 * 测试Message
 */
public class MessageTest {
	@Test
    public void test_request() throws Exception {
		// 创建Message对象
		// 注意：创建Messae对象需要的Hodler和Post对象，可以本地模拟Holder和Post对相关，不用使用MysqlHolder和MysqlPost，MysqlHolder和MysqlPost有自己单独的测试程序
		// TODO
		
        // 执行Message.request()
		// 测试输入合法参数{"type": "get_post_count"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "get_all_post"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "get_post_by_id"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "add_post"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "modify_post"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "remove_post"}，返回期望值
		// TODO
        
        // 执行Message.request()
        // 测试输入合法参数{"type": "other"}，返回期望值
		// TODO
	}
}
```

### MessageCreatorTest

```java
/**
 * 测试MessageCreator
 */
public class MessageCreatorTest {
	@Test
	public void test_createMessageByMysql() throws Exception {
		// MessageCreator.createMessageByMysql，创建Message对象
		// 测试返回的Message对象是否为空
		// TODO
	}
}
```

# 后记

大家可以在这里[hz-message](https://github.com/wuqingtao/hz-message)找到整个工程的实现版本，其中包含了另外的node、php和python版本，也包含了完整的部署、启停脚本。
