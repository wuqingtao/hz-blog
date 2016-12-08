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

OK，工程名和目录都搭建好了，我们应该及早托管，因为可能后头进一步的设计我们是分工完成的，需要代码托管平台介入了，

hz-message选择GitHub。大家可以提前点击[hz-message](https://github.com/wuqingtao/hz-message)查看代码。





















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
