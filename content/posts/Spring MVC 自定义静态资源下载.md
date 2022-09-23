---
title: "Spring MVC 自定义静态资源下载"
date: 2018-09-04T14:18:53+08:00
aliases: [spring-mvc-zi-ding-yi-jing-tai-zi-yuan-xia-zai]
---


在 Spring MVC 中上传文件后把文件保存在本地，但是数据库保存了一份路径，所有需要实现下载文件，网上的教程一般都是自己写一个Controller 去下载，但是我这里直接使用Spring MVC的 `ResourceHttpRequestHandler` 实现动态下载静态资源，只需一点点代码然后加一段配置就可以实现下载附件功能。
## 第一步
自定义一个Handler，并重写 `ResourceHttpRequestHandler` 的 `processPath` 方法。

```java
@Slf4j
public class CustomResourceHttpRequestHandler extends ResourceHttpRequestHandler {
    private final UploadRepository uploadRepository;
    public CustomResourceHttpRequestHandler(UploadRepository uploadRepository) {
        this.uploadRepository = uploadRepository;
    }
    // 重写该方法实现动态获取文件路径，而不是直接获取本地文件的路径
    @Override
    protected String processPath(String path) {
        Optional<Upload> uploadOption = uploadRepository.findById(path);
        return uploadOption.map(Upload::getPath)
                .orElseGet(() -> {
                    log.warn("文件获取path不存在。path={}", path);
                    return  null;
                });
    }
}
```
## 第二部配置
在 Spring Boot 中添加一下配置，注意LocationValues 的路径，必须以 `/` 结尾，访问本地文件必须有 `file:` ，不然会访问不到，报404
```java
    @Bean
    public ResourceHttpRequestHandler customResourceHttpRequestHandler(UploadRepository uploadRepository) {
        ResourceHttpRequestHandler requestHandler = new 
                   CustomResourceHttpRequestHandler(uploadRepository); 
        requestHandler.setLocationValues(Collections
                          .singletonList("file:/D:/upload/"));
        // 资源缓存配置，缓存90天
        requestHandler.setCacheControl(CacheControl.maxAge(90, TimeUnit.DAYS));
        return requestHandler;
    }
    @Bean
    public SimpleUrlHandlerMapping sampleServletMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Integer.MAX_VALUE - 2);
        Properties urlProperties = new Properties();
        urlProperties.put("/files/*", "customResourceHttpRequestHandler");
        mapping.setMappings(urlProperties);
        return mapping;
    }
```
## 测试
在地址栏输入地址,最后一位是数据库中 附件的id

http://localhost:8080/files/{id}