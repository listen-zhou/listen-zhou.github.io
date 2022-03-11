# 消费由Maxwell生成的JSON数据，导入到MySQL

接前面Maxwell生成JSON格式的消息发送kafka 对应表的topic，下面新建消费者工程，消费topic数据持久化到目标MySQL数据库，中间可能涉及数据转换、清洗等操作。

## 逻辑流程图

![image-20220311155022234](./Images/Chart/kafka-consumer.jpg)

## 工程结构图

```shell
C:\kafka-related\kafka-consumer-parent>dir
2022/03/07  10:51    <DIR>          .
2022/03/07  10:51    <DIR>          ..
2022/03/04  11:22               401 .project
2022/03/04  11:22    <DIR>          .settings
2022/03/07  10:51    <DIR>          kafka-consumer-common
2022/03/07  10:51    <DIR>          kafka-consumer-fsdb-2-fbdb
2022/03/07  10:51    <DIR>          kafka-consumer-fsdb-2-fbdb-run
2022/03/07  10:51    <DIR>          kafka-consumer-tdxdb-2-fsdb
2022/03/07  10:51    <DIR>          kafka-consumer-winddb-2-fsdb
2022/03/07  10:51    <DIR>          kafka-consumer-winddb-2-fsdb-run
2022/03/07  11:07             1,956 pom.xml
2022/03/07  10:50             1,965 pom.xml.versionsBackup
2022/03/04  11:22    <DIR>          src
               3 个文件          4,322 字节                     
```

### 工程功能说明

- kafka-consumer-parent 父工程，管理所有子工程；
- kafka-consumer-common 共用工程，包含消费kafka的主逻辑代码，对应上图逻辑流程图；
- kafka-consumer-fsdb-2-fbdb 服务工程，包含从fsdb到fbdb数据处理过程中的业务服务代码，类似数据转换、清洗等逻辑，依赖kafka-consumer-common；
- kafka-consumer-fsdb-2-fbdb-run 启动工程，主要是编写启动类，配置启动类需要加载的组件等，依赖kafka-consumer-fsdb-2-fbdb， 也可以依赖类似kafka-consumer-fsdb-2-fbdb这样多个服务工程，没有业务代码，包含应用启动所需所有配置信息；
- kafka-consumer-tdxdb-2-fsdb 服务工程，包含从tdxdb到fsdb数据处理过程中的业务服务代码；
- kafka-consumer-winddb-2-fsdb 服务工程，包含从winddb到fsdb数据处理过程中的业务服务代码；
- kafka-consumer-winddb-2-fsdb-run 启动工程，类似上面的kafka-consumer-fsdb-2-fbdb-run工程，也可以依赖多个服务工程，在多数据源处理的时候可能需要依赖多个服务工程；

工程github地址：

https://github.com/listen-zhou/kafka-consumer-parent.git
