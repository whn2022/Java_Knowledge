# 一、流

## getResourceAsStream

```
InputStream input = Configuration.class.getResourceAsStream(FILE_NAME);
```

当FILE_NAME为绝对路径时，任意类的流都可以

当FILE_NAME为相对路径时，需要对应路径匹配，比如

```
src/main/
├── java/
│   └── com/example/
│       ├── config/
│       │   └── AppConfig.java
│       ├── service/
│       │   └── UserService.java
│       └── controller/
│           └── UserController.java
└── resources/
    ├── global-config.properties          ← 全局配置
    ├── com/example/config/
    │   └── app-config.properties         ← 配置模块专用
    ├── com/example/service/
    │   └── service-config.properties     ← 服务模块专用
    └── com/example/controller/
        └── web-config.properties         ← 控制器模块专用
        
package com.example.config;

public class AppConfig {
    
    public void loadConfig() {
        // 相对路径：在当前包路径下查找
        InputStream is = AppConfig.class.getResourceAsStream("app-config.properties");
        // 实际查找：src/main/resources/com/example/config/app-config.properties ✅
        
        if (is != null) {
            System.out.println("AppConfig: 找到配置文件");
        } else {
            System.out.println("AppConfig: 未找到配置文件");
        }
    }
}

package com.example.service;

public class UserService {
    
    public void loadConfig() {
        // 相同的相对路径，但查找位置不同！
        InputStream is = UserService.class.getResourceAsStream("app-config.properties");
        // 实际查找：src/main/resources/com/example/service/app-config.properties ❌
        
        if (is != null) {
            System.out.println("UserService: 找到配置文件");
        } else {
            System.out.println("UserService: 未找到配置文件"); // 这个会执行
        }
    }
}
```

## bufferedReader

内部维护一个字符缓冲区，默认8192大小，可以在构造方法中指定，可以包装其他Reader

### read()

读取单个字符，重载方法包括read（char[]），read（char[],int off,int len）

### readline()

按行读取

# 二、常用类

键值对都是字符串类型，线程安全

### 文件操作方法

load(inputStream) 从输入流加载

load(Reader) 支持编码

loadFromXML(InputStream) 从XML文件加载

store storetoXML

# 三、注解及自定义注解

## 自定义注解

### 自定义注解基本声明

public @interface MyAnnotation{} // 最简单的注解

### 添加参数

String name(); 

int age();

String email() default ""; //默认值

## 元注解

对注解进行注解的注解类

### @Target

指定注解使用的位置

@Target({
    ElementType.TYPE,          // 类、接口、枚举
    ElementType.FIELD,         // 字段
    ElementType.METHOD,        // 方法
    ElementType.PARAMETER,     // 方法参数
    ElementType.CONSTRUCTOR,   // 构造函数
    ElementType.LOCAL_VARIABLE,// 局部变量
    ElementType.ANNOTATION_TYPE,// 注解类型
    ElementType.PACKAGE,       // 包
    ElementType.TYPE_PARAMETER,// 类型参数(JDK8+)
    ElementType.TYPE_USE       // 类型使用(JDK8+)
})

### @Retention

指定注解保留策略

@Retention（

RetentionPolicy.SOURCE（源码期，编译后丢弃）,

RetentionPolicy.CLASS(默认，编译期，运行时不可用,保存在class文件里)

RetentionPolicy.RUNTIME（运行时可以通过反射获取）

)

### @Documented

是否包含在javadoc中

### @Inherited

注解继承

```
import java.lang.annotation.Inherited;

@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface InheritedAnnotation {
    String value();
}

@InheritedAnnotation("父类注解")
class Parent {
}

// 子类会自动继承父类的@InheritedAnnotation注解
class Child extends Parent {
}
```

### @Repeatable

```
// Java 8 之前 - 不支持重复注解
@Schedules({
    @Schedule(dayOfMonth="last"),
    @Schedule(dayOfWeek="Fri", hour="23")
})
public void doPeriodicCleanup() { ... }

// Java 8+ 使用 @Repeatable - 更简洁
@Schedule(dayOfMonth="last")
@Schedule(dayOfWeek="Fri", hour="23")
public void doPeriodicCleanup() { ... }
```

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Schedules {
    Schedule[] value();  // 必须是数组类型，名称必须是value
}
```

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(Schedules.class)  // 指定容器注解
public @interface Schedule {
    String dayOfMonth() default "first";
    String dayOfWeek() default "Mon";
    int hour() default 12;
}
```

```
public class TaskService {
    
    @Schedule(dayOfMonth="last")
    @Schedule(dayOfWeek="Fri", hour=23)
    @Schedule(dayOfWeek="Mon", hour=8)
    public void doPeriodicCleanup() {
        System.out.println("执行定期清理任务");
    }
}
```