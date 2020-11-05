# ElasticSearch

## 第一章 ES简介

### 1.1 ES概述

- ElasticSearch是一个基于Lucene的**搜索服务器**。他提供了一个分布式多用户能力的**全文搜索引擎**。
- ElasticSearch是基于Java开发的开源搜索引擎
- 官网地址 
  - https://www.elastic.co/products/elasticsearch
  - https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-6-0
- 使用场景
  1. 按关键字查询的全文搜索
  2. 海量数据的处理分析



### 1.2 ES与其他数据库对比

|               | redis        | mysql           | elasticsearch                                                | hbase                                               | hadoop/hive     |
| ------------- | ------------ | --------------- | ------------------------------------------------------------ | --------------------------------------------------- | --------------- |
| 容量/容量扩展 | 低           | 中              | 较大                                                         | 海量                                                | 海量            |
| 查询时效性    | 极高         | 中等            | 较高                                                         | 中等                                                | 低              |
| 查询灵活性    | 较差 k-v模式 | 非常好，支持sql | 较好，关联查询较弱，但是可以全文检索，DSL语言可以处理过滤、匹配、排序、聚合等各种操作 | 较差，主要靠rowkey,  scan性能较差，需要建立二级索引 | 非常好，支持sql |
| 写入速度      | 极快         | 中等            | 较快                                                         | 较快                                                | 慢              |
| 一致性、事务  | 弱           | 强              | 弱                                                           | 弱                                                  | 弱              |



### 1.3 ES特点

1. 默认采用分片+分布式集群存储数据
   - 在es中，一份完整的数据（表）根据主键hash被切分为多个shard，分布式地存储在集群的多个节点中，分片数默认为5（6.x版本，7.x版本后为1），每个索引可分别自定义shard数量
   - es具有副本机制实现高可用，对于每一个shard，都有一或多个副本存储在集群中，并自动将每个shard和其副本分配到不同的节点中存储
2. 全索引
   - es默认为表中的所有字段都创建了索引，采用的是**倒排索引**（字段→主键）
   - 记录存储到es时，es会对数据中的字段内容进行**分词**，并以分词的结果创建额外的倒排索引



### 1.4 lucene与es

- lucene是一个提供全文搜索功能类库的核心工具包，提供es分词与构建倒排索引的能力
- es是基于lucene开发搭建的服务框架，即lucene是es的内核



## 第二章 ES部署

### 2.1 安装ES

1. 上传并解压安装包到指定目录

   ```shell
   tar -zxvf elasticsearch-6.6.0.tar.gz -C /opt/module
   ```

2. 修改es配置文件

   ```shell
   cd /opt/module/elasticsearch-6.6.0/config
   vi elasticsearch.yml
   ```

   ```yaml
   #注意yml每行必须顶格编写,:后必须有空格
   
   #1.修改集群名称,同一集群节点集群名称必须相同
   cluster.name: my-es
   
   #2.修改节点名称
   node.name: node-1
   
   #3.修改网络地址,端口号默认9200
   network.host: hadoop201
   
   #4.关闭bootstrap自检程序(影响启动)
   bootstrap.memory_lock: false
   bootstrap.system_call_filter: false
   
   #5.修改自发现配置,用于节点找到集群并报道,保险起见配置2台节点
   discovery.zen.ping.unicast.hosts: ["hadoop201", "hadoop202"]
   ```

3. （生产环境无需修改）修改es的默认虚拟机内存占用

   ```shell
   vi jvm.options
   ```

   ```shell
   -Xms256m
   -Xmx256m
   ```

4. 分发es到所有节点

   ```shell
   cd /opt/module
   xsync 
   ```

5. 修改每个节点的配置文件（除hadoop201的其他节点）

   ```shell
   cd /opt/module/elasticsearch-6.6.0/config
   vi elasticsearch.yml
   ```

   ```shell
   #1.修改节点名称
   node.name: node-2
   
   #2.修改网络地址,端口号默认9200
   network.host: hadoop202
   ```

6. 修改linux配置，支持es高并发，修改单进程的虚拟内存占用（3台节点）

   ```shell
   sudo vim /etc/security/limits.conf
   ```

   ```shell
   #追加以下配置
   * soft nofile 65536
   * hard nofile 131072
   * soft nproc 2048
   * hard nproc 65536
   ```

   ```shell
   sudo vim /etc/sysctl.conf
   ```

   ```shell
   #添加如下配置
   vm.max_map_count=262144
   ```

7. （CenOS7以下需配置）修改最大进程数（3台节点）

   ```shell
   sudo vim /etc/security/limits.d/90-nproc.conf
   ```

   ```shell
   #修改如下配置
   * soft nproc 4096
   ```

8. 重启linux生效配置（3台节点）

   ```shell
   reboot
   ```




### 2.2 安装kibana

1. 上传并解压安装包到指定目录

   ```shell
   tar -zxvf kibana-6.6.0-linux-x86_64.tar.gz -C /opt/module/
   cd /opt/module/
   mv kibana-6.6.0-linux-x86_64/ kibana-6.6.0
   ```

2. 修改配置文件

   ```shell
   cd kibana-6.6.0/config/
   vi kibana.yml
   ```

   ```yaml
   #修改以下配置
   server.host: "0.0.0.0"
   
   elasticsearch.hosts: ["http://hadoop201:9200"]
   ```

3. 编写启动es集群和kibana的脚本

   ```shell
   cd ~/bin
   vi esctl
   ```

   ```shell
   #!/bin/bash 
   es_home=/opt/module/elasticsearch-6.6.0
   kibana_home=/opt/module/kibana-6.6.0
   
   case $1  in
    "start") {
     for i in hadoop201 hadoop202 hadoop203
     do
       ssh $i  "source /etc/profile;${es_home}/bin/elasticsearch >/dev/null 2>&1 &" 
     done
     nohup ${kibana_home}/bin/kibana >/opt/module/kibana-6.6.0/logs/kibana.log 2>&1 &
   };;
   
   "stop") {
     ps -ef|grep ${kibana_home} |grep -v grep|awk '{print $2}'|xargs kill
     for i in hadoop201 hadoop202 hadoop203
     do
         ssh $i "ps -ef|grep $es_home |grep -v grep|awk '{print \$2}'|xargs kill" >/dev/null 2>&1
     done
   };;
   esac
   ```

   ```shell
   chmod +x esctl
   ```

4. web端查看es节点状态

   ```http
   http://hadoop201:9200/_cat/nodes?v
   ```

   ```shell
   #可查看到3台节点状态,如下
   ip              heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
   192.168.145.201           55          43   3    1.00    0.40     0.26 mdi       *      node-1
   192.168.145.202           46          34   0    1.12    0.38     0.19 mdi       -      node-2
   192.168.145.203           44          34   0    0.84    0.35     0.18 mdi       -      node-3
   ```

5. web端访问kibana

   ```http
   http://hadoop201:5601/
   ```

   ![image-20200720211116476](Elasticsearch.assets/image-20200720211116476.png)



### 2.3 安装中文分词器

1. 下载分词器 https://github.com/medcl/elasticsearch-analysis-ik

2. 上传并解压安装包

   ```shell
   unzip elasticsearch-analysis-ik-6.6.0.zip -d ik/
   ```

3. 移动到es安装目录的插件目录下

   ```shell
   mv ik/ /opt/module/elasticsearch-6.6.0/plugins/
   ```

4. 分发到所有节点

   ```shell
   xsync ik
   ```

5. 重启es



## 第三章 ES基本概念

### 3.1 基本名词

| 名词     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| cluster  | 整个elasticsearch 默认就是集群状态，整个集群是一份完整、互备的数据。 |
| node     | 集群中的一个节点，一般只一个进程就是一个node                 |
| shard    | 分片，即使是一个节点中的数据也会通过hash算法，分成多个片存放，默认是5片。（7.0默认改为1片） |
| index    | index相当于table                                             |
| type     | 对表的数据再进行划分，比如“区”的概念。实际上对数据查询优化的作用有限，比较鸡肋（6.x只允许建一个，7.0后被废弃） |
| document | 行，对象                                                     |
| field    | 字段、属性                                                   |



### 3.2 服务状态查询

- 查询整体服务状态

  ```json
  GET /_cat/health?v
  // 说明
  // cluster 集群名称
  // status 集群状态 green代表健康,主分片都正常且各自至少有1个不同节点的副本(高可用);yellow代表分配了所有主分片正常，但至少缺少一个副本,此时集群数据仍旧完整;red代表部分主分片不可用,可能已经丢失数据。
  // node.total 代表在线的节点总数量
  // node.data 代表在线的数据节点的数量
  // shards active_shards 存活的分片数量
  // pri active_primary_shards 存活的主分片数量,正常情况下,shards的数量是pri的两倍。
  // relo relocating_shards 迁移中的分片数量,正常情况为0
  // init initializing_shards 初始化中的分片数量,正常情况为0
  // unassign unassigned_shards 未分配的分片,正常情况为0
  // pending_tasks 准备中的任务,任务指迁移分片等,正常情况为0
  // max_task_wait_time 任务最长等待时间
  // active_shards_percent 正常分片百分比,正常情况为,100%
  ```

- 查询每个节点状态

  ```json
  GET /_cat/nodes?v
  // 说明
  // heap.percent 堆内存占用百分比
  // ram.percent 内存占用百分比
  // cpu CPU占用百分比
  // master *表示节点是集群中的主节点
  // name 节点名
  ```

- 查询所有索引状态

  ```json
  GET /_cat/indices?v
  // 说明
  // health 索引的健康状态 
  // index 索引名 
  // pri 索引主分片数量 
  // rep 索引分片的副本数 
  // store.size 索引主分片和副本的总占用空间 
  // pri.store.size 索引总占用空间,不包括副本
  ```

- 查询指定索引的分片情况

  ```json
  GET /_cat/shards/xxxx
  // index 索引名称
  // shard 分片序号
  // prirep p表示该分片是主分片, r 表示该分片是复制分片
  // store 该分片占用存储空间
  // node 所属节点节点名
  // docs 分片存放的文档数
  ```



## 第四章 ES API

- es api是一种专用语言，仅限在es服务中使用。
- DSL全称 Domain Specific language，即特定领域专用语言。

### 4.1 操作数据

#### 4.1.1 索引和数据操作

```json
//1.创建索引
PUT /movie_index

//2.删除索引
DELETE /movie_index

//3.插入/整体更新数据(幂等操作)
//使用PUT做更新操作时,必须包含修改前数据的全部字段
//movie表示type,1表示主键的值,在es中主键字段不包含在json字符串中,此处共插入3条数据
PUT /movie_index/movie/1
{ 
    "id":1,
    "name":"operation red sea",
    "doubanScore":8.5,
    "actorList":[  
        {"id":1,"name":"zhang yi"},
        {"id":2,"name":"hai qing"},
        {"id":3,"name":"zhang han yu"}
    ]
}
PUT /movie_index/movie/2
{
    "id":2,
    "name":"operation meigong river",
    "doubanScore":8.0,
    "actorList":[  
        {"id":3,"name":"zhang han yu"}
    ]
}
PUT /movie_index/movie/3
{
    "id":3,
    "name":"incident red sea",
    "doubanScore":5.0,
    "actorList":[  
        {"id":4,"name":"zhang chen"}
    ]
}

//4.删除数据
//movie表示type,1表示主键的值
DELETE /movie_index/movie/1

//5.插入数据(非幂等操作)
//不指定主键,主键会随机生成
POST /movie_index/movie
{ 
    "id":1,
    "name":"operation red sea",
    "doubanScore":8.5,
    "actorList":[  
        {"id":1,"name":"zhang yi"},
        {"id":2,"name":"hai qing"},
        {"id":3,"name":"zhang han yu"}
    ]
}

//6.更新指定字段
POST movie_index/movie/3/_update
{ 
    "doc": {
        "doubanScore":"7.0"
    } 
}
```



#### 4.1.2 查询

```json
//1.使用主键查询
GET movie_index/movie/1

//2.使用type查询
GET movie_index/movie/_search

//3.查询整个索引
GET movie_index/_search

//4.分词匹配查询 match
//使用分词查询时,首先会对查询条件进行分词处理,然后将每个分词分别匹配表中的分词索引
//如搜索name为red sea时,会将所有name中包含red或包含sed的所有数据返回
GET movie_index/_search
{
    "query": {
        "match": {
            "actorList.name": "zhang"
        }
    }
}

//5.短语匹配查询 match_phrase
//不再对查询条件做分词处理,而是使用整体匹配表中相应字段的数据
GET movie_index/_search
{
    "query": {
        "match_phrase": {
            "name": "red sea"
        }
    }
}

//6.分词模糊查询 fuzzy
//在分词查询的基础上,与查询条件分词接近的结果也被返回(如rad→red),消耗性能且仅适用于英文
GET movie_index/_search
{
    "query": {
        "fuzzy": {
            "name": "rad"
        }
    }
}

//7.查询后过滤 post_filter
GET movie_index/movie/_search
{
    "query":{
        "match": {"name":"red"}
    },
    "post_filter":{
        "term": {
            "actorList.id": 3
        }
    }
}

//8.查询同时过滤(重要) filter
GET movie_index/movie/_search
{ 
    "query":{
        "bool":{
            "filter":[ 
                {"term": {  "actorList.id": "1"  }},
                {"term": {  "actorList.id": "3"  }}
            ], 
            "must":{"match":{"name":"red"}}
        }
    }
}
//逻辑条件说明
//must 语句内容必须全部满足
//must_not 语句内容必须全部不满足
//should 语句内容只要有一个满足即可

//9.查询同时按范围过滤
GET movie_index/movie/_search
{
    "query": {
        "bool": {
            "filter": {
                "range": {
                    "doubanScore": {"gte": 8}
                }
            }
        }
    }
}
//比较符运算说明
//gt 大于
//lt 小于
//gte 大于等于
//lte 小于等于

//10.排序
GET movie_index/movie/_search
{
    "query":{
        "match": {"name":"red sea"}
    }
    , "sort": [
        {
            "doubanScore": {
                "order": "desc"
            }
        }
    ]
}

//11.分页
//from 从第n行数据开始查询(行号从0开始)
//size 查询的数据量
GET movie_index/movie/_search
{
    "query": { "match_all": {} },
    "from": 10,
    "size": 10
}

//12.查询指定字段
GET movie_index/movie/_search
{
    "query": { "match_all": {} },
    "_source": ["name", "doubanScore"]
}

//13.高亮查询
GET movie_index/movie/_search
{
    "query":{
        "match": {"name":"red sea"}
    },
    "highlight": {
        "fields": {"name":{} }
    }
}
```

- match，match_phrase与term对比

  - match将语句中作为查询条件的数据进行分词处理，将**多个分词**分别与库中的索引表数据做**被包含**匹配
  - match_phrase将语句中作为查询条件的数据整体视为**1个分词**，将该分词与库中的索引表数据做**被包含**匹配
  - term将语句中作为查询条件的数据视为**1个分词**，将该分词与库中的索引表数据做**等值**匹配

- .keyword的使用与不使用对比

  - 若查询字段不使用.keyword，则会使用库中目标字段的**分词索引**查询
  - 若查询字段使用.keyword，则会使用库中目标字段的**原始数据索引**（未分词）查询

- 总结

  ```json
  //若库中包含以下数据
  {"name": "red"}                //编号1
  {"name": "red sea"}            //编号2
  {"name": "operation red sea"}  //编号3
  
  //6种查询组合与返回结果如下
  
  {"match": {"name": "red sea"}}	//返回123
  {"match_phrase": {"name": "red sea"}}	//返回23
  {"term": {"name": "red sea"}}	//返回2
  
  {"match": {"name.keyword": "red sea"}}	//返回2
  {"match_phrase": {"name.keyword": "red sea"}}	//返回2
  {"term": {"name.keyword": "red sea"}}	//返回2
  ```

  



#### 4.1.3 含查询条件的数据操作

```json
//1.删除数据
//使用.keyword查询时,会匹配表中字段的原数据而不是分词
POST movie_index/movie/_delete_by_query 
{
    "query":{
        "term": {
            "actorList.name.keyword": "zhang chen"
        }
    }
}

//2.修改数据
POST movie_index/movie/_update_by_query
{
    "script":"ctx._source['actorList'][0]['name']='zhang san feng'",
    "query":{
        "term": {
            "actorList.name.keyword": "zhang chen"
        }
    }
}
```



#### 4.1.5 分组聚合

```json
//1.count(自带)
//groupby_actor 为自定义的分组字段名(字段别名)
//size 为分组的数量,可大不可小,默认为10,估算分组结果数后可以尽量设置大些
GET movie_index/movie/_search
{ 
    "aggs": {
        "groupby_actor": {
            "terms": {
                "field": "actorList.name.keyword"  
                "size": 1000
            }
        }
    }
}

//2.avg
//groupby_actor和avg_score为自定义的字段别名
GET movie_index/movie/_search
{ 
    "aggs": {
        "groupby_actor": {
            "terms": {
                "field": "actorList.name.keyword" ,
                "order": {
                    "avg_score": "desc"
                }
            },
            "aggs": {
                "avg_score":{
                    "avg": {
                        "field": "doubanScore" 
                    }
                }
            }
        } 
    }
}
```



### 4.2 SQL使用

- ES在6.3版本后开始支持SQL查询，不支持增删改操作

- 参考文档 https://www.elastic.co/guide/en/elasticsearch/reference/6.6/sql-functions.html

- 不支持窗口函数、高亮、分页功能

- 示例

  ```json
  GET _xpack/sql?format=txt
  {
  "query": "select actorList.name.keyword, avg(doubanScore) from movie_index where match(name,'red') group by actorList.name.keyword limit 10" 
  }
  ```



### 4.3 中文分词器

- 两种分词器

  ```json
  //1.ik_smart 将短语直接拆分,不重复
  //"我是中国人" → "我" "是" "中国人"
  GET _analyze
  {
    "text": "我是中国人", 
    "analyzer": "ik_smart"
  }
  
  //2.ik_max_word 将短语的所有可能结果都拆分
  //"我是中国人" → "我" "是" "中国人" "中国" "国人"
  GET _analyze
  {
    "text": "我是中国人", 
    "analyzer": "ik_max_word"
  }
  ```



### 4.4 数据类型

- es中字段的数据类型由mapping定义，若在插入数据时未指定，则自动推断数据类型，在生产环境下需要根据需求尽量手动设定字段类型

- 数据类型推断

  ```json
  true/false → boolean
  1000 → long
  1.1 → float
  "2020-01-01" → date
  "hello world" → keyword + text
  //只有text类型的数据会触发分词
  ```

- API操作

  ```json
  //1.查看mapping
  GET movie_index/_mapping          //查看整个index
  GET movie_index/_mapping/movie    //查看type
  
  //2.使用指定数据类型创建索引type
  //字段数据类型一旦确定,不可更改,同一index下不同type的同名字段数据类型必须一致
  //中文字段需要配置中文分词器使用
  PUT movie_chn
  {
      "settings": {                            
          "number_of_shards": 1
      },
      "mappings": {
          "movie":{
              "properties": {
                  "id":{
                      "type": "long"
                  },
                  "name":{
                      "type": "text", 
                      "analyzer": "ik_smart"
                  },
                  "doubanScore":{
                      "type": "double"
                  },
                  "actorList":{
                      "properties": {
                          "id":{
                              "type":"long"
                          },
                          "name":{
                              "type":"keyword"
                          }
                      }
                  }
              }
          }
      }
  }
  ```



### 4.5 分隔索引

- 在生产环境中，若一个业务使用一个索引存储，则时间越长，索引数据量越大，性能越差。因此一般通过为**索引名+后缀**建立**多个索引**存储**同一业务**的数据，常见的后缀为时间，类似与MySQL以时间字段创建分区表。
- 分隔索引的优点
  1. 优化查询效率：查询指定时间周期的数据，避免全数据扫描
  2. 结构变化灵活：因为es中，同一个索引内字段的数据类型一旦确定不可更改，但使用分隔索引解除了这种限制，可以在新的索引中使用新的字段数据类型



### 4.6 索引别名

-  索引别名类似于一个快捷方式，一个别名可以指向一或多个索引或索引的一部分。通过使用别名，可以提高程序的复用性，在需要改变需求的索引时，仅需要改变别名和索引的映射关系，程序只与别名建立联系 ，无需修改。

- 索引与别名是n对n的关系，即一个索引可以关联多个别名，一个别名也可以被多个索引关联

  

#### 4.6.1 索引与别名

```json
//1.为索引关联别名 
POST _aliases
{
    "actions": [
        { 
            "add":    { 
                "index": "movie_chn_20200101", 
                "alias": "movie_chn_2020-query" 
            }
        }
    ]
}

//2.创建索引时直接关联别名
PUT movie_chn_20200101
{  
    "aliases": {
        "movie_chn_2020-query": {}
    }, 
    "mappings": {
        "movie":{
            "properties": {
                "id":{
                    "type": "long"
                },
                "name":{
                    "type": "text"
                    , "analyzer": "ik_smart"
                },
                "doubanScore":{
                    "type": "double"
                },
                "actorList":{
                    "properties": {
                        "id":{
                            "type":"long"
                        },
                        "name":{
                            "type":"keyword"
                        }
                    }
                }
            }
        }
    }
}

//3.使用索引的一部分关联别名
POST  _aliases
{
    "actions": [
        { 
            "add":    
            { 
                "index": "movie_chn_xxxx", 
                "alias": "movie_chn0919-query-zhhy",
                "filter": {
                    "term": {  
                        "actorList.id": "3"
                    }
                }
            }
        }
    ]
}


//4.查询别名数据(与查询索引完全相同)
GET /movie_2020_query/_search

//5.删除索引与别名的关联
POST _aliases
{
    "actions": [
        { 
            "remove": { 
                "index": "movie_chn_xxxx", 
                "alias": "movie_chn_2020-query" 
            }
        }
    ]
}

//6.无缝切换索引与别名的关联
POST /_aliases
{
    "actions": [
        { "remove": { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }},
        { "add":    { "index": "movie_chn_yyyy", "alias": "movie_chn_2020-query" }}
    ]
}

//7.查询所有索引与别名的映射关系
GET _cat/aliases?v
```



#### 4.6.2 索引模版

- 当新建一个索引时，若索引名匹配到了已创建的索引模版，则会采用模版中配置的所有设置（包括shard数量，字段数据类型，关联的索引别名等等）新建该索引

- 索引模版的匹配方式为字符串的正则匹配

- API操作

  ```json
  //1.创建模版
  //匹配所有名称以movie_test开头的新建index
  PUT _template/template_movie2020
  {
      "index_patterns": ["movie_test*"],                  
      "settings": {                                               
          "number_of_shards": 1
      },
      "aliases" : { 
          "{index}-query": {},
          "movie_test-query":{}
      },
      "mappings": {                                          
          "_doc": {
              "properties": {
                  "id": {
                      "type": "keyword"
                  },
                  "movie_name": {
                      "type": "text",
                      "analyzer": "ik_smart"
                  }
              }
          }
      }
  }
  
  //2.查看所有模版
  GET _cat/templates
  
  //3.查看指定模版详情
  GET _template/template_movie2020
  ```




### 4.6 shard优化

- 由于在生产环境中，索引一般以天为时间建立，而一个索引又划分为多个shard存储，日积月累会产生大量shard文件，导致查询效率低下。

- shard数量优化

  1. 每日单个索引数据量低于10G时可仅使用1个shard存储，单个shard最大不能超过30G

  2. 单个节点中存储尽量不超过1000个shard，如不能满足需求则需要增加集群节点

  3. shard文件由segment组成，内存中的数据在触发刷写条件时将内存中的数据刷写到磁盘形成1个segment，因此segment数量过多也会影响shard的性能，因此需要合并。可以采用隔天对前一天的索引的所有shard手动进行segment合并的工作，操作如下

     ```json
     //1.合并指定索引的所有shard的segment数(1个shard仅有1个segment)
     POST  movie_index/_forcemerge?max_num_segments=1
     
     //2.查看所有索引的segment数量
      GET _cat/indices/?s=segmentsCount:desc&v&h=index,segmentsCount,segmentsMemory,memoryTotal,storeSize,p,r
     //优化后 segmentsCount = p + r
     ```




## 第五章 Java API

1. 创建Maven工程，导入依赖

   ```xml
   <dependency>
       <groupId>io.searchbox</groupId>
       <artifactId>jest</artifactId>
       <version>5.3.3</version>
   
   </dependency>
   
   <dependency>
       <groupId>net.java.dev.jna</groupId>
       <artifactId>jna</artifactId>
       <version>4.5.2</version>
   </dependency>
   
   <dependency>
       <groupId>org.codehaus.janino</groupId>
       <artifactId>commons-compiler</artifactId>
       <version>2.7.8</version>
   </dependency>
   
   <!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
   <dependency>
       <groupId>org.elasticsearch</groupId>
       <artifactId>elasticsearch</artifactId>
       <version>2.4.6</version>
   </dependency>
   ```

2. 编写代码

   ```scala
   object MyEsUtil {
   
       private var factory: JestClientFactory = null
   
       //返回es连接池对象的工具类方法
       def getJestClient(): JestClient ={
           if (factory == null){
               factory = new JestClientFactory
               factory.setHttpClientConfig(
                   new HttpClientConfig.Builder("http://hadoop201:9200")
                   .multiThreaded(true)
                   .maxTotalConnection(20)
                   .connTimeout(2000)
                   .readTimeout(2000)
                   .build())
           }
           factory.getObject
       }
   
       def main(args: Array[String]): Unit = {
   
           //获取es客户端对象
           val jestClient: JestClient = getJestClient()
   
           //1.写
           val index: Index = new Index.Builder(Movie("0001","建国大业"))
           .index("movie_test")
           .`type`("_doc")
           .id("1").build()
           jestClient.execute(index)
   
           //2.查询
           //方式一:将query的json字符串直接导入
           val query =
           """
                   |{"query": {"term": {"movie_name.keyword": "建国大业"}}}
           |""".stripMargin
           val search: Search = new Search.Builder(query).addIndex("movie_test").addType("_doc").build()
           val result: SearchResult = jestClient.execute(search)
           import scala.collection.JavaConversions._
           val list: util.List[SearchResult#Hit[util.Map[String,Object], Void]] = result.getHits(classOf[util.Map[String,Object]])
           for ( hit <- list ) {
               val source: util.Map[String, Object] = hit.source
               println(source)
           }
           //方式二:使用SearchSourceBuilder构建查询语句
           val builder = new SearchSourceBuilder
           builder.query(new MatchQueryBuilder("name","red"))
           builder.sort("doubanScore",SortOrder.DESC)
           builder.from(0)
           builder.size(10)
           val query2: String = builder.toString
   
           //关闭线程池的客户端对象
           jestClient.close()
   
       }
   
       //自定义类用于封装index中的每行数据
       case class Movie(id: String, movie_name: String)
   
   }
   ```

   