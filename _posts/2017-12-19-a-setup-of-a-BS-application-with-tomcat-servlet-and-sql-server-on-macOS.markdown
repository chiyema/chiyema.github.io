---
layout: post
title: 利用 Tomcat，Servlet 和 SQL Server 在 macOS 上搭建简易 B/S 应用
subtitle: 'a setup of a B/S application with Tomcat, Servlet and SQL Server on macOS'
date: 2017-12-19T00:00:00.000Z
author: Moubei
tags:
  - Tomcat
  - SQL Server
  - MacOS
style: summer
---

中间件对于物联网的重要性无需多言，通过这个简单的搭建应用可以体会中间件连接前后端的起到的作用。

## 所需材料
* MacBook

## 安装和配置 Tomcat

> The Apache Tomcat® software is an open source implementation of the Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies. —— Introduction to Tomcat

根据 [Tomcat 官网](http://tomcat.apache.org) 介绍，Tomcat 是一个开源的 Servlet 容器，在支持 Servlet 的同时，内置的 Jasper 编译器也能够将 JavaServer Page(JSP) 编译成对应的 Selvet。此外，Tomcat 内置了一个 HTTP 服务器，因此也能将它视为独立的一个 Web 服务器。

#### 在 MacOS 上安装 Tomcat

通常情况下，只需在 [Tomcat 官网](http://tomcat.apache.org) 上下载 `rar` 或者 `tar.gz` 格式的压缩包即可。但在 MacOS 上，用户可以通过 [Homebrew](https://brew.sh) 下载，Homebrew 是 MacOS 上流行的包管理软件，弥补了 MacOS 平台上缺失了 Ubuntu 上 apt-get 的遗憾。

```shell
// 安装 Tomcat 当前稳定版本
brew install tomcat
// 安装 Tomcat 最新版本
brew install tomcat --devel
```
![](/assets/img/in-post/post-BS-setup/1.png)
安装完成后，按照提示可知安装目录为 `/usr/local/Cellar/tomcat/9.0.2` ，启动 Tomcat 方式如下：

```shell
// 在后台启动 Tomcat
brew services start tomcat
// 在当前终端里运行 Tomcat
catalina run
```
#### 配置 Tomcat

安装的 Tomcat 的 home 目录位于 `/usr/local/Cellar/tomcat/9.0.2/libexec` ，其中的配置文件位于 `conf` 目录下，其中较为重要的配置文件包含 `conf/server.xml` ， `conf/web.xml` 和 `conf/context.xml` 。为了避免端口冲突，修改默认端口 8080 为 9999。这需要修改 `conf/server.xml` 里的 Connect 里的 端口，修改如下：

```xml
<Connector port="9999" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```
#### 运行 Tomcat
在终端键入 `catalina run` ，等到出现 `org.apache.catalina.startup.Catalina.start Server startup in 509 ms` 后，Tomcat 启动成功。在浏览器中键入 `http://localhost:9999` 即可看到默认的 Tomcat 主页。

![](/assets/img/in-post/post-BS-setup/2.png)

在 Tomcat运行的终端中同时键入 `control` 和 `C` 即可中断 Tomcat 服务器。

## 安装和配置 SQL Server
> Microsoft SQL Server is a relational database management system developed by Microsoft. —— Introduction to SQL Server

SQL Server 是由微软开发的关系数据库的解决方案，作为 B/S 应用将要操作的数据来源。

#### 安装 SQL Server

很可惜，SQL Server 并不原生支持 MacOS， 我们可以选择安装在 Docker 里面。Docker 是应用层面的虚拟机，通过MacOS 版的 [下载地址](https://store.docker.com/editions/community/docker-ce-desktop-mac) 下载安装。

Docker 的使用需要管理权限。

![](/assets/img/in-post/post-BS-setup/3.png)

Docker 默认只分配 2GB 的内存，而运行 SQL Server 需要至少 3.25GB 的内存，在 `Preferences` 里的 `Advanced` 面板内可以修改，并且点击 `Apply & Restart`。

在终端中键入以下指令，即可下载 SQL Server 镜像到 Docker 中。

```
docker pull microsoft/mssql-server-linux
```
键入以下命令，建立 SQL Server 实例。

```
docker run -d --name sql_server_demo -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=reallyStrongPwd123' -p 1433:1433 microsoft/mssql-server-linux
```
其中， `-d` 代表该实例会在后台运行；
`--name sql_server_demo` 代表该实例名字；
`-e 'ACCEPT_EULA=Y'` 代表同意 EULA，这样才能在 MacOS 上运行 Linux 内的 SQL Server；
`-e 'SA_PASSWORD=reallyStrongPwd123'` 代表用户名和密码；
`-p 1433:1433` 代表将本地的端口 1433 连接到 Docker 容器的端口 1433，1433 是 SQL Server 默认的 TCP 监听端口；
`microsoft/mssql-server-linux` 代表所选的镜像，也就是刚刚下载的 SQL Server 镜像。

成功运行之后，通过键入以下命令可以看到当前 Docker 中运行的程序。

```
docker ps
```
![](/assets/img/in-post/post-BS-setup/4.png) 

为了能直接操作数据库，通过以下命令安装 sql-cli，sql-cli 是数据库命令行工具。如果没有 npm 可以简单利用 homebrew 安装，npm 是 node.js 包管理工具，类似于 Python 的 pip。

```
npm install -g sql-cli 
```
安装完毕后，通过以下命令可以连接刚刚建立的 SQL Server 数据库。

```
mssql -u sa -p reallyStrongPwd123
```
![](/assets/img/in-post/post-BS-setup/5.png)

#### 建立需要的数据库

在数据库中键入以下命令生成新的数据库 ebookshop。

```sql
create database ebookshop;
```
键入以下命令进入数据库 ebookshop。

```sql
use ebookshop;
```
键入以下命令生成列表 books。

```sql
create table books (
	id     int,
	title  varchar(50),
	author varchar(50),
	price  float,
	qty    int,
	primary key (id));
```
键入以下命令插入五个条目。

```sql
insert into books values (1001, 'Java for dummies', 'Zhu, Xifu', 11.11, 11);
insert into books values (1002, 'More Java for dummies', 'Xu, Jiankui', 22.22, 22);
insert into books values (1003, 'More Java for more dummies', 'Zhu, Xifu', 33.33, 33);
insert into books values (1004, 'A Cup of Java', 'Xu, Jiankui', 55.55, 55);
insert into books values (1005, 'A Teaspoon of Java', 'Su, Jun', 66.66, 66);
```
键入以下命令可以看当前列表的数据情况。

```sql
select * from books
```
![](/assets/img/in-post/post-BS-setup/6.png) 

#### 安装 JDBC Driver for SQL Server

JDBC Driver 全称 Java Database Connecticity，显然是用来连接 Java 代码和数据库，在 Servlet 里将会起到作用，我们先行安装。在 JDBC Driver 官方 [下载页面](https://www.microsoft.com/en-us/download/details.aspx?id=55539) 内下载后，将其中对应 JDK 版本的的 JDBC JAR 包放入 `/Library/Java/Extensions` 内，比如 `mssql-jdbc-6.2.2.jre8.jar`。

## 在 Tomcat 中建立网络应用

#### 建立网络应用框架

回到 Tomcat 的 home 目录下，比如 `/usr/local/Cellar/tomcat/9.0.2/libexec`，在 webapps 里存放的就是各个网络应用。在 webapps 里新建一个文件夹 QueryDemo 作为本次的网络应用。
在 QueryDemo 内新建一个名为 WEB-INF 的文件夹，比如 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF`，用来存放一切网络资源。
在 WEB-INF 内新建一个名为 classes 的文件夹，比如 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF/classes` 用来存放 Servlet。

在 QueryDemo 可以用来存放网络应用的主页，新建 index.html，并写入以下代码。

```html
<html>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
<head>
    <title>电子书查询系统</title>
</head>
<body>
<h2>电子书查询系统</h2>
<form method="get" action="http://localhost:9999/QueryDemo/query">
    <b>选择一个作者：</b>
    <input type="radio" name="author" value="Zhu, Xifu">Zhu, Xifu
    <input type="radio" name="author" value="Xu, Jiankui">Xu, Jiankui
    <input type="radio" name="author" value="Su, Jun">Su, Jun
    <input type="submit" value="Search">
</form>
</body>
</html>
```
此时运行 Tomcat，并在浏览器中键入 `http://localhost:9999/QueryDemo/` 即可看到电子书查询系统的主页，但是目前还没有 Servlet，所以无法交互。

![](/assets/img/in-post/post-BS-setup/7.png) 

#### 创造查询 Servlet

在 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF/classes` 下新建一个名为 QueryServlet.java 的文件，并在其中写入以下代码。

```java
import java.io.IOException;
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import java.sql.*;
import javax.servlet.annotation.*;


@WebServlet("/query")
public class QueryServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=gb2312");
        PrintWriter out = response.getWriter();

        Connection conn = null;
        Statement stmt = null;
        try {
            conn = DriverManager.getConnection(
                    "jdbc:sqlserver://localhost:1433;databaseName=ebookshop", "sa", "Ikalos7816");
            stmt = conn.createStatement();

            String sqlStr1 = "从列表 books 从查找作者为"
                    + "'" + request.getParameter("author") + "'"
                    + "的电子书，并按价格排序";

            String sqlStr = "select * from books where author = "
                    + "'" + request.getParameter("author") + "'"
                    + " and qty > 0 order by price desc";

            out.println("<html><meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\"/>");
            out.println("<head><title>查询结果</title></head><body>");
            out.println("<h3>感谢你的查询</h3>");
            out.println("<p>你的查询是 " + sqlStr1 + "</p>"); // Echo for debugging
            ResultSet rset = stmt.executeQuery(sqlStr);  // Send the query to the server

            int count = 0;
            while(rset.next()) {
                // Print a paragraph <p>...</p> for each record
                out.println("<p>" + rset.getString("author")
                        + ", " + rset.getString("title")
                        + ", $" + rset.getDouble("price") + "</p>");
                count++;
            }
            out.println("<p>==== 找到 " + count + " 项纪录 =====</p>");
            out.println("</body></html>");
        } catch (SQLException ex) {
            ex.printStackTrace();
        } finally {
            out.close();  // Close the output writer
            try {
                if (stmt != null) stmt.close();
                if (conn != null) conn.close();
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        }
    }
}
```
该 Servlet 实现了从数据库中查询给定作者名字的作品，并通过生成 HTML 页面返回查询结果，其中的数据库用户名，密码和数据库名需要和之前设立的 SQL Server 相一致。

我们还需要用 Servlet API 来编译该 Servlet，Tomcat 在 `<TOMCAT_HOME>/lib/servlet-api.jar` 内存放了一个 Servlet API，只要通过 -cp 参数指定该 JAR 包就能编译。具体代码如下：

```shell
javac -cp .:/usr/local/Cellar/tomcat/9.0.2/libexec/lib/servlet-api.jar QueryServlet.java
```
在当前目录中生成 QueryServlet.class，即是可用的 Servlet。

接下去，我们需要配置该 Servlet 的调用 URL，具体在 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF/web.xml` 中实现。目前没有这个文件，所以创建一个在该目录下创建一个名为 web.xml 的配置文件，并在一面写入：


```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app version="3.0"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
 
   <!-- To save as "hello\WEB-INF\web.xml" -->
 
   <servlet>
      <servlet-name>QueryServlet</servlet-name>
      <servlet-class>QueryServlet</servlet-class>
   </servlet>
 
   <!-- Note: All <servlet> elements MUST be grouped together and
         placed IN FRONT of the <servlet-mapping> elements -->
 
   <servlet-mapping>
      <servlet-name>QueryServlet</servlet-name>
      <url-pattern>/query</url-pattern>
   </servlet-mapping>
</web-app>
```
其中 `<servlet>` 是用来绑定 Servlet 的名字和 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF/classes` 内的编译好的 class 文件名字； `<servlet-mapping>` 是用来绑定 Servlet 的名字和调用该 Servlet 的 URL，当前设置的 URL 为 `/query`。

将 `<TOMCAT_HOME>/webapps/QueryDemo/index.html` 内的 get 调用链接设置成与刚设置成的 Servlet 链接一样即可，比如：

```html
<form method="get" action="http://localhost:9999/QueryDemo/query">
```
运行测试结果如下：

![](/assets/img/in-post/post-BS-setup/8.png)
![](/assets/img/in-post/post-BS-setup/9.png)

## Troubleshooting

#### Servlet 返回的 HTML 页面中文出现乱码

只需要将 doGet 或 doPost 的
`response.setContentType("text/html");` 修改为	`response.setContentType("text/html;charset=gb2312");` 或者改为 `response.setContentType("text/html;charset=GBK");` 即可。

#### JDBC Driver 究竟放哪儿

我在这个问题上研究了很久，无论是放在 `<TOMCAT_HOME>/lib` 或者 `<TOMCAT_HOME>/webapps/QueryDemo/WEB-INF/lib` 内都无法起到作用，只有放到系统层面的 `/Library/Java/Extensions` 才起到了作用。

## 参考
* [servlet 输出中文显示为问号"??"的解决办法](http://freeskywcy.iteye.com/blog/1174133)
* [How to Install Apache Tomcat 8 (on Windows, Mac OS, Ubuntu) and Get Started with Java Servlet Programming](https://www.ntu.edu.sg/home/ehchua/programming/howto/Tomcat_HowTo.html#configure)
* [How to Install SQL Server on a Mac](http://database.guide/how-to-install-sql-server-on-a-mac/)







