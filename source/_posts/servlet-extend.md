title: 一次关于servlet的扩展
date: 2014-11-17 22:43:31
tags: servlet
categories: web开发
---

####一.需求
现有一个web框架，从HttpServletRequest中获取调用request.getParameter("orderId")获取订单的ID，为了安全，我们对订单的ID加密了，也就是此时外部传递过来的orderId已经加密，但是这个web框架不认识加密后的orderId，这就需要我们在自己的web应用中把orderId解密，然后继续传递。

<!-- more -->

####二.实现思路
1.直接先解密orderId，然后再修改request中的orderId，你会发现你没法修改，servlet规范不允许修改request。
2.对HttpServletRequest进行包装，在进入web框架的时候把这个HttpServletRequest包装一下
```java
public class MyHttpServletRequestWrapper extends HttpServletRequestWrapper {

    private Map<String, String> param = new HashMap<String, String>();

    /**
     * @param request
     */
    public MyHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    // 重写获取参数值的方法
    @Override
    public String getParameter(String name) {
        if (param.containsKey(name)) {
            return param.get(name);
        }
        return super.getParameter(name);
    }

    public void addParameter(String name, String value) {
        param.put(name, value);
    }
}
```
增加解密orderId的filter
```java
public class MyOrderIdFilter implements Filter {
    @Override
    public void destroy() {
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse servletResponse, FilterChain filterChain)                                                                   throws IOException,                                                         ServletException {
        String orderId = request.getParameter("orderId");
        // 对orderId解密
        orderId = WebUtils.decode(orderId);
        MyHttpServletRequestWrapper myRequestWrapper = new MyHttpServletRequestWrapper(                        (HttpServletRequest) request);
        myRequestWrapper.addParameter("orderId", orderId);
        // 把包装后的Request传递给下一个Filter
        filterChain.doFilter(myRequestWrapper, servletResponse);
    }
    @Override
    public void init(FilterConfig arg0) throws ServletException {
    }
}
```
