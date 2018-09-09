## REST over Vertx
### Configuration

The REST over Vertx communication channel runs in standalone mode, it can be started in the main function. In the main function, you need to initialize logs and load service configuration. The code is as follow:

```java
import org.apache.servicecomb.foundation.common.utils.BeanUtils;
import org.apache.servicecomb.foundation.common.utils.Log4jUtils;

public class MainServer {
  public static void main(String[] args) throws Exception {
  　Log4jUtils.init();//Log initialization
  　BeanUtils.init(); // Spring bean initialization
  }
}
```

To use the REST over Vertx communication channel, add the following dependencies in the maven pom.xml file:

```xml
<dependency>
　　<groupId>org.apache.servicecomb</groupId>
　　<artifactId>transport-rest-vertx</artifactId>
</dependency>
```

Configuration items that need to be set in the microservice.yaml file are described as follows:

Table 1-1 Configuration items for REST over Vertx

| Configuration Item                                 | Default Value | Value Range | Required | Description                                                  | Remark                                                       |
| :------------------------------------------------- | :------------ | :---------- | :------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| servicecomb.rest.address                           | 0.0.0.0:8080  | -           | No       | The service listening address                                | Only for service providers                                   |
| servicecomb.rest.server.thread-count               | 1             | -           | No       | The number of server threads                                 | Only for service providers                                   |
| servicecomb.rest.client.thread-count               | 1             | -           | No       | The max number of allowed client connections                 | Only for service providers                                   |
| servicecomb.rest.client.connection-pool-per-thread | 1             | -           | No       | Specifies the number of connection pools in each client thread. | Only service consumers require this parameter.               |
| servicecomb.request.timeout                        | 30000         | -           | No       | Specifies the request timeout duration.                      |                                                              |
| servicecomb.references.\[服务名\].transport        | rest          |             | No       | Specifies the accessed transport type.                       | Only service consumers require this parameter.               |
| servicecomb.references.\[服务名\].version-rule     | latest        | -           | No       | Specifies the version of the accessed instance.              | Only service consumers require this parameter. You can set it to latest, a version range such as 1.0.0+ or 1.0.0-2.0.2, or a specific version number. For details, see the API description of the service center. |

### Sample Code

An example of the configuration in the microservice.yaml file for REST over Vertx:

```yaml
servicecomb:
  rest:
    address: 0.0.0.0:8080
    thread-count: 1
  references:
    hello:
      transport: rest
      version-rule: 0.0.1
```

