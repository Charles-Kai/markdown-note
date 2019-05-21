## springboot整合elasticsearch

## 一、概念

> [elasticsearch官网](https://www.elastic.co/cn/)是一个分布式多用户能力的全文搜索引擎，也是一个具有RESTful web接口的java应用。目前开源软件商业比较不错的例子，与Solr一样都是基于Lucene，大数据hadoop也是脱胎于Lucene。Solr开源而且生态比较成熟，elasticsearch目前最火也是商业应用方面非常好的搜索引擎。

### 3w原则：

1. question what：常见站内/app内搜索服务需求：商品文章的模糊搜索，精确搜索，拼音搜索 。
2. question why：借助elasticsearch和analysis-ik中文分词器，快速实现搜索服务功能。
3. how：在微服务当中，通常利用mq消息中间件来同步数据集群搜索服务（脚手架里没有mq），借助ElasticsearchTemplate（spring 模板工具类强大）API维护索引和搜索查询。

## 二、落地实现

> 根据[码云企业级搜索脚手架](https://gitee.com/11230595/springboot-elasticsearch)的文档可知，注意版本为Springboot2.1.1+elasticsearch6.5.3，elasticsearch和analysis-ik插件版本必须统一，而且新版本elasticsearch 7不适用于该工程。这个参考工程的中文分词搜索效果不太理想，一般富文本的内容进入索引之前要利用字符过滤器清洗不正常的字符，我会在例子会修改原来的代码。通常为了保证索引时覆盖度和搜索时准确度,索引分词器采用ik_max_word,搜索分析器采用ik_smart模式。具体elasticsearch6.5.3的安装过程请参考码云的README.md，目前正在在公司项目使用请放心，单元测试的效果也非常nice。

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

#### 3、DWMQSender包装rabbitTemplate发送消息同步数据

```java
//服务类的简写如下
{
//注入sender    
@Autowired
DWMQSender sender;

//发送写法ArticlesMessage extends DWMQMessage<消息内容类型>
JSONObject jsonObject = JSON.parseObject(JSON.toJSONString(articles));
 jsonObject.put(Groups.ACTION, Groups.ADD);
 sender.sendMessage(new ArticlesMessage(jsonObject));
}
//封装发送mq message
import com.alibaba.fastjson.JSON;
import com.dwalk.common.exception.EU;
import com.dwalk.common.mq.mto.DWMQMessage;
import com.dwalk.common.utils.SU;
import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.support.CorrelationData;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.concurrent.atomic.AtomicLong;

/**
 * 消息成功发送到MQ服务器后的回调确认
 */
@Slf4j
@Component
public class DWMQSender {

    @Autowired
    RabbitTemplate rabbitTemplate;

    @Autowired
    DWMQRetry retry;
    
    
	//
    public void sendMessage(DWMQMessage mto) {

        if( mto.getObj()==null ) {
            EU.te("消息内容为空");
        }
        if( SU.isNull( mto.getRoutingKey()) ) {
            EU.te("路由规则为空");
        }

        log.info(String.format("准备发送【%s】", mto.getInfo()));

        mto.setId(retry.generateId());

        if( SU.isNull(mto.getExchange()) && mto.getExpire()<1 ) {
            //默认的没有交换器，则直接发送到指定的队列
            rabbitTemplate.convertAndSend(mto.getRoutingKey(), mto.getObj(), new CorrelationData(mto.getId()));
        } else if (mto.getExpire()<1){
            //经过交换器，按路由规则匹配队列
            rabbitTemplate.convertAndSend(mto.getExchange(), mto.getRoutingKey(), mto.getObj(), new CorrelationData(mto.getId()));
        } else {
            //默认的没有交换器，则直接发送到指定的队列，发送延迟消息
            rabbitTemplate.convertAndSend(mto.getRoutingKey(), mto.getObj(), message -> {
                message.getMessageProperties().setExpiration(mto.getExpire()+"");
                return message;
            }, new CorrelationData(mto.getId()));
        }

        mto.setCtime(System.currentTimeMillis());
        retry.add(mto, this);
    }
}

```

#### 4、RabbitListener接收到消息同步数据

```java
/**
 * 搜索服务接收到管理后台用户修改文章同步消息
 */
@Component
@Slf4j
//配置mq消息队列，接收文件同步消息
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

#### 5、公共搜索方法

```java
/**
     * 高亮显示，返回分页
     * @auther: zhoudong
     * @date: 2018/12/18 10:29
     */
    @Override
    public IPage<Map<String, Object>> queryHitByPage(int pageNo, int pageSize, String keyword, String indexName, String... fieldNames) {
        // 构造查询条件,使用标准分词器.
//      原本脚手架的：QueryBuilder matchQuery = createQueryBuilder(keyword, fieldNames);
//      修改成把搜索内容字符串转换成单字符的数组，遍历并BoolQueryBuilder.should至少有一个语句要匹配，与 OR 等价。        
        char[] chars = keyword.toCharArray();
        BoolQueryBuilder bool = QueryBuilders.boolQuery();
        for (char word : chars) {
            QueryBuilder queryBuilder = createQueryBuilder(String.valueOf(word), fieldNames);
            bool.should(queryBuilder);
        }
        // 设置高亮,使用默认的highlighter高亮器
        HighlightBuilder highlightBuilder = createHighlightBuilder(fieldNames);
        // 设置查询字段
        SearchResponse response = elasticsearchTemplate.getClient().prepareSearch(indexName)
                .setQuery(bool)
                .highlighter(highlightBuilder)
                .setFrom((pageNo - 1) * pageSize)
                .setSize(pageNo * pageSize) // 设置一次返回的文档数量，最大值：10000
                .get();


        // 返回搜索结果
        SearchHits hits = response.getHits();

        Long totalCount = hits.getTotalHits();
        IPage<Map<String, Object>> page = new Page<>(pageNo, pageSize, totalCount);
        page.setRecords(getHitList(hits));
        return page;
    }
```

### 反馈与建议

- 今天终于出了elasticsearch文章，以后我会在对应专题的文章放出相关的百度云资源，这些都是网上流传比较广的资源，想找好的学习资源也可以与我合伙买绝版视频，有钱买正版吧（作者很穷，找工作从来没有造假包装，只能混成这个卵样😭，世道维艰，如果不是感觉做码农还算有点天赋，早就转行了）。
- 百度云 :[下载街/01.Elasticsearch顶尖高手系列课程](https://pan.baidu.com/s/1d2adCgJMuwt6UmAxEMRYxA#list/path=%2F)，密码：iw7f
- 邮箱：[caochikai@qq.com](mailto:caochikai@qq.com)

