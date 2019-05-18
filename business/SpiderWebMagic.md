---
title: SpiderWebMagic
date: 2017-01-30 20:50:27
tags:
---
# WebMagic爬虫框架--京东图书

@(爬虫)[框架|爬虫|Demo总结]

**WebMagic**!项目代码分为核心(webmagic-core)和扩展(webmagic-extension)两部分(jar包)。Downloader、PageProcessor、Scheduler、Pipeline这四大组件对应爬虫生命周期中的下载、处理、管理和持久化等功能。

| 名称        | 功能          |
| ------------- |:-------------:| 
| Downloader | 基础。利用httpClient作为下载工具，下载页面内容便于后续处理解析; |
| Page | 网页内容对象。 指根据url下载到的页面内容，包括页面dom元素，css样式，javascript等;|  
| Pageprocess| 爬虫的核心。 负责解析页面，抽取有用信息，可采用css(),$(),xpath()方法对特定页面元素进行抽取; |  
| Site|  网站设置。设置网站domain，cookies,header,重试次数,访问间隔时间等;     |  
| Scheduler |   抓取页面队列。 管理待抓取的URL，以及一些去重的工作，将目标url内容push到抓取队列中;    |  
| Pipeline|   输出，收尾。 负责抽取结果的处理，包括计算、持久化到文件、数据库;    |  
| Spider|  爬虫的入口类 采用链式设计，通过它来设定多线程，页面解析器，调度以及输出方式等。     |  



### WebMagic官方链接：
- [官网](http://webmagic.io/) 包含官方文档和源码,以及相应的实例；
-  [github](https://github.com/code4craft/webmagic) 仓库保存最新版本；
-  [oschinamayun码云](http://git.oschina.net/flashsword20/webmagic) 包含所有编译好的依赖包；
    

####  爬取京东图书(https://book.jd.com/)
 
##### 在商品列表网页抓取如下商品信息
* 商品名：商品名称
* 商品网页：显示商品详细信息的网页地址。
* 市场价格：京东给出的市面价格
* 京东价格：京东的优惠价。

##### 关于ajax价格链接地址格式：http://p.3.cn/prices/mgets?skuIds=J_ + 商品ID，抓取格式为json。
* https://item.jd.com/12087016.html ：图书详情页；
* http://p.3.cn/prices/mgets?skuIds=J_12087016  （例）价格ajax请求链接；
* [{"id":"J_12087016","p":"60.80","m":"90.00","op":"60.80"}]  抓取结果 id + 京东实际价格+市场价格

-------------------

### 代码块
``` java
import us.codecraft.webmagic.Page;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.Spider;
import us.codecraft.webmagic.processor.PageProcessor;

import java.util.ArrayList;
import java.util.List;

public class JDPageProcesser implements PageProcessor {
    private Site site = Site.me().setRetryTimes(3).setSleepTime(3000).setCharset("GBK");
    private static int size = 0;// 共抓取到的图书数量
    //抓取商品信息集合
    private static List<String> name = new ArrayList<String>();//所有的书名
    private static List<String> author = new ArrayList<String>();//所有的作者
    private static List<Double> prices = new ArrayList<Double>();//所有的价格

    @Override
    public void process(Page page) {
        //图书主页
		// https://item.jd.com/12004057.html
        if (!page.getUrl().regex("https://item.jd.com/\\d{8}.html").match()&!page.getUrl().regex("p.3.cn/prices/mgets").match()) {
            // 主页中添加商品详情页到计划url
            List<String> detail = page.getHtml().links().regex("//item.jd.com/\\d{8}.html").replace("//", "https://").all();
            //控制抓取商品的数量
            if (detail.size()>0) {
                for (int i = 0; i < 4; i++) {
                    String url = detail.get(i);
                    System.out.println("url:" + url.replace("https://item.jd.com", "").replace("/", "http://p.3.cn/prices/mgets?skuIds=J_").replace(".html", ""));
                    //列队添加一条详情页后面追加一条价格ajax链接
                    page.addTargetRequest(url);
                    page.addTargetRequest(url.replace("https://item.jd.com", "").replace("/", "http://p.3.cn/prices/mgets?skuIds=J_").replace(".html", ""));
                }
            }
        }
        if (page.getUrl().regex("https://item.jd.com/\\d{8}.html").match()) {
            // 商品详情页
            size++;
            name.add(page.getHtml().xpath("//div[@id=name]/h1/text()").get());//添加书名
            author.add(page.getHtml().xpath("//div[@id=p-author]/a/text()").get());//添加作者
        }
        if (page.getUrl().regex("p.3.cn/prices/mgets").match()) {
            //ajax商品id对应价格json接口
            prices.add(Double.parseDouble(page.getHtml().replace("&quot", "").regex("p;:;.+;,;m").regex("\\d+\\.\\d+").get()));//添加价格
        }
    }


    @Override
    public Site getSite() {
        return site;
    }

    //获取所有信息导入DAO,持久化层待实现
    private static void getAll() {
        System.out.println("Size"  + name.size() + author.size() + prices.size());
        for (int i = 0; i < name.size(); i++) {
            JDLog model = new JDLog();
            model.setName(name.get(i));
            model.setAuthor(author.get(i));
            model.setPrices(prices.get(i));
            System.out.println("书名:" + model.getName());
            System.out.println("作者:" + model.getAuthor());
            System.out.println("价格:" + model.getPrices());
        }

    }

    public static void main(String[] args) {
        long startTime, endTime;
        System.out.println("【爬虫开始】请耐心等待一大波数据到你碗里来...");
        startTime = System.currentTimeMillis();
        // 从京东图书开始抓，开启5个线程，启动爬虫
        Spider.create(new JDPageProcesser()).addUrl("https://book.jd.com/").thread(3).run();
        endTime = System.currentTimeMillis();
        getAll();
        System.out.println("【爬虫结束】共抓取" + size + "本图书，耗时约" + ((endTime - startTime) / 1000) + "秒，已保存到数据库，请查收！");
    }
}
```

##  以后将会坚持更新!

##  反馈与建议
- 邮箱：<caochikai@qq.com>

---------




