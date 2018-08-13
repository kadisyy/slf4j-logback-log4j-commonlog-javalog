# java日志规范

## 配置修改
主要涉及各个日志框架和slf4j之间的转换等
### 日志框架场景
针对不同的应用，不同的组件，其引用的日志框架各不相同，现在就针对每一个场景进行分析。下图说明了各日志框架转换为slf4j，如下：
![image](https://www.slf4j.org/images/legacy.png)
#### 场景一 Logback
logback-classic直接引用的slf4j，不需要桥接。
- maven依赖
添加maven依赖如下， 供参考：
```
    <properties>
        <ch.qos.logback.version>1.2.3</ch.qos.logback.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${ch.qos.logback.version}</version>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.1</version>
        </dependency>
    </dependencies>
```

#### 场景二 Log4j

如果使用的是log4j进行日志打印，需要进行转换，需要把log4j日志框架输出的日志路由到SLF4J上，可引入log4j-over-slf4j。     
比如服务serivce1依赖service2, 但是service2使用log4j进行日志打印。 可以参考如下步骤进行改造：  
**具体用法**：使用log4j-over-slf4j取代log4j，这样log4j接口输出的日志就会通过log4j-over-slf4j路由到SLF4J上，这样即使系统（包含使用的第三方jar库，比如dubbo）都可以将日志最终路由到SLF4J上，进而集中输出。   
- 操作步骤：   
1、去除service2 log4j依赖  
2、在service1添加如下maven依赖，特别是log4j-over-slf4j。


- maven依赖，供参考：
```
   <properties>
        <ch.qos.logback.version>1.2.3</ch.qos.logback.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${ch.qos.logback.version}</version>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>
```

#### 场景三 log4j2 
同log4j配置

#### 场景四 Apache Common Logging 
如果在原有commons-logging 系统里，如果要迁移到slf4j, 使用slf4j 替换commons-logging ，也是可以做到的。原理使用到了上述commons-logging 加载的第二点。需要引入jcl-over-slf4j。这个jar 包提供了一个桥接，让底层实现是基于slf4j。  
比如服务serivce1依赖service2, 但是service2使用log4j进行日志打印。
- 操作步骤：  
1、去除service2 apache common logging的依赖
2、在service1添加如下maven依赖，特别是jcl-over-slf4j
- maven依赖，供参考：
```
    <properties>
        <ch.qos.logback.version>1.2.3</ch.qos.logback.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${ch.qos.logback.version}</version>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.slf4j/jcl-over-slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>
```
#### 场景五 java.util.logging
Java Logging API相关类由JDK提供，我们不能排除掉JDK，因而，在"Java Logging API转SLF4J"过程中采用的转换方案跟"JCL转SLF4J"和"Log4J转SLF4J"采用的转换方案不同，具体思路是以jul-to-slf4j提供的"org.slf4j.bridge.SLF4JBridgeHandler"替换掉Java Logging API使用的Handler（Handler即是Appender），Java Logging API原来使用的Handler将日志输出到Console，文件，数据库等，现在"org.slf4j.bridge.SLF4JBridgeHandler"将日志输出到SLF4J日志框架。简单来说，就是将原本Java Logging API的日志输出重定向到SLF4J日志框架。

Java Logging API的默认配置文件路径为：JDK_HOME/jre/lib/logging.properties，根据以上说明，需要修改“handlers”属性值，修改后"handlers"属性的内容如下：
```
handlers= org.slf4j.bridge.SLF4JBridgeHandler
```
- maven依赖如下，供参考：
```
    <properties>
        <ch.qos.logback.version>1.2.3</ch.qos.logback.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${ch.qos.logback.version}</version>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.slf4j/jul-to-slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>
```


### logback.xml配置
通过logback.xml配置，各个应该会将日志发送到指定主机和端口，此主机和端口即logstash接受端口，简单示例如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />
    <appender name="SYSLOGA" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>172.20.2.101:5140</destination>
        <!-- includeCallerData会产生一定的性能开销，如果压力较大的服务，可以关闭该功能 -->
        <includeCallerData>false</includeCallerData>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeCallerData>false</includeCallerData>
            <includeContext>false</includeContext>
        </encoder>
        <!-- 超过5分钟没有新日志产生，则发送一条空日志用作保持连接 -->
        <keepAliveDuration>5 minutes</keepAliveDuration>
        <!-- 断开后尝试重连的时间间隔，避免过于频繁-->
        <reconnectionDelay>5 second</reconnectionDelay>
        <!-- 消息实际缓冲条数 提前60条丢弃 -->
        <ringBufferSize>64</ringBufferSize>
        <!-- 30分钟重新连接，配合LVS负载均衡-->
        <connectionStrategy>
            　　<roundRobin>
            　　　　<connectionTTL>30 minutes</connectionTTL>
            　　</roundRobin>
        </connectionStrategy>
    </appender>
    <root>
        <appender-ref ref="SYSLOGA" />
    </root>
</configuration>
```

## 代码修改  
引入SLF4J MDC, 通过MDC加入如下字段：  
**注意：**  
建议使用slf4j占位符语法进行日志打印。
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class AppA {
    private static Logger logger = LoggerFactory.getLogger("LoggerA");
    public static void main(String[] args) throws InterruptedException {
        MDC.put("appId", "assd12303sdjaksj");
        MDC.put("appName","kafka_source_sink");
        MDC.put("jobId", "sdhendmnakdhjaebma");
        MDC.put("operatorId","qwerasdf090");
        MDC.put("operatorName", "operatorName1");
        logger.info("test_logback {}", "Hello");
        Thread.sleep(50);
    }
}


```
## LogStash接收结果展示  
如下为logstash通过tcp或者syslog协议接收到的日志格式：
```
{
        "@version" => "1",
         "appName" => "kafka_source_sink",
           "appId" => "assd12303sdjaksj",
           "level" => "INFO",
    "operatorName" => "operatorName1",
        "@metdata" => {
        "ip_address" => "172.20.1.21"
    },
     "thread_name" => "main",
     "logger_name" => "LoggerA",
     "level_value" => 20000,
            "host" => "base1.xxxx.com",
      "@timestamp" => 2018-08-03T06:57:00.974Z,
           "jobId" => "sdhendmnakdhjaebma",
         "message" => "test_logback Hello",
      "operatorId" => "qwerasdf090",
            "port" => 65051
}
```
