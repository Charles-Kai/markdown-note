# JVM记录

## 一、错误

1. 背景：[自动答题爬虫](https://github.com/caochikai/webmagicForPros)由于买的腾讯云1核2G1M，Jenkins、springboot爬虫和火狐浏览器驱动。
2. 知识背景：[JDWP](https://www.jianshu.com/p/134bd5b913c5)：调试网络协议(Java Debug Wire Protocol)；调试线协议；[jvmti](https://juejin.im/post/5b0925ec51882538aa1ee248)：（Java Virtual Machine Tool Interface）jvm代理；
3. 猜测：Selenium通过driver驱动Firefox浏览器，多次无法关闭浏览器造成内存无法释放最后溢出； 
4. 引用文章（有兴趣可深入了解）：[jvmti agent黑科技](https://blog.csdn.net/duqi_2009/article/details/94518203)，阿里云云监控；

```
FATAL ERROR in native method: JDWP Can't allocate jvmti memory, jvmtiError=JVMTI_ERROR_OUT_OF_MEMORY(110)
```

### 反馈与建议

- 立个flag：不定期更新，一更一周。
- markdown原文件在github里面，感谢各位大佬看官star，面试我要往脸上贴金哈哈哈😂。
- 邮箱：[caochikai@qq.com](mailto:caochikai@qq.com)，有问题发邮件。
