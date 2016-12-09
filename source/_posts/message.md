---
title: hz-message构建过程
date: 2016-12-05 21:39:35
tags:
---

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

本文将以一个简单的DEMO项目hz-message为例，逐步描述上面说的架构历程。

[hz-message](https://github.com/wuqingtao/hz-message)是一个演示项目，以多语言的方式展示了一个简单动态Web项目的设计、开发、测试和部署过程，功能类似于一个心情记事本，包括消息展示、发布和获取。

# 工程目录

## 工程名称

工程名称将是整个项目生命周期最多被引用到的名词，所以多花点时间琢磨工程名称很要必要。

合适的工程名称应该满足：
* 简洁，不要过于冗长。
* 又不能太过简洁，能够大致描述项目目的。
* 不要用拼音或简拼。
* 也不要过分纠结。

由于该项目是一个简单的消息发布和浏览应用，messag是一个不错的选择，当然msg也不错，但是message还是太泛，我们再给他加点限制，message of hongzhong，也就是hz-message，当然hz-msg也不错，至于是-还是_，都可以。
```bash
hz-message
```

## 目录结构

最终所有项目的源文件都讲位于各个目录中，各个文件之间互相依赖调用，我们不希望在开发过程中发现目录结构或是目录名称定义不合理再来修改，而致代码日志混乱。

目录结构的定义应该满足以下原则：
* 各个子系统分开，如前端（web）和后端（server）。
* 不同的组件分开，如源码（src）、测试（test）、配置（conf）和库目录（libs）。
* 不同类型归类，如js、css，utils等。
* 对于和src和test，应该体现层级域名的概念。

目录名称的命名应该：
* 遵循惯例，如html、js、css、src、test、conf、libs、utils、bin、obj、tmp等。
* 惯例以外的名称不要过分简洁，亦不要太臃肿。
* 忌拼音或简拼，很容易不知道是什么意思。

对于hz-message，对应连个子系统，前端和后端。

前端我们命令为web，www看上去更像是一个部署目录，html不足以涵盖前端的概念。当然，在这里，更好的做法应该是app/web，app意味前端，而前端可能会有web，也可能有android和ios。
```bash
hz-message
└── web
```
web端具备连个组件src和test。src分为html、js和css不同类型，hz-message把html文件直接至于src目录下也OK，毕竟js和css都归html调用。对于test，也应该分为html、js和css，hz-message虽然仅做了部分js相关的测试，考虑后续也许会做html相关测试，所以仅有js目录。
```bash
hz-message
└── web
    ├── src
    │   ├── css
    │   └── js
    └── test
        └── js
```

后端命名为server，srv好像也不错，不过我个人觉得太缩写。
```bash
hz-message
└── server
```

对于后端，由于hz-message目的是技术演示，针对目前常用服务端语言，实现了四个版本（java、node、php和python），对于每种语言，又分为src、test、conf和libs，conf和libs最好不要放在src下面，因为test中测试程序也可能调用conf和libs，同时也方便后续编译和打包。对于，src和test，我们增加层级域名的结构hz/message，类似于java包结构。
```bash
hz-message
├── server
│   ├── java
│   │   ├── conf
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── node
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── php
│   │   ├── conf
│   │   ├── libs
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   └── python
│       ├── src
│       │   └── hz
│       │       └── message
│       └── test
│           └── hz
│               └── message
```
最好完成的目录结构：
```bash
hz-message
├── server
│   ├── java
│   │   ├── conf
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── node
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── php
│   │   ├── conf
│   │   ├── libs
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   └── python
│       ├── src
│       │   └── hz
│       │       └── message
│       └── test
│           └── hz
│               └── message
└── web
    ├── src
    │   ├── css
    │   └── js
    └── test
        └── js
```

## 目录结构

OK，工程名和目录都搭建好了，我们应该及早托管，因为可能后头进一步的设计我们是分工完成的，需要代码托管平台介入了。

hz-message选择GitHub。大家可以提前点击[hz-message](https://github.com/wuqingtao/hz-message)查看代码。

# 模块切分

我们先说server，从java服务开始。

## java服务

既然是服务，我们首先需要一个模块来实现服务入口。结构的搭建是hz-message演示的重点，所以我们不会使用tomcat来帮忙，当然也不会重新从底层开始写一个服务。Servt是java服务最合适的标准，我们使用Jetty嵌入式组件来实现hz-message java服务。

我们把这个模块命名为ServletServer，直接命名为Server也没问题，不过我想明确他是一个Servlet服务，而不是facgi服务。
```bash
hz-message
└── server
    └── java
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
 * <p>
 * 基于Jetty嵌入式功能实现
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

	public static void main(String[] args) throws Exception {
		// 创建Server，监听指定端口
		Server server = new Server(8401);
		
		// 设置HTTP请求处理Handler
		server.setHandler(new Handler());
		
		// 启动服务
		server.start();
		
		// 等待退出
		server.join();
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
* 最后我们也要处理好各种异常

对于ServletServer，我们进一步演化：
```java
/**
 * Servlet服务
 * <p>
 * 基于Jetty嵌入式功能实现
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
			// TODO: [1]处理POST以外的不支持的HTTP method
			
			// TODO: [2]获取POST请求字符串参数
			
			// TODO: [3]将字符串参数转换为参数模型
			
			// TODO: [4]将参数模型传递给内部处理对象处理
			
			// TODO: [5]返回结果模型
			
			// TODO: [6]将返回结果模型转换为字符串，HTTP返回
		}
	}

	public static void main(String[] args) throws Exception {
		// 创建Server，监听指定端口
		Server server = new Server(8401);
		
		// 设置HTTP请求处理Handler
		server.setHandler(new Handler());
		
		// 启动服务
		server.start();
		
		// 等待退出
		server.join();
	}
}
```

对于[1]没什么好说的了，待实现。

对于[2]和[6]需要的字符串，我们要定义输入和输出字符串的的协议，这里，我们定义输入和返回的协议格式为标准json字符串，具体业务规格后续体现。

对于[3]和[5]涉及的参数和返回模型类，我们直接使用org.json.JSONObject，当然具体定义输入和返回的模型类，效率会更好，这里忽略。

但是，我们需要一个转换模块，用于在字符串参数、字符串结果和JSONObject之间做转换，我们我们命名整个转换模块为Converter，他有两个静态函数str2json和json2str来实现转换。

对于[4]，这个内部处理对象应该输入JSONObject参数，处理之后，返回一个JSONObject结果，整个内部对象将处理来自前端的所有请求，这里命名为Message，具备一个request函数。

再考虑异常捕获，ServletServer变为：
```java
/**
 * Servlet服务
 * <p>
 * 基于Jetty嵌入式功能实现
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
		// 创建Server，监听指定端口
		Server server = new Server(8401);
		
		// 设置HTTP请求处理Handler
		server.setHandler(new Handler());
		
		// 启动服务
		server.start();
		
		// 等待退出
		server.join();
	}
}
```

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
		// TODO:
	}

	/**
	 * json转字符串
	 * 
	 * @param json json对象
	 * @return 字符串
	 */
	public static String json2str(JSONObject obj) {
		// TODO:
	}
}

```
作为敏捷的原则之一，我们希望文档以标准的注释方式嵌入代码。

### Message

轮到Message模块了，最初的Message对象是：
```java
/**
 * 消息类
 * <p>
 * 消息类用于处理用户的消息请求，包括添加、查询、修改和删除操作
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
		// TODO:
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

另外一个问题是，Message对象的创建过程，需要实例化具体的Holder和Post，考虑mysql和mongodb两种实现方式，显示创建Message过程是一个典型工厂模式，我们将需要一个MessageCreator，MessageCreator的设计将涉及具体的Holder和Post的实现，我们接下来先深入MysqlHolder和MysqlPost。

### MysqlHolder






















***
***
***
***
***
***















本文将以一个简单的hz-message



一个项目的架构将左右


[hz-message](https://github.com/wuqingtao/hz-message)是一个个人演示项目，以多语言的方式展示了一个简单的动态Web项目的设计、开发、测试和部署过程。

功能类似于一个心情记事本，包括消息展示、发布和获取，点击[hz-message主页](http://120.55.185.245/message/)就可以完全了解其功能。

本文不会讨论或涉及任何框架（如java spring、python django、php ci、node.js express等）的使用，除了必要的编译和单元测试框架，也不会研究具体功能的实现（如怎么在服务端访问mysql），分布式、并发和缓存的概念也不会体现，其目标将围绕着如何做架构、规范和单元测试展开，再给出一些总结。

尽管规范、架构及测试的概念跟语言无关，但是作为演示，整个服务端还是分别以java、python、node.js和php实现了四个版本。接下来的部分，我们将分别以这四个版本为例描述我对规范、架构和测试的理解。

# 团队分工

假设整个项目的开发团队组成为：
* 设计者。负责实现代码管理、架构、接口协议指定。
* java开发者。负责实现设计者指定的各个模块的函数。
* node.js开发者。负责实现设计者指定的各个模块的函数。
* python开发者。负责实现设计者指定的各个模块的函数。
* php开发者。负责实现设计者指定的各个模块的函数。

# 目录结构

为了快速推进项目进入开发阶段，开发者需者设计者告诉他要实现的功能模块的代码在哪儿，于是，设计者首先要建立代码目录结构。

当然，实际开发工程中，我们都习惯了开发框架（如tomcat、php ci等）为我们做的一切准备工作，不用操心目录管理，但是合理的目录的设计也是架构能力的总要组成。

下面是整个项目的代码目录结构：
```bash
hz-message
├── server
│   ├── java
│   │   ├── conf
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── node
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   ├── php
│   │   ├── conf
│   │   ├── libs
│   │   ├── src
│   │   │   └── hz
│   │   │       └── message
│   │   └── test
│   │       └── hz
│   │           └── message
│   └── python
│       ├── src
│       │   └── hz
│       │       └── message
│       │           └── utils
│       └── test
│           └── hz
│               └── message
│                   └── utils
└── web
    ├── src
    │   ├── css
    │   └── js
    └── test
        └── js
```

建立整个目录结构的先后原则：
* 整个工程有一个目录名。如hz-message，这个是一个顶级目录，需要首先确定。
* 后端（server）和前端（web）目录分开。如server、web，其他的所有逻辑目录都将分属于后端和前端。
* 不同语言的后端目录分开。如java、node、php和python，对于后端，应该首先按语言区分，因为任何后端的逻辑目录都将分属于某个语言。
* 不同的语言的后端，配置、依赖库、源码和测试目录，方便后续打包和部署，不同的语言限于其功能和不同的包管理机制，包含的目录不同。
	* java后端，如src、test。
	* node后端，如src、test。
	* php后端，如conf、libs、src、test。
	* python后端，如src、test。
* 前端的源码和测试目录分开，如src、test。对于该项目，前端只有web，所以server和web是平级目录，如果前端还包括Android端和IOS端，我们应该在web目录上加一层目录，如app，然后在app目录下建web、android和ios目录。

# 代码管理

目录结构确定后，设计者需要将整个工程目录做代码托管，该项目简单的选择GitHub作为托管平台，这里不探讨GitHub和git命令的使用。

# java服务

nginx和后端java服务是通过反向代理实现的，java服务作为第一个模块，需要实现服务，这里采用常用的Servlet实现（这里不讨论Servlet和fcgi的区别）。





*./hz-message/server/java/hz/message/ServletServer.java*
```java
package hz.message;

/**
 * Servlet服务
 * <p>
 * 基于Jetty嵌入式功能实现
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
				// 处理HTTP请求
                // TODO:
            }
        }
    }

    public static void main(String[] args) throws Exception {
        // 解析命令行端口参数，
        // TODO:
        
        // 根据指定端口，创建Server，启动Server
		// TODO:
    }
}
```







### 架构设计

### 单元测试

### 代码规范


## python服务

## php服务

## node.js服务

# 前端
