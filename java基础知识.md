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