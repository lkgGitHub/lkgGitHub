---
title: elasticSearch
date: 2019-05-29 14:19:18
tags: elasticSearch
---
# DSL基本属性说明
## 1.query: 用于包含查询使用到的语法 

- 1.1. **match_all:** 最简单的查询，获取索引所有数据，类似搜索 *。如：”query”:{“match_all”:{}} 
- 1.2 bool 复合查询，可以包含多个查询条件，主要有(must,must_not,should)
  - 1.2.1. must: 用于包含**逻辑与**查询条件，即所有查询条件都满足才行 
  - 1.2.2. must_not: 用于包含**逻辑非**查询条件，即不包含所有查询的条件数据 
  - 1.2.3. should: 用于包含**逻辑或**查询条件，即其中有一个条件满足即可 
  - 1.2.4. filter: 与must一样，包含的所有条件都要满足，但**不会参与计算分值**，查询速度上会提升不少 
- 1.3. match: **匹配查询**，用于匹配指定属性数据，也可以匹配时间，IP等特殊数据 
    - 1.3.1. 注意： match匹配不会解析通配符，匹配的效果受到索引属性类型影响，如果索引属性设置了分词，那么match匹配也会分词匹配，他也不解析”“，但可以设置逻辑关系来处理 
    - 1.3.2. operator: 匹配逻辑关系，默认是or，可设置为and，这样可达到精确匹配的效果，具体见：https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-match-query.html#query-dsl-match-query-boolean 
- 1.4. query_string: 使用**查询解析器**来解析查询内容，如port:80 AND server:http。注意：此类型请求有很多选项属性，可以设置一些特殊的行为，具体见：https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-query-string-query.html 
- 1.5. term: 术语查询，查找包含在反向索引中指定的**确切术语的文档** 
- 1.6. terms: 术语查询，筛选**具有匹配任何条件的字段**，如”terms” : { “user” : [“kimchy”, “elasticsearch”]} 
- 1.7. range: 术语查询，将文档与具有**特定范围内的字段相匹配**。Lucene查询的类型依赖于字段类型，对于字符串字段，即TermRangeQuery，而对于number/date字段，查询是一个数字的范围。如：”range”:{“port”:{“gte”:10,”lte”:20,”boost”:2.0}} 
    - 1.7.1. **gte**: 大于或等于 
    - 1.7.2. gt: 大于 
    - 1.7.3. **lte**: 小于或等于 
    - 1.7.4. lt: 小于 
    - 1.7.5. boost: 设置查询的boost值，默认值为1.0 
- 1.8. exists: 术语查询，返回在原始字段中至少有一个**非空值的文档**，注意：”“,”-“这些都不算是空值 
- 1.9. prefix: 术语查询，匹配包含带有指定**前缀字段的字段**(没有分析)，前缀查询映射到Lucene前缀查询，如：”prefix” : { “user” : “ki” }，查询user数据前缀为ki的doc 
- 1.10. wildcard: 术语查询，匹配具有**匹配通配符表达式的字段**(未分析)的文档。支持(*通配符)是匹配任何字符序列(包括空的序列)和(?通配符)它匹配任何一个字符。注意，这个查询可能比较慢，因为它需要迭代多个术语。为了防止非常慢的通配符查询，一个通配符项不应该从通配符开始，或者?通配符查询映射到Lucene通配符查询。如：”wildcard” : { “user” : “ki*y” } 
- 1.11. regexp: 术语查询，regexp查询允许您使用**正则表达式术语查询**，意味着Elasticsearch将把regexp应用到该字段的标记器所产生的词汇，而不是该字段的原始文本。regexp查询的性能严重依赖于所选择的正则表达式，通配符往往会降低查询性能。如：”regexp”:{ “name.first”:”s.*y” } 
- 1.12. fuzzy: 术语查询，模糊查询使用基于Levenshtein编辑距离的相似性。如：”fuzzy” : { “user” : “ki” } 
- 1.13. type: 术语查询，过滤文档匹配所提供的文档/映射类型。如：”type”:{ “value” : “my_type” } 
- 1.14. ids: 术语查询，**过滤只具有提供的id的文档**。注意：这个查询使用了_uid字段，类型是可选的，可以省略，也可以接受一组值。如果没有指定类型，那么将尝试在索引映射中定义的所有类型。如：”ids”:{ “type” : “my_type”,”values” : [“1”,”4”,”100”] }。
    highlight: 允许在一个或多个字段中突出显示搜索结果，基于lucene plain highlighter。在默认情况下，高亮显示会将高亮显示的文本包装在 and ，可以通过设置pre_tags 与 post_tags来自定义，如：”highlight”:{ “pre_tags” : [““], “post_tags” : [““], “fields” : {“_all”:{}} } 

## 2.highlight

: 允许在一个或多个字段中突出显示搜索结果，基于lucene plain highlighter。在默认情况下，高亮显示会将高亮显示的文本包装在 and ，可以通过设置pre_tags 与 post_tags来自定义，如：”highlight”:{ “pre_tags” : [““], “post_tags” : [““], “fields” : {“_all”:{}} } 

- 2.1. pre_tags: 自定义包含搜索关键字的前缀 

- 2.2. post_tags: 自定义包含搜索关键字的后缀 

- 2.3. fields: 用于指定要高亮的属性，_all表示所以属性都需要高亮，如：”fields”:{ “_all” : {} }，也可以指定具体属性 “fields”:{ “app” : {} }，也可以给每个属性单独指定设置 “fields”:{ “app” : {“fragment_size” : 150, “number_of_fragments” : 3} } 

- 2.4. highlight_query: 可以通过设置highlight_query来突出显示搜索查询之外的查询，通常，最好将搜索查询包含在highlight_query中。如：”highlight_query”:{ “bool”:{“must”:[{“query_string”:{“query”:app:apache,”analyze_wildcard”:True,”all_fields”:True}}]} } 

- 2.5. fragment_size: 用于指定高亮显示时，碎片的长度，如果过短，高亮内容会被切分为多个碎片。默认情况下，当使用高亮显示的内容时，碎片大小会被忽略，因为它会输出句子，而不管它们的长度 

- 2.6. number_of_fragments 用于指定高亮显示时，碎片的数量，如果指定为0，那么就不会产生任何片段

## 3.from

  可以通过使用from和size参数来对结果进行分页，from参数指定您想要获取的第一个结果的偏移量

##   4.size: 

可以通过使用from和size参数来对结果进行分页，size参数指定要返回结果的最大数量

##   5.sort: 

**desc降序，asc升序**

允许在特定的字段上添加一个或多个排序，排序是在每个字段级别上定义的，用特殊的字段名来排序，然后根据索引排序进行排序，如”sort”: [ { “date”: { “order”: “desc” } } ]，

##   6.aggs:

 简单来说，aggs主要用于分类集合，可以将查询的数据按指定属性进行分类集合统计等，如：”aggs”:{ “deviceType”:{ “terms”:{ “field”:”deviceType”, “size”:6 } } } 

- 6.1. field: 简单来说，用于指定要分类的属性名称 

- 6.2. size: 用于指定分类集合的数量，即只集合前N名

# DSL查询结果格式

- **ELK的查询结果格式**都是一致的，所以在这里做统一说明，格式如下

- **注意:** 下面的结果格式包含了查询，高亮，排序，分类聚合等数据格式示例，如果只是进行查询，就只会有结果格式，只有进行了相应的操作，才会有相应的结果

  ```shell
  {
      # hits包含了所有结果信息，如条目总数等
      "hits": {
          # hits包含了具体的结果
          "hits": [
              {
                  # sort包含时间排序值，结果已经是按时间排序了的
                  "sort": [
                      1508408100000
                  ],
                  # 索引类型
                  "_type": "test",
                  # 结果原数据
                  "_source": {
                      "redunip": "122.115.79.2",
                      "date": "2017-10-19T18:15:00+08:00",
                      "country": "中国"
                  },
                  "_score": null,
                  # 索引名称
                  "_index": "test_test_test",
                  # 包含对原数据高亮的部分，如果要显示高亮，需要将此数据覆盖原数据
                  "highlight": {
                      "country": [
                          "<em>中国</em>"
                      ]
                  },
                  # 索引数据ID
                  "_id": "122.115.79.2"
              },
              {
                  "sort": [
                      1508408100000
                  ],
                  "_type": "test",
                  "_source": {
                      "redunip": "182.254.216.71",
                      "date": "2017-10-19T18:15:00+08:00",
                      "country": "中国"
                  },
                  "_score": null,
                  "_index": "test_test_test",
                  "highlight": {
                      "country": [
                          "<em>中国</em>"
                      ]
                  },
                  "_id": "182.254.216.71"
              }
          ],
          # 查询数据总条目数量
          "total": 18934351,
          "max_score": null
      },
      # 查询涉及到的数据分块
      "_shards": {
          "successful": 30,
          "failed": 0,
          "skipped": 0,
          "total": 30
      },
      "took": 228,
      # 查询分类聚合数据
      "aggregations": {
          # 分类的名称
          "country": {
              # 聚合的结果
              "buckets": [
                  {
                      # 聚合的数据
                      "key": "中国",
                      # 聚合统计的总数量
                      "doc_count": 18934351
                  }
              ],
              "sum_other_doc_count": 0,
              "doc_count_error_upper_bound": 0
          }
      },
      # 表示查询是否超时
      "timed_out": false
  }
  ```

# [elasticsearch 查询（match和term）](https://www.cnblogs.com/yjf512/p/4897294.html)



## match 

分词后匹配，并且根据lucene的评分机制(TF/IDF)来进行评分。

### match_phrase

精确匹配，分词后的词都必须匹配上。eg："宝马多少马力"会被分词为"宝马 多少 马力"，match_phrase结果必须同时包含"宝马 多少 马力"

### multi_match

两个字段进行匹配，其中一个字段有这个文档就满足就能匹配上。

## term 

term是代表完全匹配，即不进行分词器分析，文档中必须包含整个搜索的词汇。

使用term要确定的是这个字段是否“被分析”(analyzed)，默认的字符串是被分析的。

# Mapping(映射)



**注意**：mapping的注意事项

mapping一旦创建之后，就无法修改，只能追加，如果要修改，就需要删除掉整个文档进行重建。



## 2. mapping中的可设置的属性

mappings ： 在index（库）下创建时使用，下面可以有多个mapping 以下数据结构主要针对每个mapping进行说明：

| 一级属性   | 二级属性 | 三级属性        | 说明                                                         |
| ---------- | -------- | --------------- | ------------------------------------------------------------ |
| dynamic    |          |                 | 新增字段自动模式；true:表示自动识别新字段并创建索引，false:不自动索引新字段，strict:遇到未知字段，抛异常，不能存入 |
| _timestamp |          |                 | 是否使用时间戳，ES会自动加时间戳，使用的话请百度             |
| properties |          |                 | 属性列表（类似数据库多个字段定义）                           |
|            | {字段名} |                 | 某个字段的定义                                               |
|            |          | type            | 数据类型，参见数据类型说明                                   |
|            |          | index           | 映射选型，参见映射选型说明                                   |
|            |          | doc_values      | 布尔值， 对not_analyzed字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存 |
|            |          | format          | 如果数据类型为日期格式，传入值得时候是字符串，ES需要一个格式进行识别，如：yyyy-MM-dd HH:mm: ss |
|            |          | analyzer        | 分词器，如ik,ansj(中文分词)                                  |
|            |          | boost           | 浮点型，字段级别的分数加权（权重）                           |
|            |          | ignore_above    | 超过多少字符，就不处理，分词性能损耗较大，对字符串较长的可不分词 |
|            |          | null_value      | 设置一些缺失字段的初始化值，只有string可以使用，分词字段的null值也会被分词 |
|            |          | store           | 是否单独设置此字段的是否存储而从_source字段中分离，默认是false，只能搜索，不能获取值 |
|            |          | search_analyzer | 设置搜索时的分词器，默认跟ananlyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能 |
|            |          | 其它            | similarity，term_vector，norms，include_in_all，index_options，fielddata，ignore_malformed，precision_step |



一个典型的mapping对象的属性有：

```json
{
    "mappings": {
        "my_type": {
        //true:表示自动识别新字段并创建索引，false:不自动索引新字段，strict:遇到未知字段，抛异常，不能存入
            "dynamic":      "strict", 
            
              //动态模板
             "dynamic_templates": [
                    { "stash_template": {
                      "path_match":  "stash.*",
                      "mapping": {
                        "type":           "string",
                        "index":       "not_analyzed"
                      }
                    }}
                  ],
            //属性列表
            "properties": {
                //一个strign类型的字段
                "title":  { "type": "string"},
                
                "stash":  {
                    "type":     "object",
                    "dynamic":  true 
                }
            }
        }
    }
}
```



## Elasticsearch数据类型

**JSON基础类型如下：**

字符串：text、string
数字：byte、short、integer、long、float、double、
时间：date
布尔值:boolean（true、false） 
数组: array
对象: object

**Elasticsearch独有的类型：**

多重: multi
经纬度: geo_point
网络地址: ip
堆叠对象: nested object
二进制: binary
附件: attachment



### 映射选型说明

某个属性（字段）的index可选值有：

analyzed:默认选项，以标准的全文索引方式，分析字符串，完成索引。
not_analyzed:精确索引，不对字符串做分析，直接索引字段数据的精确内容。
no：不索引该字段。