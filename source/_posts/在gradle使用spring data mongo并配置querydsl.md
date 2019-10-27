---
title: 在 gradle 使用 spring-data-mongo 并配置 querydsl
permalink: zai-gradle-shi-yong-spring-data-mongo-bing-pei-zhi-querydsl
id: 5d5e865ce9326100bc7569b3
updated: '2018-01-06T00:47:39.000Z'
date: 2017-10-18 09:59:12
---

### 准备工作
1.Java 版本是 1.8
2.Gradle 版本是 4.2.1
3.spring boot 版本是 1.5.7.RELEASE
4.spring-boot-starter-data-mongodb
5.创建 gradle 新项目

### 编辑 gradle 配置文件

1.配置 spring boot
```
buildscript {
    ext {
        springBootVersion = '1.5.7.RELEASE'
    }
    repositories {
        mavenCentral()
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath "gradle.plugin.com.ewerk.gradle.plugins:querydsl-plugin:1.0.9"
    }
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
repositories {
    mavenLocal()
    mavenCentral()
}
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.springframework.boot:spring-boot-starter-data-mongodb')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
> spring boot 已经配置好了，下面就可以配置 querydsl 了
2. 配置 querydsl
> 在 build.gradle 配置文件中新增
```
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.springframework.boot:spring-boot-starter-data-mongodb')
    testCompile('org.springframework.boot:spring-boot-starter-test')
    compile "com.querydsl:querydsl-mongodb" // querydsl
    compile "com.querydsl:querydsl-apt"  // querydsl
}
sourceSets {
    main {
        java {
            srcDir "$buildDir/generated/source/main"
        }
    }
}
querydsl {
    springDataMongo = true
    querydslSourcesDir = "$buildDir/generated/source/main"
}
```
> querydsl 的 gradle配置已经完成，但是这个时候如果你使用 Qxxx  还是找不到类。需要先编译 querydsl 才可以找到类
3.执行命令
```
gradle build -x test 
```
> 上面的命令就是使用 gradle 编译项目，同时忽略测试
### 使用
> **注意**:你的 Repository 需要基础`org.springframework.data.querydsl.QueryDslPredicateExecutor`这个接口。
```
//BookRepository.java
public interface BookRepository extends PagingAndSortingRepository<Book,
        String>, QueryDslPredicateExecutor<Book>{
    @Override
    List<Book> findAll(Predicate predicate);
}

```
1.在代码中使用
```
BooleanExpression expression = QBook.book.name.contains("Java");
List<Book> books = bookRepository.findAll(expression);
```
2.在 spring mvc 中使用
> 注意:在 spring mvc中的话你要自定义搜索需要继承`org.springframework.data.querydsl.binding.QuerydslBinderCustomizer`这个接口，然后使用Java8 中的默认方法重写`customize` 方法。
```
//BookRepository.java
public interface BookRepository extends PagingAndSortingRepository<Book,
        String>, QueryDslPredicateExecutor<Book>,
        QuerydslBinderCustomizer<QBook> {
    @Override
    default void customize(QuerydslBindings bindings, QBook root) {
        // 绑定 book 中的 name 字段。判断列的值是否存在 springmvc 传过来的字符串
        bindings.bind(root.name)
                .first(StringExpression::contains);
        // 排除 
        bindings.excluding(root.createAt, root.detail, root.nodes, root
                .updateAt, root.url);
    }
    @Override
    List<Book> findAll(Predicate predicate);
}

```

```
//BookCtrl.java
/**
 * @author chenzhenjia
 * @since 2017/10/17
 */
@Slf4j
@RestController
@RequestMapping("/v1/books")
public class BookCtrl {
    private final BookRepository bookRepository;

    @Autowired
    public BookCtrl(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    @GetMapping("/search")
    public List<Book> search(@QuerydslPredicate(root = Book.class) Predicate
                                     predicate) {
        log.info("搜索, param{}", predicate);
        return bookRepository.findAll(predicate);
    }

}

```

