# 数据导入导出管理

eKuiper REST api 允许您导入导出当前的所有数据。

## 数据格式

导入导出数据的文件格式为 JSON, 包含流 `stream`，表 `table`， 规则 `rule`，插件 `plugin`，源配置 `source yaml` 等。每种类型保存名字和创建语句的键值对。在以下示例文件中，我们定义了流、规则、表、插件、源配置、目标动作配置。

```json
{
    "streams": {
        "demo": "CREATE STREAM demo () WITH (DATASOURCE=\"users\", FORMAT=\"JSON\")"
    },
    "tables": {
      "T110":"\n CREATE TABLE T110\n (\n S1 string\n )\n WITH (DATASOURCE=\"test.json\", FORMAT=\"json\", TYPE=\"file\", KIND=\"scan\", );\n "
    },
    "rules": {
        "rule1": "{\"id\": \"rule1\",\"sql\": \"SELECT * FROM demo\",\"actions\": [{\"log\": {}}]}",
        "rule2": "{\"id\": \"rule2\",\"sql\": \"SELECT * FROM demo\",\"actions\": [{  \"log\": {}}]}"
    },
    "nativePlugins":{
        "sinks_file":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sinks/file_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sinks/file_amd64.zip: no such file or directory",
        "sinks_tdengine":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sinks/tdengine_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sinks/tdengine_amd64.zip: no such file or directory",
        "sources_random":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sources/random_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sources/random_amd64.zip: no such file or directory"
    },
    "portablePlugins":{
    },
    "sourceConfig":{
      "mqtt":"{\"td\":{\"insecureSkipVerify\":false,\"password\":\"public\",\"protocolVersion\":\"3.1.1\",\"qos\":1,\"server\":\"tcp://broker.emqx.io:1883\",\"username\":\"admin\"},\"test\":{\"insecureSkipVerify\":false,\"password\":\"public\",\"protocolVersion\":\"3.1.1\",\"qos\":1,\"server\":\"tcp://127.0.0.1:1883\",\"username\":\"admin\"}}"
    },
    "sinkConfig":{
      "edgex":"{\"test\":{\"bufferLength\":1024,\"contentType\":\"application/json\",\"enableCache\":false,\"format\":\"json\",\"messageType\":\"event\",\"omitIfEmpty\":false,\"port\":6379,\"protocol\":\"redis\",\"runAsync\":false,\"sendSingle\":true,\"server\":\"localhost\",\"topic\":\"application\",\"type\":\"redis\"}}"
    },
    "connectionConfig":{
    },
    "Service":{
    },
    "Schema":{
    }
}
```

## 导入数据

该 API 接受数据并将其导入系统中。若已有历史遗留数据，则首先清除旧有数据，然后导入。 API 支持通过文本内容或者文件 URI 的方式指定数据。

示例1：通过文本内容导入


```shell
POST http://{{host}}/data/import
Content-Type: application/json

{
  "content": "$数据 json 内容"
}
```

示例2：通过文件 URI 导入

```shell
POST http://{{host}}/data/import
Content-Type: application/json

{
  "file": "file:///tmp/a.json"
}
```

示例3：通过文件 URI 导入数据并退出 (用于插件和静态模式更新,用户需保证 eKuiper 退出后能重新启动)

```shell
POST http://{{host}}/data/import?stop=1
Content-Type: application/json

{
  "file": "file:///tmp/a.json"
}
```

## 导入数据状态查询

该 API 返回数据导入出错情况，如所有返回为空，则代表导入完全成功。

```shell
GET http://{{host}}/data/import/status
```

示例1：导入数据完全成功

```shell
GET http://{{host}}/data/import/status
Content-Type: application/json

{
  "streams":{},
  "tables":{},
  "rules":{},
  "nativePlugins":{},
  "portablePlugins":{},
  "sourceConfig":{},
  "sinkConfig":{},
  "connectionConfig":{},
  "Service":{},
  "Schema":{}}
```

示例2：导入插件失败

```shell
GET http://{{host}}/data/import/status
Content-Type: application/json

{
  "streams":{},
  "tables":{},
  "rules":{},
  "nativePlugins":{
    "sinks_file":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sinks/file_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sinks/file_amd64.zip: no such file or directory",
    "sinks_tdengine":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sinks/tdengine_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sinks/tdengine_amd64.zip: no such file or directory",
    "sources_random":"fail to download file file:///root/ekuiper-jran/_plugins/ubuntu/sources/random_amd64.zip: stat /root/ekuiper-jran/_plugins/ubuntu/sources/random_amd64.zip: no such file or directory"},
  "portablePlugins":{},
  "sourceConfig":{},
  "sinkConfig":{},
  "connectionConfig":{},
  "Service":{},
  "Schema":{}}
```

## 导出数据

导出 API 返回二进制流，在浏览器使用时，可选择下载保存的文件路径。

```shell
GET http://{{host}}/data/export
```