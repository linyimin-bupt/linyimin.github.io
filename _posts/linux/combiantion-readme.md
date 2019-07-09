---
layout: post
title: Ubuntu apt-get update和apt-get upgrade的区别
categories: [Linux]
---

# api-combination

测试链接:[http://182.92.222.75:50000/](http://182.92.222.75:50000/) 

## 启动

注: (运行mongo和运行redis可以直接运行start-database.sh脚本完成.)

1. 运行mongo

- 在本地创建相关文件夹

```bash
cd ~
mkdir -p docker/mongo/data docker/mongo/conf
```

- 添加配置

```bash

cd  ~/docker/mongo/conf

cat << EOF > mongod.conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
EOF
```

- 运行mongo

```bash
cd ~/docker/mongo/
docker run -p 27017:27017 -v $PWD/data:/data -v $PWD/conf/:/etc/mongo --name mymongo --restart always -d mongo
```

2. 运行redis

- 在本地创建相关文件夹

```bash
cd ~
mkdir -p docker/reids/data docker/redis/conf
```

- 添加配置文件

```bash
cd ~/docker/redis/conf/

cat << EOF > redis.conf

# bind 127.0.0.1

protected-mode no
port 6379
tcp-backlog 511

timeout 0

tcp-keepalive 300

daemonize no

supervised no

pidfile /var/run/redis_6379.pid

loglevel notice

logfile "access.log"

databases 16

always-show-logo yes

save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./

slave-serve-stale-data yes

slave-read-only yes

repl-diskless-sync no

repl-diskless-sync-delay 5

repl-disable-tcp-nodelay no

slave-priority 100

# 设置密码
requirepass 123456

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no

appendonly yes

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-use-rdb-preamble no

slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128

latency-monitor-threshold 0

notify-keyspace-events ""

hash-max-ziplist-entries 512
hash-max-ziplist-value 64

list-max-ziplist-size -2

list-compress-depth 0

set-max-intset-entries 512

zset-max-ziplist-entries 128
zset-max-ziplist-value 64

hll-sparse-max-bytes 3000

activerehashing yes

client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

hz 10

aof-rewrite-incremental-fsync yes
EOF
```

- 运行redis

```bash
cd ~/docker/redis/
docker run -p 6379:6379 -v $PWD/data:/data -v $PWD/conf/redis.conf:/etc/redis/redis.conf --privileged=true --name myredis --restart=always -d redis redis-server /etc/redis/redis.conf
```

3. 修改resource/application.properties下的配置

4. 打包项目(在项目根目录下运行)

```bash
mvn clean package
```

5. 构建镜像(在项目根目录下运行)

```bash
docker build -t api-combiantion .
```

6. 运行项目

```bash
docker run -p 5000:8080 api-combination
```

## 测试例子


### 3个原子API


1. http://182.92.222.75:8001/send

接收一个简体中文, 返回一个json串

```bash
curl http://182.92.222.75:8001/send?text=国
```

响应

```bash
{"text": "国"}
```

2. http://www.webxml.com.cn/WebServices/TraditionalSimplifiedWebService.asmx

soap协议: 发送一个简体中文, 返回一个繁体中文

发送参数:
```xml
<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <toTraditionalChinese xmlns="http://webxml.com.cn/">
      <sText>国</sText>
    </toTraditionalChinese>
  </soap12:Body>
</soap12:Envelope>
```

响应:

```
<?xml version="1.0" encoding="utf-8" ?>
<soap:Envelopexmlns:soap="http://www.w3.org/2003/05/soap-envelope"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <soap:Body>
        <toTraditionalChineseResponsexmlns="http://webxml.com.cn/">
            <toTraditionalChineseResult>國</toTraditionalChineseResult>
        </toTraditionalChineseResponse>
    </soap:Body>
</soap:Envelope>
```

3. http://182.92.222.75:8001/transfer

请求参数:

```
<?xml version="1.0" encoding="utf-8" ?>
<soap:Envelopexmlns:soap="http://www.w3.org/2003/05/soap-envelope"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <soap:Body>
        <toTraditionalChineseResponsexmlns="http://webxml.com.cn/">
            <toTraditionalChineseResult>國</toTraditionalChineseResult>
        </toTraditionalChineseResponse>
    </soap:Body>
</soap:Envelope>
```

响应值: json

```json
{"text": "國"}
```

### 服务注册中心返回的原子API信息

http://182.92.222.75:8001/queryServicesInDataCenterWithDetail

返回的数据类型:

```
[
  {
    Value: {
      id: '1',
      name: 'send',
      argument: 'text: stirng',
      response: '{text: string}',
      method: 'get',
      type: '外呼类',
      status: '已上线',
      atomUrl: '/send',
      des: '首页',
      port: '8001',
      address: '182.92.222.75',
      contentType: 'text/plain',
    }
  }, {
    Value: {
      id: '2',
      name: 'TraditionalSimplifiedWebService',
      argument: `
      <?xml version="1.0" encoding="utf-8"?>
      <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
          <toTraditionalChinese xmlns="http://webxml.com.cn/">
            <sText>string</sText>
          </toTraditionalChinese>
        </soap12:Body>
      </soap12:Envelope>
      `,
      response: `
      <?xml version="1.0" encoding="utf-8" ?>
      <soap:Envelopexmlns:soap="http://www.w3.org/2003/05/soap-envelope"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <soap:Body>
      <toTraditionalChineseResponsexmlns="http://webxml.com.cn/">
      <toTraditionalChineseResult>string</toTraditionalChineseResult>
      </toTraditionalChineseResponse>
      </soap:Body>
      </soap:Envelope>
      `,
      method: 'post',
      type: '消息类',
      status: '已上线',
      atomUrl: '/WebServices/TraditionalSimplifiedWebService.asmx',
      des: '将简体字转成繁体字',
      port: '80',
      address: 'www.webxml.com.cn',
      contentType: 'application/soap+xml; charset=utf-8',
    }
  }, {
    Value: {
      id: '3',
      name: 'transfer',
      argument: `
      <?xml version="1.0" encoding="utf-8" ?>
      <soap:Envelopexmlns:soap="http://www.w3.org/2003/05/soap-envelope"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <soap:Body>
      <toTraditionalChineseResponsexmlns="http://webxml.com.cn/">
      <toTraditionalChineseResult>string</toTraditionalChineseResult>
      </toTraditionalChineseResponse>
      </soap:Body>
      </soap:Envelope>
      `,
      response: '{text: string}',
      method: 'post',
      type: '消息类',
      status: '已上线',
      atomUrl: '/transfer',
      des: '获取json形式的繁体字',
      port: '8001',
      address: '182.92.222.75',
      contentType: 'application/soap+xml'
    }
  },
]
```

### 组合例子

1 --> 2 --> 3;

首先调用```/send```发送一个简体字, 返回一个json串, 然后将json串转成soap,传给```TraditionalSimplifiedWebService.asmx```,获取soap xml格式的繁体字结果, 最后将xml结果传给```/transfer```解析最后返回一个JSON串.


最终结果:

![](./images/final-result.png)

1. 访问首页: http://localhost:5000/

2. 拖拽外呼类, 并点击, 在右方选择```send```接口

![](./images/send.png)

设置MVEL脚本和condition脚本.其中MVEL脚本做协议的转换, condition是执行条件, 若为空,默认为可执行.

统一的传入参数格式: 

```
header: JSONObject
requestParams: String
```

- MVEL脚本(此处直接返回, 不做任何转换)

```
import com.alibaba.fastjson.JSONObject;
JSONObject result = new JSONObject();
result.put("header", header);
result.put("requestParams", requestParams);
return result;
```

- condition设为空,默认为可执行

然后点击保存, 第一个API设置完毕.

3. 拖拽消息类设置第二个API, 拖拽外呼类, 并点击, 在右方选择```TraditionalSimplifiedWebService```接口

设置相关脚本

- MVEL脚本(此处将json转成soap)

```
import com.alibaba.fastjson.JSONObject;
JSONObject result = new JSONObject();
result.put("header", header);
StringBuilder sb = new StringBuilder();
sb.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>");
sb.append("<soap12:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap12=\"http://www.w3.org/2003/05/soap-envelope\">");
sb.append("<soap12:Body>");
sb.append("<toTraditionalChinese xmlns=\"http://webxml.com.cn/\">");
JSONObject temp = JSONObject.parseObject(requestParams);
String text = temp.get("text").toString();
sb.append("<sText>" + text + "</sText>");
sb.append("</toTraditionalChinese>");
sb.append("</soap12:Body>");
sb.append("</soap12:Envelope>");
result.put("requestParams", sb.toString());
return result;
```

- condition设为空,默认为可执行

然后点击保存, 第2个API设置完毕.


4. 拖拽消息类, 并点击, 在右方选择```transfer```接口

设置MVEL脚本和condition脚本.其中MVEL脚本做协议的转换, condition是执行条件, 若为空,默认为可执行.

统一的传入参数格式: 

```
header: JSONObject
requestParams: String
```

- MVEL脚本(此处直接返回, 不做任何转换)

```
import com.alibaba.fastjson.JSONObject;
JSONObject result = new JSONObject();
result.put("header", header);
result.put("requestParams", requestParams);
return result;
```

- condition设为空,默认为可执行

然后点击保存, 第3个API设置完毕.

5. 注册API

点击功能栏的操作按钮, 选择注册

![](./images/register.png)

填写注册信息.

输出脚本用于指定返回的数据, MVEL传入参数是所有请求的结果组成的一个ArrayList<Map<name, result>>, 如果不指定,默认返回最后一个原子API的执行结果.

6. 组合API的url: http://localhsot:5000/combination/{name}

