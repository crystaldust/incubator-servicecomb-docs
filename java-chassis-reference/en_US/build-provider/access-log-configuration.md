## Concepts

ServiceComb provides access log functionality based on Vert.x. When users use REST over Vertx communication, access log printing can be enabled through a simple configuration.

## Scenarios

The user may need to open the access log when debugging the service. In the case of using the REST over servlet communication method, the access log function of the web container can be used; and in the case of using the REST over Vertx communication method, a set of access log functions provided by ServiceComb can be used.

## Configuration

### Enable Access Log

Users need to add configuration in the microservice.yaml file to enable access log. The configuration example is as follows:

```yaml
servicecomb:
  accesslog:
    enabled: true  ## Enable access log
```

_**Access log Configuration Items**_

| Configuration Item            | Rabge                | Default Value        | Description                            |
| :---------------------------- | :------------------- | :------------------- | :------------------------------------- |
| servicecomb.accesslog.enabled | true/false           | false                | 如果为true则启用access log，否则不启用 |
| servicecomb.accesslog.pattern | 表示打印格式的字符串 | "%h - - %t %r %s %B" | 配置项见_**日志元素说明表**_           |

> _**Note**_
>
> - The 2 items are optional, if not configured, the default value will be applied.

### Log format configuration

目前可用的日志元素配置项见 ***日志元素说明表(Apache & W3C)*** 和 ***日志元素说明表(ServiceComb)*** 。

_**日志元素说明表 (Apache & W3C)**_

| 元素名称                                      | Apache日志格式                        | W3C日志格式  | 说明                                                         |
| :-------------------------------------------- | :------------------------------------ | :----------- | :----------------------------------------------------------- |
| HTTP method                                   | %m                                    | cs-method    | -                                                            |
| HTTP status                                   | %s                                    | sc-status    | -                                                            |
| Duration in second                            | %T                                    | -            | -                                                            |
| Duration in millisecond                       | %D                                    | -            | -                                                            |
| Remote hostname                               | %h                                    | -            | -                                                            |
| Local hostname                                | %v                                    | -            | -                                                            |
| Local port                                    | %p                                    | -            | -                                                            |
| Size of response                              | %B                                    | -            | 如果消息体长度为零则打印"0"                                  |
| Size of response                              | %b                                    | -            | 如果消息体长度为零则打印"-"                                  |
| First line of request                         | %r                                    | -            | 包含HTTP Method、Uri、Http版本三部分内容                     |
| URI path                                      | %U                                    | cs-uri-stem  | -                                                            |
| Query string                                  | %q                                    | cs-uri-query | -                                                            |
| URI path and query string                     | -                                     | cs-uri       | -                                                            |
| Request protocol                              | %H                                    | -            | -                                                            |
| Datetime the request is received              | %t                                    | -            | 按照默认设置打印时间戳，格式为"EEE, dd MMM yyyy HH:mm:ss zzz"，语言为英文，时区为GMT |
| Configurable datetime the request is received | %{PATTERN}t                           | -            | 按照指定的格式打印时间戳，语言为英文，时区为GMT              |
| Configurable datetime the request is received | %{PATTERN&#124;TIMEZONE&#124;LOCALE}t | -            | 按照指定的格式、语言、时区打印时间戳。允许省略其中的某部分配置（但两个分隔符号"&#124;"不可省略）。 |
| Request header                                | %{VARNAME}i                           | -            | 如果没有找到指定的header，则打印"-"                          |
| Response header                               | %{VARNAME}o                           | -            | 如果没有找到指定的header，则打印"-"                          |
| Cookie                                        | %{VARNAME}C                           | -            | 如果没有找到指定的cookie，则打印"-"                          |

_**日志元素说明表(ServiceComb)**_

| Element            | Placeholder       | Comment                                                   |
| :----------------- | :---------------- | :-------------------------------------------------------- |
| TraceId            | %SCB-traceId      | 打印ServiceComb生成的trace id，找不到则打印"-"            |
| Invocation Context | %{VARNAME}SCB-ctx | 打印key为`VARNAME`的invocation context值，找不到则打印"-" |

### Log output file configuration

The log print implementation framework for Access log defaults to Log4j and provides a default set of log files. Users can override these configurations in their own defined log4j.properties file. User-configurable log file configuration items are listed in the table below.

_**日志文件配置项**_

| 配置项                               | 默认值           | 含义                       | 说明                                       |
| :----------------------------------- | :--------------- | :------------------------- | :----------------------------------------- |
| paas.logs.accesslog.dir              | ${paas.logs.dir} | 日志文件输出目录           | 与普通日志输出到同一个目录中               |
| paas.logs.accesslog.file             | access.log       | 日志文件名                 | -                                          |
| log4j.appender.access.MaxBackupIndex | 10               | 最大保存的日志滚动文件个数 | -                                          |
| log4j.appender.access.MaxFileSize    | 20MB             | 日志文件最大体积           | 正在记录的文件达到此大小时触发日志滚动存储 |
| log4j.appender.access.logPermission  | rw-------        | 日志文件权限               | -                                          |

> _**Note**_ 
> Since ServiceComb's log printing function only relies on the slf4j interface, users can select other log printing frameworks. Users need to configure the log file output option when selecting other log printing frameworks.

### Log implementation framework switched to logback

> For the project that uses logback as the log printing framework, the log printing framework needs to be changed from Log4j to logback and some configurations are added to make the access log function take effect.

#### 1. Exclude Log4j dependencies

Before switching the log implementation framework to logback, you need to check the dependencies of the project and exclude Log4j related dependencies. Run the maven command dependency:tree in the project, find the ServiceComb component that depends on Log4j, and add the following configuration to its <dependency> dependency:

```xml
<exclusion>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</exclusion>
```

#### 2. Add a logback dependency

Add a dependency for the logback in the pom file:

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
</dependency>
```

#### 3. Configure the logger for the access log component

Since the log printing component provided by ServiceComb obtains the logger named accesslog to print the access log, the key to switching the log implementation framework from Log4j to logback is to provide a file called accesslog and configure the log output file for it. The following is an example of the configuration of the access log in the logback configuration file. This example only shows the configuration related to the access log. Other log configurations are omitted:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <!-- 用户可根据需要自定义appender -->
  <appender name="ACCESSLOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>./logs/access.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>./logs/access-%d{yyyy-MM-dd}.log</fileNamePattern>
    </rollingPolicy>
    <!-- 注意：由于access log的内容是在代码中完成格式化的，因此这里只需输出message即可，无需添加额外的格式 -->
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <!-- 提供一个名为"accesslog"的logger供access log打印组件使用 -->
  <logger name="accesslog" level="INFO" additivity="false">
    <appender-ref ref="ACCESSLOG" />
  </logger>
</configuration>
```

### Custom Extended Access Log

Users can customize their AccessLogItem by using the AccessLogItem extension mechanism provided by ServiceComb.

#### Related class description

1. `AccessLogItem`

```java
  public interface AccessLogItem<T> {
    /**
     * 从accessLogParam中获取特定的内容,组装成access log的打印内容并返回
     */
    String getFormattedItem(AccessLogParam<T> accessLogParam);
  }
```

  The definition of AccessLogItem is as shown above. When each request triggers Access Log printing, ServiceComb's Access Log mechanism will traverse a valid AccessLogItem, call the getFormattedItem method to get the Access Log fragment generated by this Item, and splicing all the fragments into an Access Log. In the log file.

   The parameter AccessLogParam<T> contains the request start time, the end time, and the request context information of type T. In the REST over Vertx communication mode, the type T is the RoutingContext of Vert.x.

1. `VertxRestAccessLogItemMeta`

```java
  // pattern占位符前缀
  protected String prefix;
  // pattern占位符后缀
  protected String suffix;
  // 优先级序号
  protected int order;
  // AccessLogItem构造器
  protected AccessLogItemCreator<RoutingContext> accessLogItemCreator;
```

  `VertxRestAccessLogItemMeta`包含如上属性，它定义了ServiceComb如何解析pattern字符串以获得特定的AccessLogItem。

- 如果用户想要定义一个占位符为`%user-defined`的`AccessLogItem`，则需要声明一个`VertxRestAccessLogItemMeta`的子类，设置prefix="%user-defined"，suffix=null，当`AccessLogPatternParser`解析到"%user-defined"时，从此meta类中取得`AccessLogItemCreator`创建对应的`AccessLogItem`。**注意**：由于"%user-defined"占位符中没有变量部分，因此调用`AccessLogItemCreator`传入的配置参数为null。
- 如果用户想要定义一个占位符为`%{VARNAME}user-defined`的`AccessLogItem`，则声明的`VertxRestAccessLogItemMeta`子类中，设置prefix="%{"，suffix="}user-defined"，当`AccessLogPatternParser`解析到"%{VARNAME}user-defined"时，会截取出"VARNAME"作为配置参数传入`AccessLogItemCreator`，创建一个`AccessLogItem`。

  `VertxRestAccessLogItemMeta`有一个子类`CompositeVertxRestAccessLogItemMeta`，当用户需要定义多个AccessLogItem时，可以将多个`VertxRestAccessLogItemMeta`聚合到`CompositeVertxRestAccessLogItemMeta`中。Parser加载到类型为`CompositeVertxRestAccessLogItemMeta`的AccessLogItemMeta时，会调用其`getAccessLogItemMetas()`方法获得一组AccessLogItemMeta。`VertxRestAccessLogItemMeta`使用SPI机制加载，而`CompositeVertxRestAccessLogItemMeta`可以让用户只在SPI配置文件中配置一条记录就加载多条meta信息，给了用户更灵活的选择。

1. `AccessLogItemCreator`

```java
  public interface AccessLogItemCreator<T> {
    // 接收配置值，返回一个AccessLogItem。如果AccessLogItem的占位符没有可变的配置值部分，则此方法会接收到null。
    AccessLogItem<T> createItem(String config);
  }
```

The user instantiates his AccessLogItem by setting the AccessLogItemCreator in the custom VertxRestAccessLogItemMeta. Since this is a functional interface, when the AccessLogItem is initialized in a simple way, you can directly define the Creator using a Lambda expression to simplify development.

#### AccessLogItemMeta的匹配规则

1. Once AccessLogItemMeta is loaded into the Parser, it will be sorted once. Parser will match the meta list from front to back when parsing the pattern string. The general matching rules are as follows:
   1. Prioritize matching high priority metas.
   2. Match the meta with suffix first. When matching multiple suffixes with meta, take the one with the smallest suffix.
   3. Match the meta of the placeholder first, for example, there are two metas, "%abc" and "%a". If the match is "%abc", it will return directly, no longer match "%a".

#### 示例说明

1. 扩展自定义AccessLogItem

 First, the user needs the AccessLogItem interface to implement their own item:

```java
  public class UserDefinedAccessLogItem implements AccessLogItem<RoutingContext> {
    private String config;

    public UserDefinedAccessLogItem(String config) {
      this.config = config;
    }

    @Override
    public String getFormattedItem(AccessLogParam<RoutingContext> accessLogParam) {
      // 此处是用户自定义的逻辑，需要从AccessLogParam或其他地方取相关数据，生成并返回access log片段
      return "user-defined-[" + config + "]-[" + accessLogParam.getStartMillisecond() + "]";
    }
  }
```

1. 定义AccessLogItem的meta类

Inherit the VertxRestAccessLogItemMeta or CompositeVertxRestAccessLogItemMeta class to define the suffix of the AccessLogItem and other information:

```java
  public class UserDefinedCompositeExtendedAccessLogItemMeta extends CompositeVertxRestAccessLogItemMeta {
    private static final List<VertxRestAccessLogItemMeta> META_LIST = new ArrayList<>();

    static {
      META_LIST.add(new VertxRestAccessLogItemMeta("%{", "}user-defined", UserDefinedAccessLogItem::new));
    }

    @Override
    public List<VertxRestAccessLogItemMeta> getAccessLogItemMetas() {
      return META_LIST;
    }
  }
```

1. 配置SPI加载文件

In the resources/META-INF/services/ directory, define a file named "org.apache.servicecomb.transport.rest.vertx.accesslog.parser.VertxRestAccessLogItemMeta" and fill in the full class name of the meta class defined in the previous step. In this file, the Parser loads the meta class.

1. Configure Access Log pattern

The configuration pattern in the microservice.yaml file is assumed to be "%{test-config}user-defined". The running service triggers the Access Log to print. If the request start time is 1, you can see that the Access Log print content is "user- Defined-[test-config]-[1]".

## Sample code

### Configurations in microservice.yaml

```yaml
## other configurations omitted
servicecomb:
  accesslog:
    enabled: true  ## 启用access log
    pattern: "%h - - %t %r %s %B"  ## 自定义日志格式
```

### Configurations in log4j.properties

```properties
# access log configuration item
paas.logs.accesslog.dir=../logs/
paas.logs.accesslog.file=access.log
# access log File appender
log4j.appender.access.MaxBackupIndex=10
log4j.appender.access.MaxFileSize=20MB
log4j.appender.access.logPermission=rw-------
```