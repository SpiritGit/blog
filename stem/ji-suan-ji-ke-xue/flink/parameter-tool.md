# Parameter Tool

在`flink`实践中，经常使用到大量配置参数，如`kafka`地址，`topic`名称等，常用做法是，将配置写在专门的配置文件中，如`.properties`，并打在`jar`包被调用。

对于配置参数相对固定的任务，以上做法简单高效，但是当任务配置参数频繁变动，或本身就是通用任务，如`kafka`转发程序，其地址和`topic`都不固定，以上做法显然不再合适，因此本文使用`flink`自带的`ParameterTool`动态指定`flink job`的配置参数。

### 通过命令行指定参数

如一`kafka`转发程序，通常需要动态指定`topic, source_address, target_address, jobname, groupid`，命令行代码示例如下

```bash
flink run kafka_forward.jar -topic microvideo_trans_data_gantry_free_topic -source basic-mom-kafka-node1.daw.localdomain:9092,basic-mom-kafka-node2.daw.localdomain:9092,basic-mom-kafka-node3.daw.localdomain:9092 -target 192.168.0.2:9092,192.168.0.3:9092,192.168.0.4:9092 -group mvlab -jobname toll_main_source -parallel 4
```

`flink job`内解析参数的代码如下

```java
import org.apache.flink.api.java.utils.ParameterTool;

public static void main(String[] args) throws Exception {

    ParameterTool parameter = ParameterTool.fromArgs(args);

    String topic = parameter.get("topic");
    String addressIn = parameter.get("source");
    String addressOut = parameter.get("target");
    String groupName = parameter.get("group");
    String jobName = `parameter.get("jobname","nubility job"); // 可设置默认值
    int parallel = parameter.getInt("parallel",1); // 可设置默认值
    // 后续代码...
}
```

注意，以上`parameter.get`方法或`parameter.getXXX`方法均可设置默认值，即在不指定该参数时的备用值，用法见上述代码。

### 读取.properties配置文件

对于配置较多的情况，使用`.properties`作为配置文件，可避免在命令行中手动输入过多参数，读取代码如下。

```java
import org.apache.flink.api.java.utils.ParameterTool;

public static void main(String[] args) throws Exception {

    String propertiesFile = "/Users/spirit/dev.properties";
    ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFile);
    // parameter.get方法参见上文
}
```

其中，`.properties`内参数形式如下

```properties
topic=microvideo_trans_data_gantry_free_topic 

source=basic-mom-kafka-node1.daw.localdomain:9092,basic-mom-kafka-node2.daw.localdomain:9092,basic-mom-kafka-node3.daw.localdomain:9092 

target=192.168.0.2:9092,192.168.0.3:9092,192.168.0.4:9092 

group=mvlab 

jobname=toll_main_source 

parallel=4
```

以上就是本文对`ParameterTool`的简单介绍，欢迎采纳与指正。
