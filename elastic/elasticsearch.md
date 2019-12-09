# elasticsearch
- 倒排索引
  - 单词词典 Term Dictionary
    - B+树或哈希拉链法实现，以满足查询需要
  - 倒排列表 Posting List
    - 文档ID
    - 词频TF(Term frequencies)：用于相关性算分
    - 位置Position：用于语句搜索
    - 偏移量Offset：用于高亮显示
- 集群
  - 节点node
    - Master-eligible Nodes
      - 预备选举节点
      - 默认启动后都是预备节点，第一个启动起来的就是主节点
    - Master Node
      - 只有Master节点才能修改集群的状态信息
    - Data Node
      - 保存分片数据
    - Coordinating Node
      - 接收RESTful用户请求，并将请求转发到合适的节点，最终把结果汇总
      - 默认每个节点默认都起到Coordinating Node的职责
    - Hot & Warm Node
      - 冷热节点，降低集群部署的成本
    - Machine Learning Node
      - 负责跑机器学习Job
  - 分片shard
    - primary shard
      - 数据水平扩展
      - 一个分片是一个运行Lucene的实例
      - 主分片数在索引创建时指定，后续不允许修改，除非Reindex
    - replica shard
      - 数据高可用
      - 可以动态调整，提升服务的可用性（读取的吞吐量）
    - 分片数的设定
      - 分片数过小
        - 水平扩展能力减少
        - 单分片数据量过大
      - 分片数过大
        - over-sharding问题
        - 影响搜索结果的相关性打分，影响统计结果的准确性
        - 单个节点上过多分片，会导致资源浪费，影响性能
  - 集群健康
    - ```
      GET _cluster/health
      ```
    - Green
      - 主分片和副本都正常分配
    - Yellow
      - 主分片正常分片，副本分片未能正常分配
    - Red
      - 主分片未能正常分配
- 元素
  - Document
    - _index:文档所在的索引名
    - _type:文档所在的类型名（7.0以后一个索引对应一个type）
    - _id:文档唯一id
    - _uid:组合id，6.x以后废弃
    - _source:文档原始Json数据
    - _all:整合所有字段内容，默认禁用
  - Index
    - mapping
      - type 7.0后一一对应
      - 定义索引中的字段名称和数据类型
      - 定义倒排索引配置
      - dynamic mapping
        - 如果未定义mapping直接写入文档时，es自动创建mapping的机制
        - 字段类型推断
          - reindex 修改字段类型
        - dynamic 参数可限制mapping范围，在修改文档字段时，可控制mapping是否动态调整
          - |dynamic参数|true|false|strict|
            |---|---|---|---|
            |文档可索引|YES|YES|NO|
            |字段可索引|YES|NO|NO|
            |Mapping可更新|YES|NO|NO|
          ```
          PUT {index}/_mapping
          {
              "dynamic":false
          }
          ```
        - Exact Values vs. Full Text （精确值和全文本）
          - 精确值类型使用 keyword，不进行分词
          - 全文本类型使用 text，进行分词
    - analysis
    ```
    PUT {index}
    {
        "settings":{
            "analysis":{
                "analyzer":{
                    "{analyzer_name}":{
                        "type":"english",
                        "stem_exclusion":["organization"],
                        "stopwords":["a","an","and"]    
                    }
                }
            }
        }
    }
    PUT {index}
    {
        "settings":{
            "analysis":{
                "analyzer":{
                    "{analyzer_name}":{
                        "type":"custom",
                        "char_filter":["{char_filter_name}"],
                        "tokenizer":"{tokenizer_name}",
                        "filter":["lowercase","{filter_name}"]
                    }
                },
                "tokenizer":{
                    "{tokenizer_name}":{
                        "type":"pattern",
                        "pattern":"[.,!?]",
                    }
                },
                "char_filter":{
                    "{char_filter_name}":{
                        "type":"mapping",
                        "mappings":["_ => -"]
                    }
                },
                "filter":{
                    "{filter_name}":{
                        "type":"stop",
                        "stopwords":"_a"
                    }
                }
            }
        }
    }
    ```
    - setting
      - shard
- 基本操作
  - CURD
    - | API |Example|Explain|
      | --- |  ---  |  ---  |
      |Index| PUT {index}/_doc/{id}<br>{"test":"abc"}|如果ID不存在，创建新的文档；<br>如果存在，则删除后新建；<br>版本增加|
      |Create| PUT {index}/_create/{id}<br>{"test":"abc"}|如果使用PUT不指定id，则自动生成；<br>如果使用POST指定id已存在，则新增失败|
      |Delete| DELETE {index}/_doc/{id}||
      |Update| POST {index}/_update/{id}<br>{ "doc":{"test":"abc"}}|指定的文档id必须存在；<br>只对相应字段做增量修改，不能删除已存在字段；<br>版本增加|
      |Read| GET {index}/_doc/{id}||
  - 批量
    - Bulk API一次提交执行Index/Create/Update/Delete，其中执行失败的操作不影响其他操作，返回结果包括每一条操作执行的结果
    - _mget
    - _msearch
- 数据类型
  - 字符串：text，keyword
  - 数值型：long，integer，short，byte，double，float，half_float，scaled_float
  - 布尔：boolean
  - 日期：date
  - 二进制：binary
  - 范围累心：integer_range，float_range，long_range，double_range，date_range
- 索引
  - 正排索引
    - 文档id到文档内容单词的关联关系
  - 倒排索引
    - 单词到文档id的关联关系
    - 组成
      - Term Dictionary：倒排索引的分词词典
      - Posting List：记录分词所在的偏移量
        - index_options 参数可规定倒排列表记录的内容
          - docs:doc id
          - freqs:doc id,term frequencies
          - positions:doc id,,term frequencies,term position
          - offsets:doc id,,term frequencies,term position,character offset
  - 搜索引擎查询流程
    - 查询倒排索引单词所在的具体文档id
    - 拿到文档id查询正排索引文档的具体内容
- 分词
  - 分词器 analysis
    - Character Filters：对原始文本进行处理，如去除特殊标记符
      - HTML Strip
      - Mapping 字符替换
      - Pattern Replace 正则替换
    - Tokenizer：按一定规则分词
      - standard
      - letter
      - whitespace
      - UAX URL Email
      - NGram 和 Edge NGram连词分割
      - Path Hierarchy 文件路径分割
    - Token Filters：对分词结果进行再加工，如转小写，stop words
      - lowercase
      - stop
      - NGram和Edge Ngram连词分割
      - Synonym添加近义词的term
  - 自带分词器
    - Standard
    - Simple
      - 非字母切分
    - Whitespace
      - 空格切分
    - Stop
      - stop word切分
    - Keyword
      - 不进行分词，保留原文作为关键字
    - Pattern
      - 正则表达式切分
    - Language
      - 常见语言分词器
  - 中文分词器
    - ICU
    - IK
    - jieba
    - Hanlp
    - THULAC
  - Analyze API
    - 指定analyzer进行分析
    ```
    POST _analyze
    {
        "analyzer":"standard",
        "text":"hello world"
    }
    ```
    - 指定索引中的字段进行分析
    ```
    POST test_index/_analyze
    {
        "field":"username",
        "text":"hello world"
    }
    ```
    - 自定义分词器进行测试
    ```
    POST _analyze
    {
        "tokenizer":"standard",
        "filter":["lowercase"],
        "text":"Hello World"
    }
    ```
- 自定义分词
  - 索引时分词
  ```
  PUT test_index
  {
      "setting":{
          "analysis":{
              "analyzer":{},
              "tokenizer":{},
              "char_filter":{},
              "filter":{}
          }
      }
  }
  ```
  - 查询时分词
  ```
  
  ```
- Index Templates
  - 查看模板
  ```
  GET /_template/{template_name}
  ```
  - 创建索引模板
    - 模板优先级从低到高
      - 默认settings和mappings
      - order参数低
      - order参数高
      - 创建索引时指定
  ```
  PUT _template/{template_name}
  {
      "index_patterns":["*"],
      "order":0,
      "version":1,
      "settings":{
          "number_of_shards":1,
          "number_of_replicas":1
      }
  }
  PUT _template/{template_name}
  {
      "index_patterns":["test*"],
      "order":1,
      "version":1,
      "settings":{
          "number_of_shards":1,
          "number_of_replicas":2
      },
      "mappings":{
          "date_detection":false,
          "numeric_detection":true
      }
  }
  ```
- Dynamic Template
  - 自定义动态类型推断
  ```
  PUT {index_name}
  {
      "mappings":{
          "dynamic_templates":[
              "{dynamic_tmpl_name}":{
                  "path_match":"name.*",
                  "path_unmatch":"*.middle",
                  "mapping":{
                      "type":"text",
                      "copy_to":"full_name"
                  }
              }
          ]
      }
  }
  ```
- 查询
  - URI Search
    ```
    GET {index}/_search?q={keyword}&df={field}&sort=year:desc&from=0&size=10&timeout=1s
    {
        "profile":true
    }
    ```
    |参数|说明|实例|
    |---|---|---|
    |q|指定查询语句|?q={keyword}|
    |df|默认字段，不指定会对所有字段进行查询|&df={field}|
    |sort|排序|&sort=year:desc|
    |from/size|分页|&from=0&size=10|
    |profile|查询过程分析|"profile":true|
  - term
    - 代表完全匹配，不进行分词器分析
  - match
    - 查询词会被分词
    ```
    POST {index}/_search
    {
        "query":{
            "match":{
                "{field}":{
                    "query":"{keyword}",
                    "operator":"and"
                }
            }
        }
    }
    ```
  - match_phrase
    - 查询词顺序固定
    - slop 单词间最多分隔词数
    ```
    POST {index}/_search
    {
        "query":{
            "match_phrase":{
                "{field}":{
                    "query":"{keyword}"
                    "slop":1
                }
            }
        }
    }
    ```
    - query_string
      - query参数支持运算符 AND OR NOT 括号()
    ```
    POST {index}/_search
    {
        "query":{
            "query_string":{
                "default_field":"{field}",
                "query":"{keyword}"
            }
        }
    }
    POST {index}/_search
    {
        "query":{
            "query_string":{
                "fields":["{field1}","{field2}"],
                "query":"{keyword}"
            }
        }
    }
    ```
    - simple_query_string
      - query参数不支持运算符，使用default_operator参数代替
    ```
    POST {index}/_search
    {
        "query":{
            "simple_query_string":{
                "fields":["{field1}","{field2}"],
                "query":"{keyword}",
                "default_operator":"AND"
            }
        }
    }
    ```
- Aggregation 聚合
  - Bucket Aggregation(group by)
  ```
  GET {index}/_search
  {
      "size":0,
      "aggs":{
          "{agg_name}":{
              "terms":{
                  "field":"{field_name}"
              }
          }
      }
  }
  ```
  - Metric Aggregation(count,avg,max,min,sum)
   ```
  GET {index}/_search
  {
      "size":0,
      "aggs":{
          "{agg_name}":{
              "terms":{
                  "field":"{field_name}"
              },
             "aggs":{
                 "{agg_avg}":{
                     "avg":{
                         "field":"{field_name}"
                     }
                 }
             },
             "aggs":{
                 "{agg_max}":{
                     "max":{
                         "field":"{field_name}"
                     }
                 }
             },
             "aggs":{
                 "{agg_min}":{
                     "min":{
                         "field":"{field_name}"
                     }
                 }
             }
          }
      }
  }
  ```
  - Pipeline Aggregation
  - Matrix Aggregation
    
- 相关性算分的指标 Information Retrieval
  - Precision 查准率
  - Recall 查全率
  - Ranking 相关度排序
  - 相关性算分计算
    - 词频 TF(Term Frequency)
      - TF=检索词出现的次数/分档的总字数
    - 逆文档频率 IDF(Inverse Document Frequency)
      - IDF=log(全部文档数/检索词出现过的文档数)
    - TF-IDF 加权求和
      - TF-IDF=TF(keyword)*IDF(keyword)
    - BM25
      - 对TF-IDF进行了优化，算分不会趋于无穷增长
    - explain 参数可以查看算分的详情
    - profile 参数可以查看检索类型详情
    ```
    GET /test_user/_search
    {
      "profile": "true",
      "explain": true, 
      "query":{
        "match":{
          "name":"ZurkBob"
        }
      }
    }
    ```