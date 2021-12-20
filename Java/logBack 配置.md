http://www.dczou.com/viemall/885.html
https://www.ibm.com/developerworks/cn/java/j-lo-practicelog/index.html

logback 和 log4j 是同一个人写的.
logback 由三个模块组成

1. logback-core 是其他模块的基础设施
2. logback-classic 日志门面的具体实现
3. logback-access 提供一些 Http 访问相关功能

配置文件

首先来看下具体的 logback.xml 结构
![1568192706995](D:\Typora\image\1568192706995.png)

```xml
<!-- 
scan 当属性为 true 配置文件发生改变将会被重新加载.默认为 true
scanPeriod 设置检测配置文件是否有修改时间间隔，如果没有给出，默认为毫秒.当 scan true 时此属性生效默认为 1 分钟
debug 当此属性为 true 时，将打印出 logback 内部日志信息，实时查看 logback 运行状态.默认 false
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
    <!-- 
	用来定义变量值的标签 property 有两个属性 name 变量名 value 变量值.
	定义完变量后可以使用 ${name} 来使用变量.
	-->
    <property name="app-name" value="app-name" />
    <property name="logging.path" value="logging.path" />
    
    <!-- 
	每个 logger 都关联到 logger 上下文，默认上下文为 "default".
	但可以使用 contextName 标签来设置.用来区分不同的应用程序记录.
	-->
    <contextName>${app-name}</contextName>
    
    <!-- 默认的控制台日志输出，一般生产环境都是后台启动，这个没太大作用 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d{HH:mm:ss.SSS} %-5level %logger{80} - %msg%n</Pattern>
        </encoder>
    </appender>
    
    <!-- 
	appender 种类
	1. ConsoleAppender 把日志添加到控制台
 	2. FileAppender 把日志添加到文件
	3. RollingFileAppender 滚动记录文件，先将日志记录到文件，当符合某个条件时，将日志记录到其他文件.
	-->
    <appender name="TEST-LOGGER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- append 为 true 时，日志被追加到文件结尾.false 清空现存文件 默认 true -->
        <append>true</append>
        <!-- 
		可以为 appender 添加一个或多个过滤器，可以用任意条件对日志进行过滤.
		appender 有多个过滤器时按照配置顺序执行.
		ThresholdFilter 临界值过滤器，过滤掉指定临界值的日志.当日志级别等于或高于时过滤器返回 NEUTRAL，当小于时将被拒绝
		LevelFilter 级别过滤器，根据日志级别进行过滤.如果日志级别等于配置级别，过滤器会根据 onMath(用于配置符合过滤条件的操作)和 onMismatch(用于配置不符合过滤条件的操作)接收或拒绝日志。

		filter 返回值枚举
		1. DENY 日志将立即被抛弃不再经过其他过滤器
		2. NEUTRAL 有序列表里的下个过滤器过接着处理日志
		3. ACCEPT 日志会被立即处理，不再经过剩余过滤器
		-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <!-- 
		file 标签用于指定被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建.
		没有默认值。
		-->
        <file>
            ${logging.path}/app-log.log
        </file>
        <!-- 
		rollingPolicy 用于描述滚动策略，这个只有 appender 是 RollingFileAppender 时才需要配置.
		这个也会涉及文件的移动和重命名(a.log -> a.log.2018.07.22)
		
		TimeBasedRollingPolicy 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动.
		FixedWindowRollingPolicy 根据固定窗口算法重命名文件的滚动策略.
		-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件输出的文件名: 按天回滚 daily -->
            <FileNamePattern>${logging.path}/app-log.log.%d{yyyy-MM-dd}</FileNamePattern>
            <!-- 日志文件保留天数 -->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <!-- 
		encoder 对记录事件进行格式化.只有 PatternLayoutEncoder 一种类型.
		-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>   
    
    <!-- 
	用来设置某一个包或具体的某一个类的日志打印级别以及指定 appender
	name 用来指定受此 logger 约束的某一个包或者具体的某一个类.
	level 设置打印级别.
	addtivity 描述是否向上级 logger 传递打印信息.默认 true
	-->
    <logger name="com.glmapper.spring.boot.controller" level="${logging.level}" additivity="false">
        <appender-ref ref="TEST-LOGGER" />
    </logger>
    
    <!-- 根 logger，也是一种 logger 且只有一个 level 属性 -->
    <root level="INFO">             
       <appender-ref ref="TEST-LOGGER"/>
    </root>  
</configuration>
```

