SpringMvc 是 spring 的一个模块，基于 MVC 的一个框架。这样理解的话，spring的原理mvc里面都有啊，这只是web嘛，Model View Controller

流程说明（重要）

客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，可以根据 xml 配置、注解进行查找，开始由 HandlerAdapter 适配器处理。
HandlerAdapter 会根据 Handler 来调用真正的处理器来处理请求，并处理相应的业务逻辑。
处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
ViewResolver 会根据逻辑 View 查找实际的 View。
DispaterServlet 把返回的 Model 传给 View（视图渲染）。
把 View 返回给请求者（浏览器）

### MVC 的单例情况

mvc是单例模式,所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的,解决方案是在控制器里面不能写字段。

单例bean是整个应用共享的，所以需要考虑到线程安全问题，之前在玩springmvc的时候，springmvc中controller默认是单例的，有些开发者在controller中创建了一些变量，那么这些变量实际上就变成共享的了，controller可能会被很多线程同时访问，这些线程并发去修改controller中的共享变量，可能会出现数据错乱的问题；所以使用的时候需要特别注意。

### WebApplicationContext
WebApplicationContext 继 承 了 ApplicationContext
并 增 加 了 一 些 WEB 应 用 必 备 的 特有 功 能 ， 它 不 同 于 一
般 的 ApplicationContext ， 因 为 它 能 处 理 主 题 ， 并 找
到 被 关 联 的 servlet。

评论：我们有一大堆的 ApplicationContext，也就是顶层是 BeanFactory，就是spring容器嘛，Java不一定做web服务器啊，能量是很大的，所以web后端服务器只是Java生态的一个子类，所以有专门的 WebApplicationContext