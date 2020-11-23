> 跨站脚本攻击(Cross Site Scripting) 避免混淆缩写为 XSS



通过在数据中添加 JavaScript 脚本, 这些脚本程序将会在用户浏览器中执行。这就会产生XSS漏洞。



`jsoup` 对提交得数据进行标签过滤

```java
public void test() {
    String unsafe = "<p><a href='http://example.com/' onclick='stealCookies()'>Link</a></p>";
    String safe = Jsoup.clean(unsafe, Whitelist.basic());
    System.out.println(safe);
}
```

