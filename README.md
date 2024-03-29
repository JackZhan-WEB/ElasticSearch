# ElasticSearch

## 基本的CRUD

### 增加

```
指定ID创建新文档，如果已经存在，会报错：
POST/PUT users/_create/3
{
  "context":"小蕾蕾获取了今年的奥数冠军",
  "age":19
}

指定ID创建新文档，如果已经存在，会报错：
POST/PUT users/_doc/1?op_type=create
{
  "age":11,
  "user":"chencheng"
}


不指定ID创建(只能使用POST)
POST users/_doc
{
  "context":"张三捡到了5元",
  "age":19
}

```

### 删除

```
删除文档：DELETE index/_doc/1
删除索引：DELETE index
```

### 修改

```
Index = 删除再创建 版本号+1
POST/PUT users/_doc/1?op_type=index  (默认就是index，可以不用显示的写出来)
{
  "age":11,
  "user":"chencheng"
}

Update = 会对文档做增量的更新  更新成功 版本号+1 更新不成功(没有更新的内容) 版本号不+1
POST users/_update/1
{
  "doc":{
    "age":22 
  }
}
```

### 查询

```
查询文档：GET index/_doc/1
查询索引配置：GET index
```

### 注意：

**Index 和 Create 不一样的地方：如果文档不存在，就索引新的文档。否则现有文档会被删除，新的文档被索引。版本信息+1**

## 批量操作

### bulk 

批量操作，在一次请求中多次操作索引

```
POST _bulk
{"create":{"_index":"users","_id":18}}
{"doc":{"name":"王武","age":"36"}}
{"delete":{"_index":"users","_id":3}}
{"update":{"_index":"users","_id":4}}
{"doc":{"name":"小蕾"}}
{"index":{"_index":"users","_id":1}}
{"doc":{"context":"李思想去考研!!!"}}
```

### mget

只能根据ID批量获取


```
GET _mget
{
  "docs":
  [
    {
      "_index":"users",
      "_id":1
    },
     {
      "_index":"users",
      "_id":2
    },
     {
      "_index":"users",
      "_id":3
    }
  ]
}

如果index都一样可以简写成下面：

GET users/_mget
{
  "docs":
  [
    {
      "_id":1
    },
     {
      "_id":2
    },
     {
      "_id":3
    }
  ]
}
```

### msearch

根据条件批量获取

```
GET users/_msearch
{}
{"query":{"match_all":{}},"from":4,"size":1}
{}
{"query":{"match":{"_id":1}}}
```

### 注意

**mget 是通过文档ID列表得到文档信息。msearch 是根据查询条件，搜索到相应文档。**

## 倒排索引

**[倒排索引](https://blog.csdn.net/andy_wcl/article/details/81631609)(Inverted Index)**：倒排索引是实现“单词-文档矩阵”的一种具体存储形式，通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：“单词词典”和“倒排文件”。

Elasticsearch的JSON文档中的每个字段，都有自己的倒排索引

可以指定对某些字段不做索引

  -  优点：节省存储空间
  -  缺点：字段无法被搜索

## 分词（Analyzer）

### Analysis 和 Analysis 

- Analysis 文本分析是把全文本转换一系列单词（term/token）的过程，也叫分词

- Analysis 是通过Analyzer来实现的
  - 可使用Elasticsearch内置的分析器或者按需定制化分析器

- 除了在数据写入时转换词条，匹配Query 语句时候也需要用相同的分析器对查询语句进行分析

### Analyzer 的组成

- 分词器是专门处理分词的组件，Analyzer由三部分组成

  - Character Filters（针对原始文本处理，例如去除html）

  - Tokenizer（按照规则切分为单词）
  - Token Filter（将切分的的单词进行加工，小写，删除stopwords，增加同义词）

### ElasticSearch 内置的分词器

- Standard Analyzer-默认分词器，按词切分，小写处理
- Simple Analyzer-按照非字母切分（符号被过滤），小写处理
- Stop Analyzer-小写处理，停用词过滤（the，a，is）
- Whitespace Analyzer-按照空格切分，不转小写
- Keyword Analyzer-不分词，直接将输入当作输出
- Patter Analyzer-正则表达式，默认\W+（非字符分隔）
- Language-提供了30多种常见语言的分词器
- Customer Analyzer 自定义分词器

### 使用分词API

```
指定Analyzer：
GET /_analyze
{
  "analyzer": "standard",
  "text":"I like eat apple in morning"
}
指定索引的字段：
GET users/_analyze
{
  "field": "context",
  "text":"I like eat apple in morning"  
}

自定义分词：例如指定TokenFiter 将切分的的单词进行小写
GET /_analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase"],
  "text":"I like eat apple in morning"  
}
```

## Search API

### 种类

- URI Search 
  - 在URL中使用查询参数

- Request Body Search 
  - 使用Elasticsearch提供的，基于JSON格式的更加完备的Query Domain Specific Language（DSL）

<img src="./SearchAPI/范围.png" alt="image-20191214164447487" style="zoom: 50%;" />

### URI Search

- 使用 "q" ，指定查询字符串
- df 默认字段，不指定时，会对所有字段进行查询
- sort 排序 / from 和 size 用于分页
- profile 可以查看查询是如何执行的



```
如果需要搜索中文需要转成 UrlEncode：查询 红色 
GET users/_search?q=%E7%BA%A2%E8%89%B2
```



![image-20191214164840831](./SearchAPI/KV键值对.png)

### Request Body Search

![image-20191214165037720](./SearchAPI/image-20191214165037720.png)

### Response

返回的信息

![image-20191214165149363](./SearchAPI/image-20191214165149363.png)