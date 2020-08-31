java 层面的spi: 

为什么需要spi。java定义了一些规范。然后让三方去实现。在代码中, 程序并不知道如何找到接口和对应实现类的关系。并通过spi的方式指定具体实现类。

```java
ServiceLoader<Driver> load = ServiceLoader.load(Driver.class);
load.forEach(load::getClass);
```

java的spi也是打破了双亲委派模型的一次案例。

正常的按照双亲委派模型: 

`appClassLoader -> extClassLoader -> bootstrapClassLoader`

但是在spi中, 需要appClassLoader去加载这些外部实现类。所以java通过一种不太优雅的方式来满足这个需求。通过`Thread.currentThread.getContextClassLoader()` 来加载。其实就是bootstrapClassLoader通过appClassLoader来加载类。

之前jdk的实现: 比如`jdbc`, 之前处理的方式是通过如下方式: 

```java
// Class.forName() 是通过获取方法调用者的类, 并通过调用者的类加载器来对参数中的类进行加载
Class.forName("");
```

来对具体的实现类进行加载。

现在jdk的实现: 通过spi的方式对实现类进行加载: `java.sql.DriverManager#loadInitialDrivers`

`spi`方式通过`Thread.currentThread.getContextClassLoader()`拿到`classloader`。拿到的也是`AppClassLoader`。

查看各个classloader的加载类的范围: 

```java
System.out.println("--------------bootstrapClassLoader------------");
URL[] urLs = Launcher.getBootstrapClassPath().getURLs();
for (URL urL : urLs) {
  System.out.println(urL.toString());
}

System.out.println("--------------extClassLoader------------");
URLClassLoader extClassLoader = (URLClassLoader) 	ClassLoader.getSystemClassLoader().getParent();
//System.out.println(extClassLoader);
URL[] extUrls = extClassLoader.getURLs();
for (URL extUrl : extUrls) {
  System.out.println(extUrl);
}

System.out.println("--------------appClassLoader------------");
URLClassLoader appClassLoader = (URLClassLoader) Application.class.getClassLoader();
// System.out.println(appClassLoader);
URL[] appUrls = appClassLoader.getURLs();
for (URL appUrl : appUrls) {
  System.out.println(appUrl);
}
```

```
--------------bootstrapClassLoader------------
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/resources.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/rt.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/sunrsasign.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jsse.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jce.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/charsets.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/jfr.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/classes
--------------extClassLoader------------
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunec.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/nashorn.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/cldrdata.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/jfxrt.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/dnsns.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/localedata.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/jaccess.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/ext/zipfs.jar
file:/System/Library/Java/Extensions/MRJToolkit.jar
```

**问题**: 在使用dubbo的spi的时候为什么需要在接口上添加一个@SPI注解。

在`ExtensionLoader`中的`getExtensionLoader`将判断`@SPI`注解去掉，单纯的spi功能是可以用的。

#### ico

#### aop

