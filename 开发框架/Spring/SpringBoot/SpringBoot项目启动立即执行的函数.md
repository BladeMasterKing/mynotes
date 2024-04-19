# SpringBoot项目启动立即执行的函数

> 项目启动需要加载数据到redis

## CommandLineRunner

```java
package com.example.demo.config;
import com.example.demo.service.ICodeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Component
// 初始化加载优先级，数字越小优先级越高
@Order(1) 
public class InitData2 implements CommandLineRunner {
    public static Map<Integer, String> codeMap = new HashMap<Integer, String>();
    @Autowired
    private ICodeService codeService;
    @Override
    public void run(String... args) throws Exception {
        // 查询数据库数据
        List<String> codeList = codeService.listAll();
        for (int i = 0; i < codeList.size(); i++) {
            codeMap.put(i, codeList.get(i));
        }
    }
}
```

## ApplicationRunner

```java
package com.example.demo.config;
import com.example.demo.service.ICodeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Component
// 初始化加载优先级，数字越小优先级越高
@Order(1)
public class InitData3 implements ApplicationRunner {
    public static Map<Integer, String> codeMap = new HashMap<Integer, String>();
    @Autowired
    private ICodeService codeService;
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("示例3：加载codeMap中......");
        // 查询数据库数据
        List<String> codeList = codeService.listAll();
        for (int i = 0; i < codeList.size(); i++) {
            codeMap.put(i, codeList.get(i));
        }
    }
}
```