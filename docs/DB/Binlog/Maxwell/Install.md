# Maxwell 安装步骤

## 1.下载

```http
https://github.com/zendesk/maxwell
```

下载最新Releases版本，然后解压。

## 2.修改配置文件

### 修改应用启动配置文件

```shell
cd maxwell-home/
cp config.properties.example config.properties
vim config.properties
```

修改内容如下：

```ini
# tl;dr config
log_level=info

producer=kafka
#kafka.bootstrap.servers=kafka0:9092,kafka1:9092,kafka2:9092
# kafka服务端配置，即将binlog数据发送到的目的地
kafka.bootstrap.servers=10.0.29.165:9092

# mysql login info
# maxwell配置信息表，会初始化一些表和数据，主要是采集下载binlog的服务端表和数据信息
host=10.0.30.59
port=3306
user=maxwell_app
password=123123


#     *** general ***
# choose where to produce data to. stdout|file|kafka|kinesis|pubsub|sqs|rabbitmq|redis
#producer=kafka

# set the log level.  note that you can configure things further in log4j2.xml
#log_level=DEBUG # [DEBUG, INFO, WARN, ERROR]

# if set, maxwell will look up the scoped environment variables, strip off the prefix and inject the configs
#env_config_prefix=MAXWELL_

#     *** mysql ***

# mysql host to connect to
#host=hostname

# mysql port to connect to
#port=3306

# mysql user to connect as.  This user must have REPLICATION SLAVE permissions,
# as well as full access to the `maxwell` (or schema_database) database
#user=maxwell

# mysql password
#password=maxwell

# options to pass into the jdbc connection, given as opt=val&opt2=val2
#jdbc_options=opt1=100&opt2=hello

# name of the mysql database where maxwell keeps its own state
# 将要下载binlog的服务端数据库名称
schema_database=maxwell_fsdb

# whether to use GTID or not for positioning
#gtid_mode=true

# maxwell will capture an initial "base" schema containing all table and column information,
# and then keep delta-updates on top of that schema.  If you have an inordinate amount of DDL changes,
# the table containing delta changes will grow unbounded (and possibly too large) over time.  If you
# enable this option Maxwell will periodically compact its tables.
#max_schemas=10000

# SSL/TLS options
# To use VERIFY_CA or VERIFY_IDENTITY, you must set the trust store with Java opts:
#   -Djavax.net.ssl.trustStore=<truststore> -Djavax.net.ssl.trustStorePassword=<password>
# or import the MySQL cert into the global Java cacerts.
# MODE must be one of DISABLED, PREFERRED, REQUIRED, VERIFY_CA, or VERIFY_IDENTITY
#
# turns on ssl for the maxwell-store connection, other connections inherit this setting unless specified
#ssl=DISABLED
# for binlog-connector
#replication_ssl=DISABLED
# for the schema-capture connection, if used
#schema_ssl=DISABLED

# maxwell can optionally replicate from a different server than where it stores
# schema and binlog position info.  Specify that different server here:
# 下载binlog的服务端数据库配置
replication_host=10.0.29.5
replication_user=platform_app
replication_password=yV2x60A3
replication_port=3306

jdbc_options= useSSL=false&serverTimezone=Asia/Shanghai
# Specifies that Maxwell should capture schema from a different server than
# it replicates from:
# 备库，下载binlog的服务端备库，这里设置为同一个
schema_host=10.0.29.5
schema_user=platform_app
schema_password=yV2x60A3
schema_port=3306


#       *** output format ***

# records include binlog position (default false)
output_binlog_position=true

# records include a gtid string (default false)
#output_gtid_position=true

# records include fields with null values (default true).  If this is false,
# fields where the value is null will be omitted entirely from output.
output_nulls=true

# records include server_id (default false)
output_server_id=true

# records include thread_id (default false)
output_thread_id=true

# records include schema_id (default false)
output_schema_id=true

# records include row query, binlog option "binlog_rows_query_log_events" must be enabled" (default false)
output_row_query=true

# DML records include list of values that make up a row's primary key (default false)
output_primary_keys=true

# DML records include list of columns that make up a row's primary key (default false)
output_primary_key_columns=true

# records include commit and xid (default true)
output_commit_info=true

# This controls whether maxwell will output JSON information containing
# DDL (ALTER/CREATE TABLE/ETC) infromation. (default: false)
# See also: ddl_kafka_topic
output_ddl=true

# turns underscore naming style of fields to camel case style in JSON output
# default is none, which means the field name in JSON is the exact name in MySQL table
#output_naming_strategy=underscore_to_camelcase

#       *** kafka ***

# list of kafka brokers
#kafka.bootstrap.servers=hosta:9092,hostb:9092

# kafka topic to write to
# this can be static, e.g. 'maxwell', or dynamic, e.g. namespace_%{database}_%{table}
# in the latter case 'database' and 'table' will be replaced with the values for the row being processed
# 自动创建topic的规则，前缀+数据库名+表名
kafka_topic=maxwell_binlog_%{database}_%{table}

# alternative kafka topic to write DDL (alter/create/drop) to.  Defaults to kafka_topic
# 自动创建ddl语句对应的topic名称，不能使用上面的规则，需固定指定一个topic
ddl_kafka_topic=maxwell_binlog_fsdb_ddl

# hash function to use.  "default" is just the JVM's 'hashCode' function.
# 数据分发到partations的规则，这里使用按key hash分发，避免partations数据倾斜
kafka_partition_hash=murmur3 # [default, murmur3]

# how maxwell writes its kafka key.
#
# 'hash' looks like:
# {"database":"test","table":"tickets","pk.id":10001}
#
# 'array' looks like:
# ["test","tickets",[{"id":10001}]]
#
# default: "hash"
#kafka_key_format=hash # [hash, array]

# extra kafka options.  Anything prefixed "kafka." will get
# passed directly into the kafka-producer's config.

# a few defaults.
# These are 0.11-specific. They may or may not work with other versions.
kafka.compression.type=snappy
kafka.retries=0
kafka.acks=1
#kafka.batch.size=16384


# kafka+SSL example
# kafka.security.protocol=SSL
# kafka.ssl.truststore.location=/var/private/ssl/kafka.client.truststore.jks
# kafka.ssl.truststore.password=test1234
# kafka.ssl.keystore.location=/var/private/ssl/kafka.client.keystore.jks
# kafka.ssl.keystore.password=test1234
# kafka.ssl.key.password=test1234#

# controls a heuristic check that maxwell may use to detect messages that
# we never heard back from.  The heuristic check looks for "stuck" messages, and
# will timeout maxwell after this many milliseconds.
#
# See https://github.com/zendesk/maxwell/blob/master/src/main/java/com/zendesk/maxwell/producer/InflightMessageList.java
# if you really want to get into it.
#producer_ack_timeout=120000 # default 0


#           *** partitioning ***

# What part of the data do we partition by?
# 数据分发的规则，安装主键分发
producer_partition_by=primary_key # [database, table, primary_key, transaction_id, thread_id, column]

# specify what fields to partition by when using producer_partition_by=column
# column separated list.
#producer_partition_columns=id,foo,bar

# when using producer_partition_by=column, partition by this when
# the specified column(s) don't exist.
#producer_partition_by_fallback=database

#            *** kinesis ***

#kinesis_stream=maxwell

# AWS places a 256 unicode character limit on the max key length of a record
# http://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html
#
# Setting this option to true enables hashing the key with the md5 algorithm
# before we send it to kinesis so all the keys work within the key size limit.
# Values: true, false
# Default: false
#kinesis_md5_keys=true

#            *** sqs ***

#sqs_queue_uri=aws_sqs_queue_uri

# The sqs producer will need aws credentials configured in the default
# root folder and file format. Please check below link on how to do it.
# http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html

#            *** pub/sub ***

#pubsub_project_id=maxwell
#pubsub_topic=maxwell
#ddl_pubsub_topic=maxwell_ddl

#            *** rabbit-mq ***

#rabbitmq_host=rabbitmq_hostname
#rabbitmq_port=5672
#rabbitmq_user=guest
#rabbitmq_pass=guest
#rabbitmq_virtual_host=/
#rabbitmq_exchange=maxwell
#rabbitmq_exchange_type=fanout
#rabbitmq_exchange_durable=false
#rabbitmq_exchange_autodelete=false
#rabbitmq_routing_key_template=%db%.%table%
#rabbitmq_message_persistent=false
#rabbitmq_declare_exchange=true

#           *** redis ***

#redis_host=redis_host
#redis_port=6379
#redis_auth=redis_auth
#redis_database=0

# name of pubsub/list/whatever key to publish to
#redis_key=maxwell

# this can be static, e.g. 'maxwell', or dynamic, e.g. namespace_%{database}_%{table}
#redis_pub_channel=maxwell
# this can be static, e.g. 'maxwell', or dynamic, e.g. namespace_%{database}_%{table}
#redis_list_key=maxwell
# this can be static, e.g. 'maxwell', or dynamic, e.g. namespace_%{database}_%{table}
# Valid values for redis_type = pubsub|lpush. Defaults to pubsub

#redis_type=pubsub

#           *** custom producer ***

# the fully qualified class name for custom ProducerFactory
# see the following link for more details.
# http://maxwells-daemon.io/producers/#custom-producer
#custom_producer.factory=

# custom producer properties can be configured using the custom_producer.* property namespace
#custom_producer.custom_prop=foo

#          *** filtering ***

# filter rows out of Maxwell's output.  Command separated list of filter-rules, evaluated in sequence.
# A filter rule is:
#  <type> ":" <db> "." <tbl> [ "." <col> "=" <col_val> ]
#  type    ::= [ "include" | "exclude" | "blacklist" ]
#  db      ::= [ "/regexp/" | "string" | "`string`" | "*" ]
#  tbl     ::= [ "/regexp/" | "string" | "`string`" | "*" ]
#  col_val ::= "column_name"
#  tbl     ::= [ "/regexp/" | "string" | "`string`" | "*" ]
#
# See http://maxwells-daemon.io/filtering for more details
# 过滤规则，哪些表或字段不需要发送到kafka，或者哪些表或字段必须发送到kafka
#filter= exclude: *.*, include: foo.*, include: bar.baz, include: foo.bar.col_eg = "value_to_match"
#filter= include: winddb./ipo+/
#filter= include: winddb.ashareipo,include: winddb.asharecniindustriesclass,winddb.hksharedescription
#filter= exclude: *.*, include: winddb./ipo+/
filter= exclude: *.*, include: fsdb.*
#filter= exclude: *.*, include: winddb./ashare+/
#filter= exclude: *.*, include: winddb./aindex+/, include: winddb./ashare+/, include: winddb./cbindexeodprices+/, include: winddb./cbond+/, include: winddb./ccbond+/, include: winddb./cdr+/, include: winddb./china+/, include: winddb./cmf+/, include: winddb./compintroduction+/, include: winddb./etfpchredm+/, include: winddb./fund+/, include: winddb./fxrmbmidrate+/, include: winddb./hbzqdziopv+/, include: winddb./hk+/, include: winddb./ipo+/, include: winddb./lofpchredm+/, include: winddb./neeq+/, include: winddb./ralatedsecuritiescode+/, include: winddb./scf+/, include: winddb./shs+/, include: winddb./sindexperformance+/, include: winddb./windcustomcode+/
#filter= include: winddb.ashareipo
# javascript filter
# maxwell can run a bit of javascript for each row if you need very custom filtering/data munging.
# See http://maxwells-daemon.io/filtering/#javascript_filters for more details
#
#javascript=/path/to/javascript_filter_file

#       *** encryption ***

# Encryption mode. Possible values are none, data, and all. (default none)
#encrypt=none

# Specify the secret key to be used
#secret_key=RandomInitVector

#       *** monitoring ***

# Maxwell collects metrics via dropwizard. These can be exposed through the
# base logging mechanism (slf4j), JMX, HTTP or pushed to Datadog.
# Options: [jmx, slf4j, http, datadog]
# Supplying multiple is allowed.
#metrics_type=jmx,slf4j

# The prefix maxwell will apply to all metrics
#metrics_prefix=MaxwellMetrics # default MaxwellMetrics

# Enable (dropwizard) JVM metrics, default false
#metrics_jvm=true

# When metrics_type includes slf4j this is the frequency metrics are emitted to the log, in seconds
#metrics_slf4j_interval=60

# When metrics_type includes http or diagnostic is enabled, this is the port the server will bind to.
#http_port=8080

# When metrics_type includes http or diagnostic is enabled, this is the http path prefix, default /.
#http_path_prefix=/some/path/

# ** The following are Datadog specific. **
# When metrics_type includes datadog this is the way metrics will be reported.
# Options: [udp, http]
# Supplying multiple is not allowed.
#metrics_datadog_type=udp

# datadog tags that should be supplied
#metrics_datadog_tags=tag1:value1,tag2:value2

# The frequency metrics are pushed to datadog, in seconds
#metrics_datadog_interval=60

# required if metrics_datadog_type = http
#metrics_datadog_apikey=API_KEY

# required if metrics_datadog_type = udp
#metrics_datadog_host=localhost # default localhost
#metrics_datadog_port=8125 # default 8125

# Maxwell exposes http diagnostic endpoint to check below in parallel:
# 1. binlog replication lag
# 2. producer (currently kafka) lag

# To enable Maxwell diagnostic
#http_diagnostic=true # default false

# Diagnostic check timeout in milliseconds, required if diagnostic = true
#http_diagnostic_timeout=10000 # default 10000

#    *** misc ***

# maxwell's bootstrapping functionality has a couple of modes.
#
# In "async" mode, maxwell will output the replication stream while it
# simultaneously outputs the database to the topic.  Note that it won't
# output replication data for any tables it is currently bootstrapping -- this
# data will be buffered and output after the bootstrap is complete.
#
# In "sync" mode, maxwell stops the replication stream while it
# outputs bootstrap data.
#
# async mode keeps ops live while bootstrapping, but carries the possibility of
# data loss (due to buffering transactions).  sync mode is safer but you
# have to stop replication.
bootstrapper=async
# [sync, async, none]

# output filename when using the "file" producer
#output_file=/path/to/file
# 这个要设置一下，避免多个server_id相同，导致和mysql服务端通讯失败
replica_server_id=295

```

### 修改日志配置文件

```sh
cp log4j2.xml log4jx.xml.bak
vim log4j2.xml
```

内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
 <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
 <!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
 <!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
 <configuration status="WARN" monitorInterval="30">
     <!--先定义所有的appender-->
     <appenders>
     <!--这个输出控制台的配置-->
         <console name="Console" target="SYSTEM_OUT">
         <!--输出日志的格式-->
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
         </console>
        
     <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
         <RollingFile name="RollingFileInfo" fileName="/data/maxwell-winddb/logs/info.log"
                      filePattern="/data/maxwell-winddb/logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
             <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
             <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
         <RollingFile name="RollingFileWarn" fileName="/data/maxwell-winddb/logs/warn.log"
                      filePattern="/data/maxwell-winddb/logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
             <DefaultRolloverStrategy max="20"/>
         </RollingFile>
         <RollingFile name="RollingFileError" fileName="/data/maxwell-winddb/logs/error.log"
                      filePattern="/data/maxwell-winddb/logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
     </appenders>
     <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
     <loggers>
         <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
         <logger name="org.springframework" level="INFO"></logger>
         <logger name="org.mybatis" level="INFO"></logger>
         <root level="all">
             <appender-ref ref="Console"/>
             <appender-ref ref="RollingFileInfo"/>
             <appender-ref ref="RollingFileWarn"/>
             <appender-ref ref="RollingFileError"/>
         </root>
     </loggers>
 </configuration>
```

## 3.启动应用

编写启动脚本

```sh
vim maxwell
```

内容如下：

```shell
#!/bin/bash
#需要启动的应用名，一般包含在下面启动命令的路径里面，这样更容易辨识出应用
APP_NAME='maxwell-winddb'
#启动命令
startComment='/data/maxwell-winddb/bin/maxwell --config /data/maxwell-winddb/config.properties'

#使用说明，用来提示输入参数，如果输出其他不相关的命令，会提示正确命令
usage() {
    #使用具体命令 例如 sh core.sh start 启动应用程序
    echo "Usage: sh maxwell [start|stop|restart|status]"
    exit 1
}
 
#检查程序是否在运行
is_exist(){
  #查看具体应用的进程号，并且进行记录 grep -v grep 过滤grep
  pid=`ps -ef|grep $APP_NAME|grep -v grep|awk '{print $2}'`
  #如果不存在返回1，存在返回0
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}

#启动方法
start(){
  #调用检查应用方法
  is_exist
  if [ $? -eq 0 ]; then
    echo "${APP_NAME} is already running. pid=${pid}"
  else
     #启动Java应用，如果需要后台运行则使用 nohup
     nohup $startComment > /dev/null 2>&1 &
     sleep 2
     echo "${APP_NAME} is running"
  fi
}

#停止方法
stop(){
  is_exist
  if [ $? -eq "0" ]; then
    #杀死进程
    kill -9 $pid
    sleep 2
    echo "${APP_NAME} is stop"
  else
    echo "${APP_NAME} is not running"
  fi
}

#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is running. Pid is ${pid}"
  else
    echo "${APP_NAME} is not running."
  fi
}

#重启
restart(){
  stop
  sleep 5
  start
}

#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac
```

```shell
chmod 755 maxwell
```

如果启动异常，可以查看相关日志提示来定位问题，并解决。

最终的目录结构如下

```shell
[root@localhost maxwell-1.37.0]# ll
total 108
drwxr-xr-x 2 root root  4096 Mar  2 11:50 bin
-rw-r--r-- 1 root root 24974 Mar  2 11:50 config.md
-rw-r--r-- 1 root root 13065 Mar  2 11:50 config.properties
-rw-r--r-- 1 root root 11970 Mar  2 11:50 config.properties.example
-rw-r--r-- 1 root root 10259 Mar  2 11:50 kinesis-producer-library.properties.example
drwxr-xr-x 3 root root 12288 Mar  2 11:50 lib
-rw-r--r-- 1 root root   548 Mar  2 11:50 LICENSE
-rw-r--r-- 1 root root   641 Mar  2 14:59 log4j2.xml
drwxr-xr-x 2 root root  4096 Mar  2 11:54 logs
-rwxr-xr-x 1 root root  1719 Mar  2 11:53 maxwell
-rw-r--r-- 1 root root  3182 Mar  2 11:50 quickstart.md
-rw-r--r-- 1 root root  1429 Mar  2 11:50 README.md

```





