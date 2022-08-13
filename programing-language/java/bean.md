# Spring中bean的使用

最近在做一个项目的时候遇到一个问题，期望在main方法里面获取到Spring容器里的类，由于main函数是整个代码的入口。所以，直接获取是获取不到的，必须借助其它方法来实现。

首先，创建一个BeanUtil类

```java
package com.tuya.demo.java.starter;

import org.springframework.context.ConfigurableApplicationContext;

public class BeanUtil {

    public static ConfigurableApplicationContext applicationContext;

    public static <T> T getBean(Class<T> c){
        return applicationContext.getBean(c);
    }
}

```

然后我们在Starter里面把Spring上下文注入进去

```java
public class DemoJavaStarterApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(DemoJavaStarterApplication.class, args);
        BeanUtil.applicationContext = run;
        GrpcConsumerService grpc = BeanUtil.getBean(GrpcConsumerService.class);
        System.out.println("====================================");
        System.out.println(grpc);
        System.out.println("====================================");
    }
}
```

