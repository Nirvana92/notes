`tomcat` 的架构图: 

![tomcat](/Users/edz/Documents/GitHub/notes/images/tomcat.png)

`Server` 代表整个`Catalina` 的`servlet`容器。

`Engine`代表一个`Catalina`的`servlet` 引擎。最可能包含一个或多个子容器[`Host`或`Context`实现]。

`Host`代表一个包含多个上下文[`Context`]的虚拟主机。

`Context: `单个`ServletContext`的表示形式。包含一个或多个`Servlet`的包装者[`Wrappers`]。

`Wrapper: `代表了单个的`servlet`的定义。可以支持多个`servlet`实例。除非`servlet`实现了`SingleThreadModel`。

> 实现了`SingleThreadModel`的`servlet`保证只有一个线程进去到`servlet`中的`service`方法。`servlet api 2.4`  往后废弃。