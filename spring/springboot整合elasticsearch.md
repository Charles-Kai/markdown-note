## springboot整合elasticsearch

## 一、概念

> [elasticsearch官网](https://www.elastic.co/cn/)是一个分布式多用户能力的全文搜索引擎，也是一个具有RESTful web接口的java应用。目前开源软件商业比较不错的例子，与Solr一样都是基于Lucene，大数据hadoop也是脱胎于Lucene。Solr开源而且生态比较成熟，elasticsearch目前最火也是商业应用方面非常好的搜索引擎。

### 3w原则：

1. question what：常见站内/app内搜索服务需求：商品文章的模糊搜索，精确搜索，拼音搜索 。
2. question why：借助elasticsearch和analysis-ik中文分词器，快速实现搜索服务功能。
3. how：在微服务当中，通常利用mq消息中间件来同步数据集群搜索服务，借助ElasticsearchTemplate（spring 模板工具类强大）API维护索引和搜索查询。

## 二、落地实现

> 根据[码云企业级搜索脚手架](https://gitee.com/11230595/springboot-elasticsearch)的文档可知，注意版本为Springboot2.1.1+elasticsearch6.5.3，elasticsearch和analysis-ik插件版本必须统一，而且新版本elasticsearch 7不适用于该工程。这个参考工程的中文分词搜索效果不太理想，一般富文本的内容进入索引之前要利用字符过滤器清洗不正常的字符，我会在例子总修改原来的代码。通常为了保证索引时覆盖度和搜索时准确度,索引分词器采用ik_max_word,搜索分析器采用ik_smart模式。具体elasticsearch6.5.3的安装过程请参考码云的README.md，目前正在在公司项目使用请放心，单元测试的效果也非常nice。

#### 1、添加依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

#### 2、配置引导类

```properties
#============================
# 默认的节点名称elasticsearch
spring.data.elasticsearch.cluster-name=my-application
# elasticsearch 调用地址，多个使用“,”隔开
spring.data.elasticsearch.cluster-nodes=localhost:9300
```

#### 3、RabbitListener接收到消息同步数据

```java
/**
 * 管理后台用户修改文章同步消息接收
 */
@Component
@Slf4j
@RabbitListener(queues = DirectMQConfig.DIRECT_ARTICLES_ELASTIC_QUEUE)
public class ArticlesReceiver extends DWMQBaseReceiver<String> {
    @Autowired
    private ArticleETOService etoService;

    @Override
    public Class getClazz() {
        return String.class;
    }

    @Override
    public boolean processMessage(String mto) {
        log.info("Received 管理后台用户-修改文章同步消息接收:{}", mto);
        JSONObject jsonObject = JSON.parseObject(mto);
        String action = jsonObject.getString(Groups.ACTION);
        //删除操作要删除索引，更新操作先删除后
        ArticleETO articles = JSON.parseObject(jsonObject.toJSONString(), ArticleETO.class);
        String articlesId = articles.getId();
        switch (action) {
            case Groups.ADD:
                etoService.save(articles);
                break;
            case Groups.UPDATE:
                etoService.delete(articlesId);
                etoService.save(articles);
                break;
            case Groups.DELETE:
                etoService.delete(articlesId);
                break;
        }
        return true;
    }
}
```

#### 4、任务详情继承QuartzJobBean或者实现Job接口

```java

```

#### 5、立即执行

```java

```

### 反馈与建议

- 。
- 邮箱：[caochikai@qq.com](

## elasticsearch

