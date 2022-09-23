---
title: "SpringBoot配置VueJs或者React的history路由"
date: 2019-10-24T10:24:53+08:00
aliases: [spring-boot-vuejs-and-react-history-router-2]
---

一般在配置前端的文件夹的时候都会指定一个路径，如果你想使用根路径我这种方法不支持根路径，请使用另外一种支持根路径的配置方法。
## 配置
比如把前端放在 h5 路径下面
1. 把前端打包的文件放在resource/META-INF/h5 目录

	![img](/images/DraggedImage-2-20191026140056310.png)

2. 修改Spring Boot WebMvc的配置
```java
    @Configuration
    public class WebMvcConfiguration implements WebMvcConfigurer {

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/h5/**")
                    .addResourceLocations("classpath:/META-INF/h5/");
        }

        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            // 配置如果用户访问的是根目录直接重定向掉h5页面
                registry.addRedirectViewController("/", "h5");
            // 配置用户如果访问h5路径但是没有带index.html 转发到 /h5/index.html 路径
                registry.addViewController("/h5")
                    .setViewName("forward:/h5/index.html");
            registry.addViewController("/h5/")
                    .setViewName("forward:/h5/index.html");
            // 一下是为了给前端 history路由的一些配置
            registry.addViewController("/h5/{spring:\\w+}")
                    .setViewName("forward:/h5/index.html");
            registry.addViewController("/h5/**/{spring:\\w+}")
                    .setViewName("forward/h5/index.html");
            registry.addViewController("/h5/{spring:\\w+}/**{spring:?!(\\.js|\\.css)$}")
                    .setViewName("forward:/h5/index.html");
        }
    }
