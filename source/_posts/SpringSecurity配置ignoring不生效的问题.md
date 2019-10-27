---
title: Spring Security 配置 ignoring 不生效的问题
permalink: spring-security-pei-zhi-ignoring-bu-sheng-xiao-de-wen-ti
id: 5d5e865ce9326100bc7569bb
updated: '2017-12-04T09:29:53.000Z'
date: 2017-12-04 17:28:19
---

 ## 遇到的问题
给 Spring Security 加上了一个 filter，配置了 ignoring 但是，filter 还是会拦截到 ignoring 的路径，导致 ignoring 不生效。
## 问题代码
```
@Bean
private InjectResourceSecurityFilter mySecurityFilter() throws Exception {
    return new InjectResourceSecurityFilter(securityMetadataSource,
            accessDecisionManager, authenticationManagerBean());
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .addFilterBefore(mySecurityFilter(), InjectResourceSecurityFilter.class)
            .authorizeRequests()
            .antMatchers("/static/**").permitAll()
            .antMatchers("/index.html", "/").permitAll()

            .anyRequest().authenticated()
            .and()
            .csrf().disable();
}

@Override
public void configure(WebSecurity web) throws Exception {
    super.configure(web);
    web.ignoring()
            .antMatchers("/static/**", "/index.html",
                    "/favicon.ico", "/");
}
```

## 解决问题
通过 Google 找到在 stackoverflow上的一个回答 <a href="https://stackoverflow.com/a/40969780" target="_blank">stackoverflow</a>
上说去掉 filter 上的 `Component` 注解就好，由于我在这里写的是 `@Bean`，所以把`@Bean` 给去掉，解决 ignoring 不生效的问题。

