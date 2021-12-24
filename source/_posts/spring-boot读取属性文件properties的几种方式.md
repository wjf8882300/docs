---
title: spring boot读取属性文件properties的几种方式
date: 2021-12-23 13:50:57
tags: java
categories: 语言
---
前三种测试配置文件为springboot默认的application.properties文件
{% blockquote %}
com.zyd.url=http://www.baidu.com
{% endblockquote %}
#### 一、@ConfigurationProperties方式

```
@Configuration
@ConfigurationProperties(prefix = "com.zyd")
public class PropertiesConfig {
    private String url;
}
```

#### 二、使用@Value注解方式
```
@Controller
public class TestController {
 
    @Value("${com.zyd.url}")
    private String url;
}
```

#### 三、使用Environment
```
@Controller
public class TestController {
    @Autowired
    private Environment env;
 
    @RequestMapping("/env")
    public Map<String, Object> env() throws UnsupportedEncodingException {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("url", env.getProperty("com.zyd.url"));
        return map;
    }
}
```

#### 四、使用PropertiesLoaderUtils
可以自定义属性文件，如app-config.properties

> com.zyd.url=http://www.baidu.com

```
try {
            Properties properties = PropertiesLoaderUtils.loadAllProperties("app-config.properties");
            String url = properties.get("com.zyd.url");
        } catch (IOException e) {
            e.printStackTrace();
        }
```		