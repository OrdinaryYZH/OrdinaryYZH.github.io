## 第8章 SPring MVC之初体验

搭建一个最简单的Spring MVC环境：

1. maven依赖servlet-api和spring-webmvc
2. web.xml中配置DispatcherServlet
3. 创建Spring MVC xml配置文件:
   * `<mvc:annotation-driven/>`

   * `<context: component-scan base-package="xxx">`，一般还配置只扫面@Controller注解

     ```xml
     <context:component-scan base-package="com.xxx"
                             use-default-filters="false">
         <context:include-filter type="annotation"
                                 expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     ```

4. 创建Controller（和View）

