---
title: hz-message构建过程
date: 2016-12-05 21:39:35
tags:
---

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
