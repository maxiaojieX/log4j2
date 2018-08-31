
# Spring Boot Log4j2使用

### Log4j2是Log4j的框架的一个改进版，性能方面远远高于Log4j，并提供异步记录器(在多线程场景中，异步记录器比Log4j 1.x和Logback具有18倍的吞吐量和数量级的延迟)和压缩策略，在配置方面一般还是采用XML形式，也可以采用properties的形式。
<hr>
步骤：
1.SpringBoot默认使用了logback，所以在使用Log4j2之前需要先移除该包。
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<!--排除包-->
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>

2.引入Log4j2-starter
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

3.创建log4j2.xml文件。


以上图为例简单介绍Log4j2的新特性(上图配置附录在文档末)。
在12行声明了日志打印使用异步写入(先将日志写入缓冲区，当缓冲区刷满之后再写入到指定日志文件，并清空缓存区)。
immediateFlush="false"设置了不立即刷入
filePattern="...gz"设置了文件格式为gz.
在19行声明了日志滚动策略为文件最大60KB
在31-33行，引用定义的Appender并指定使用该Appender的包。

4.在主配置文件中指向该配置文件，使该配置文件生效。


5.测试打印日志
为了看到滚动策略效果，循环打印日志


连续刷新访问 “/”，可以在日志目录下看到日志生成，因为是异步写入，所以未滚动的文件没有被立即刷入。

打开已滚动生成压缩文件，大小为60K,压缩之后大小为1K



OVER~
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration monitorInterval="60">
    <Properties>
        <Property name="LOG_HOME">applogs</Property>
    </Properties>
    <Appenders >
        <!--控制台输出所有日志-->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
        <!--Info级别日志输出-->
        <RollingRandomAccessFile name="InfoFile"
                                 immediateFlush="false"
                                 fileName="${LOG_HOME}/controller/info.log"
                                 filePattern="${LOG_HOME}/controller/info - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n" />
            <Policies>
                <!--<TimeBasedTriggeringPolicy />-->
                <SizeBasedTriggeringPolicy size="60 KB" />
            </Policies>
            <Filters>
                <ThresholdFilter level="warn" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY" />
            </Filters>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20" />
        </RollingRandomAccessFile>
    </Appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <Loggers>
        <Logger name="com.example.demo.controller" level="debug" additivity="false">
            <AppenderRef ref="InfoFile" />
        </Logger>
    </Loggers>
</Configuration>
```
