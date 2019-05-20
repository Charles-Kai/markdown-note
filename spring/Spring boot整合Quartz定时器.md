# Spring boot整合Quartz定时器

## 一、概念

> [quartz官网](http://www.quartz-scheduler.org)是一个完全由 Java 编写的开源作业调度框架，结合数据库甚至可以做到分布式调度。目前参考的[RuoYi后台脚手架](https://gitee.com/y_project/RuoYi)的定时任务模块，支持在线（添加、修改、删除)任务调度，并记录执行日志作业结果。

### 3w原则：

1. question what：定时任务，比如定时答题、商家结算等需求，并且支持立即运行、暂停和禁止。
2. question why：借助quartz springboot生态和后台脚手架，快速实现定时调度功能。
3. how：实现**JobDetail**运行任务详情，**Trigger** 触发器定义触发规则，**Scheduler** 调度中心/容器注册多个 JobDetail 和 Trigger。Trigger 与 JobDetail 组合即可被Scheduler调用。

## 二、落地实现

> 根据[若依脚手架](http://doc.ruoyi.vip/#/standard/htsc?id=定时任务)的文档可知，定时任务工程模块为ruoyi-quartz，结合sql/quartz.sql导入关于定时器数据库表。当然这种做法需要数据库和bootstrap，为了简化，我采取的替代方案是保留定时和立即执行功能，抛弃手动在代码硬编码新加定时器，web管理面板则通过swagger触发任务调度立即执行一次。极端偷懒方式，@Scheduled(cron = "")放在在cotroller方法，同事推荐给我的😀。

#### 1、添加依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>
        </dependency>
```

#### 2、配置引导类

```java
//模块加载
@EnableScheduling
@EnableSwagger2
@SpringBootApplication
public class WeixinApplication {
......
}
```

#### 3、注册JobDetail和Trigger

```java
/**
 * 在线表达式：http://cron.qqe2.com/
 */
@Slf4j
@Configuration
public class QuartzConfig {
    public static final String TASK_CLASS_NAME = "reportNowTask";

    @Bean
    public JobDetail reportNowTask() {
        return JobBuilder.newJob(reportNowTask.class).withIdentity(TASK_CLASS_NAME).storeDurably().build();
    }

    @Bean
    public Trigger reportNowTaskTrigger(JobDetail reportNowTask) {
        //cronSchedule等于@Scheduled(cron = ""),但是通过注解无法配置jobkey
        return TriggerBuilder.newTrigger().forJob(reportNowTask)
                .withIdentity("reportNowTaskTrigger")
                .withSchedule(CronScheduleBuilder.cronSchedule("9 0 19 * * ?"))
                .build();
    }

}

```

#### 4、任务详情继承QuartzJobBean或者实现Job接口

```java
@Slf4j
public class reportNowTask extends QuartzJobBean {

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        ......//任务内容
    }
}

```

#### 5、立即执行

```java
@Slf4j
@Api(tags = "问题模块")
@RequestMapping(value = "question")
@RestController
public class QuestionController {

    /**
     * 任务调度立即执行一次
     */
    @PostMapping("/run")
    @ResponseBody
    public ResponseEntity run() throws SchedulerException {
        //api秘诀就在这里根据QuartzConfig jobKey触发作业调度
        scheduler.triggerJob(JobKey.jobKey(QuartzConfig.TASK_CLASS_NAME));
        return ResponseEntity.ok("执行成功！");
    }

}

```

### 反馈与建议

- 今天粗略简易版的定时任务,功能强大的请查看若依后台脚手架,github和码云有很多类似脚手架,但是我们有选择性copy学习才是重点😊。
- 邮箱：[caochikai@qq.com](