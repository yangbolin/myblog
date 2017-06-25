title: velocity工具函数以及InputStream到String的转换方法
date: 2015-10-15 20:41:33
tags: JAVA
categories: 编程开发
---
####一.背景
最近在做需求的时候，需要在程序中把一个vm文件渲染成一个字符串，同时也需要把一个InputStream流转换成一个字符串，虽然说很简单，但是自己还是翻了翻以前的代码，也在网上找了相关的例子，为了下次使用的是能能够快速找到，专门记录一下。
<!-- more -->
####二.实现
#####velocity渲染的工具类
```java
private static final Logger LOG = LoggerFactory.getLogger(VelocityUtil.class);

public static String render(String content, Map<String, String> param) {
    VelocityEngine ve = new VelocityEngine();
    VelocityContext context = new VelocityContext(param);
    try {
    	//解决velocity LOG的问题       
    	ve.setProperty(RuntimeConstants.RUNTIME_LOG_LOGSYSTEM, new NullLogChute());
        ve.init();
        StringWriter writer = new StringWriter();
        ve.evaluate(context, writer, content, content);
        return writer.toString();
    } catch (Exception e) {
        LOG.error("velocity render exception...", e);
    }

    return null;
}
```
####InputStream到String转换的工具类
```java
/**
 * 把流转换成字符串的工具函数
 * 
 * @param in
 * @return
 * @throws Exception
 */
public static String inputStreamToString(InputStream in) throws Exception {
    StringBuilder sb = new StringBuilder();
    String line = null;
    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
    try {
        while ((line = reader.readLine()) != null) {
            sb.append(line.trim());
        }
    } catch (IOException e) {
        throw e;
    } finally {
        try {
            in.close();
        } catch (IOException e) {
            throw e;
        }
    }
    return sb.toString();
}
```
也可以直接使用org.apache.commons.io.IOUtils这个工具函数，注意commons-io包中工具函数的使用。
