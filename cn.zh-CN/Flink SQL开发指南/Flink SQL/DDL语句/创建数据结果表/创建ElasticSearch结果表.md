# 创建ElasticSearch结果表 {#concept_94716_zh .concept}

本文为您介绍如何创建实时计算ElasticSearch（ES）结果表。

**说明：** 本文仅适用于实时计算3.2.2及以上版本。

## DDL定义 {#section_ls1_qhg_cgb .section}

ElasticSearch结果表的实现使用REST API，理论上兼容ElasticSearch的各个版本。实时计算支持使用ES作为结果输出。示例代码如下。

``` {#codeblock_n9q_a2n_3j8 .language-sql}
 CREATE TABLE es_stream_sink(
  field1 LONG,
  field2 VARBINARY,
  field3 VARCHAR,
  PRIMARY KEY(field1)
)WIHT(
  type ='elasticsearch',
  endPoint = 'http://es-cn-mp****.public.elasticsearch.aliyuncs.com:****',
  accessId = '<yourAccessId>',
  accessKey = '<yourAccessSecret>',
  index = '<yourIndex>',
  typeName = '<yourTypeName>'
);
```

**说明：** 

-   ES支持根据PRIMARY KEY进行UPDATE，且PRIMARY KEY只能为1个字段。
-   指定PRIMARY KEY后，Document的ID为PRIMARY KEY字段的值。
-   未指定PRIMARY KEY的Document对应的ID为随机，详情请参见[Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)。
-   `full`更新模式下，新增的`doc`会完全覆盖已存在的`doc`。
-   `inc`更新模式下，会依据输入的字段值更新对应的字段。
-   所有的更新默认为UPSERT语义，即INSERT或UPDATE。

## WITH参数 {#section_xgs_43g_cgb .section}

通用配置:

|参数|注释说明|默认值|Required|
|--|----|---|--------|
|endPoint|server地址，例入：http://127.0.0.1:9211。|无|是|
|accessId|访问实例ID。 **说明：** 如果您通过Kibana插件操作ES，请填写Kibana登录ID。

 |无|是|
|accessKey|访问实例密钥。 **说明：** 如果您通过Kibana插件操作ES，请填写Kibana登录密码。

 |无|是|
|index|索引名称，类似于数据库Database的名称。|无|是|
|typeName|Type名称，类似于数据库的Table名称。|无|是|
|bufferSize|流入多少条数据后开始去重。|1000|否|
|maxRetryTimes|异常重试次数。|30|否|
|timeout|读超时，单位为毫秒。|600000|否|
|discovery|是否开启节点发现。如果开启，客户端每5分钟刷新一次Server List。|false|否|
|compression|是否使用GZIP压缩Request Bodies。|true|否|
|multiThread|是否开启JestClient多线程。|true|否|
|ignoreWriteError|是否忽略写入异常。|false|否|
|settings|创建Index的Settings配置。|无|否|
|updateMode|指定主键（PRIMARY KEY）后的更新模式。|full **说明：** 

-   full：全量覆盖
-   inc：增量更新

 |否|

## 动态索引相关WITH参数 {#section_pyd_qms_jgb .section}

|参数|注释说明|默认值|Required|
|dynamicIndex|是否开启动态索引|false\(true/false\)|否。|
|indexField|抽取索引的字段名|无|dynamicIndex为true时必填，只支持timestamp（以秒为单位的）、date和long3种数据类型。|
|indexInterval|切换索引的周期|d|dynamicIndex为true时必填 ，可选参数值如下： -   d：天
-   m：月
-   w：周

 |

**说明：** 

-   仅实时计算2.2.7及以上版本支持动态索引功能。
-   当开启动态索引后，基本配置中的`index`名称会作为后续创建索引的统一Alias，Alias和索引为一对多关系。
-   不同的`indexInterval`对应的真实索引名称：
    -   d -\> Alias + "yyyyMMdd"
    -   m -\> Alias + "yyyyMM"
    -   w -\> Alias + "yyyyMMW"
-   对于单个的真实索引可使用Index API进行修改，但对于Alias只支持`get`功能。若需要更新Alias，请参见[Index Aliases](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html)。

