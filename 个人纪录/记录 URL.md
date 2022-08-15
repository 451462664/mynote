```shell
学习 8 步走，学习 Elasticsearch 为例，具体的做法是：
1. 搭建一个单机伪集群，搭建完成后看看安装路径下的文件和目录，看看配置文件有哪些配置项，不同的配置项会有什么样的影响。
2. 执行常用的操作，例如创建索引，插入、删除、查询文档，查看一下各种输出。
3. 研究其 基本原理 ，例如索引、分片、副本等，研究的时候要多思考，例如索引应该如何建，分片数量和副本数量对系统有什么影响等。
4. 和其他类似系统对比，例如 Solr、Sphinx，研究其 优点、缺点、适用场景 。
5. 模拟一个案例看看怎么应用。例如，假设我用 Elasticsearch 来存储淘宝的商品信息，我应该如何设计索引和分片。
6. 查看业界使用的案例，思考一下别人为何这么用；看看别人测试的结果，大概了解性能范围。
7. 如果某部分特别有兴趣或者很关键，可能去看源码，例如Elasticsearch的选举算法（我目前还没看^_^）。
8. 如果确定要引入，会进行性能和可用性测试。

# 设计原则
1.服务接口尽可能大粒度，每个服务方法应代表一个功能，而不是某功能的一个步骤，否则将地面临分布式事务问题.
2.dubbo 暂未提供分布式事务支持，同时可以减少系统间的网络交互.
3.服务接口建议以业务场景为单位划分，并对相近的业务做抽象,防止接口数量爆增(爆炸).
4.例:某一个接口有多个实现，做成一个接口，再在 dubbo 分组中多实现.
5.不建议使用过于抽象的通用接口，如 Map query(Map),这样的接口没有明确语义，会给后期维护带来不便.
# 超时处理
https://www.cnblogs.com/xuwc/p/8974709.html
https://www.codeprj.com/blog/88f1751.html
# 扩展屏问题
扩展显示器分屏展示多项目后窗口空白问题解决,通过 Alt+space+x最大化快捷键弹出窗口.
```

### 记录 URL

白板 https://excalidraw.com/
最棒的设计模式 https://java-design-patterns.com/patterns
清华大学开源镜像站 https://mirrors.tuna.tsinghua.edu.cn/
idea 设置 https://mp.weixin.qq.com/s/LxJZEO6i3PT7iJeDRMg05A
自考 https://zk.zjzs.net/Index/index.aspx
球鞋相关头像 https://www.sohu.com/a/229734289_430941
C 清理 https://www.zhihu.com/question/51654630

Break 易站 https://www.breakyizhan.com
Linux 常用命令整理 https://segmentfault.com/a/1190000011854356
动态规划 https://www.cnblogs.com/sdjl/articles/1274312.html

Spring 中文手册 https://www.simviso.com/doc/spring-framework-5.2.x-cn/
https://lfvepclr.gitbooks.io/spring-framework-5-doc-cn/content/
http://shouce.jb51.net/spring/
https://docs.gitcode.net/spring/guide/
SpringMVC 中文手册 https://linesh.gitbooks.io/spring-mvc-documentation-linesh-translation/content/
SpringBoot http://www.ityouknow.com/spring-boot.html
SpringBoot https://spring.io/projects/spring-boot/#overview
SpringBoot 中文文档 https://www.springcloud.cc/spring-boot.html
https://docshome.gitbooks.io/springboot/content/
springboot 参数绑定
https://juejin.im/post/5d614813f265da03ef7a1da7
https://juejin.im/post/5d66628ce51d4561cd246670
springcloud 中文网 https://www.springcloud.cc/mybatis
mybatis源码 https://www.cnblogs.com/zhjh256/p/8512392.html

ES 中文文档 http://doc.codingdict.com/elasticsearch/

Linux 下载&安装 & 鸟哥私房菜基础版
https://cloud.tencent.com/developer/article/1332409
http://mirror.nsc.liu.se/centos-store/6.9/isos/x86_64/
https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/27.html
安装 https://my.oschina.net/calmsnow/blog/3007126

位运算
https://link.zhihu.com/?target=http%3A//www.matrix67.com/blog/archives/263
https://link.zhihu.com/?target=http%3A//www.matrix67.com/blog/archives/264
https://link.zhihu.com/?target=http%3A//www.matrix67.com/blog/archives/266
https://link.zhihu.com/?target=http%3A//www.matrix67.com/blog/archives/268

算法 https://wizardforcel.gitbooks.io/the-art-of-programming-by-july/content/00.01.html

ali-email jipengfei.jpf@alibaba-inc.com
鸠摩搜书 https://www.jiumodiary.com/
精品下载 http://www.j9p.com/down/528341.html
在线制图 https://www.processon.com/
Redis深度历险 https://github.com/Zhengfangxing/Book/blob/master/Redis%E6%B7%B1%E5%BA%A6%E5%8E%86%E9%99%A9%EF%BC%9A%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5.pdf

汇编检测题 https://www.cnblogs.com/Base-Of-Practice/tag/%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80%20%E7%8E%8B%E7%88%BD/

Reactor https://projectreactor.io/docs/core/release/reference/
https://www.jianshu.com/p/7ee89f70dfe5

Typora 主题
https://blinkfox.github.io/2018/11/19/ruan-jian-gong-ju/markdown/vue-wen-dang-feng-ge-de-typora-zhu-ti/
https://zhuanlan.zhihu.com/p/133863913

flink https://confucianzuoyuan.github.io/flink-tutorial/book/chapter01-01-01-%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86.html

DDD
https://github.com/citerus/dddsample-core
https://qiyu2580.gitbooks.io/iddd/content/guide-to-this-book.html
https://www.zybuluo.com/iamfox/note/93864
https://blog.csdn.net/wangli13860426642
https://blog.csdn.net/gmingzhou/category_9603250.html

IDEA 破解码
备用提取地址：idea.medeming.com/jets
永久更新地址：idea.medeming.com/jihuo

idea 替换中文文档
https://blog.csdn.net/u013158317/article/details/94718481

联想笔记本拆键帽 https://haokan.baidu.com/v?pd=wisenatural&vid=2128010119704618348

拉勾网盘资料 5218587374

### 一些好的博客

oracle https://www.oracle.com/technetwork/cn/articles/java/index.html

小马哥 https://github.com/mercyblitz/segmentfault-lessons
汪明鑫 http://xinyeshuaiqi.cn/

知秋

1. https://github.com/muyinchen/simviso-Source-code-interpretation-sharing
2. https://muyinchen.github.io/archives
3. https://juejin.im/user/59c7640851882578e00ddf90
4. 问答集 https://www.yuque.com/server_mind/answer

尚硅谷springboot笔记 https://www.yuque.com/atguigu
aqs https://segmentfault.com/a/1190000016058789
永顺 https://segmentfault.com/u/yongshun/articles?page=1
说好不能打脸 https://blog.csdn.net/yinwenjie
Throwable https://www.throwx.cn/
设计模式 https://legacy.gitbook.com/@alleniverson
源码分析 https://www.cnblogs.com/java-chen-hao/
田小波 http://www.tianxiaobo.com
美团技术 https://tech.meituan.com
javadoop https://javadoop.com
java 干货 http://www.itsoku.com/
java 技术驿站 http://cmsblogs.com?vip=1
只会一点 java https://www.cnblogs.com/dennyzhangdd/
rxjava https://www.jianshu.com/nb/14302692 https://zhuanlan.zhihu.com/p/20687178
并发大师 doug lea http://ifeve.com/doug-lea/
设计模式 https://gof.quanke.name/    https://www.jianshu.com/u/8bc5f4428ca2
敖丙 https://www.cnblogs.com/aobing/
spring 博客 https://blog.csdn.net/likun557
bugstack虫洞栈
https://bugstack.cn/itstack-demo-jvm/itstack-demo-jvm.html
https://bugstack.blog.csdn.net/
子路 https://juejin.im/user/5e82aafee51d4546cd2fd158
nacos https://blog.csdn.net/wangwei19871103
flink https://flink-learning.org.cn/
flink 源码
https://juejin.cn/user/2875978147955741/posts
http://www.54tianzhisheng.cn/2019/03/14/Flink-code-structure/

### 好的开源项目

设计模式 https://github.com/iluwatar/java-design-patterns
后端学习
https://github.com/fuzhengwei/CodeGuide/wiki
https://github.com/Snailclimb/javaGuide
https://github.com/doocs/advanced-java
https://github.com/CyC2018/CS-Notes
arthas https://github.com/alibaba/arthas
高并发解决思路
https://github.com/qiurunze123/miaosha
https://github.com/macrozheng/mall
https://github.com/Snailclimb/JavaGuide
浙江大学 https://github.com/QSCTech/zju-icicles
中国科学技术大学
https://github.com/USTC-Resource/USTC-Course
https://ustc-resource.github.io/USTC-Course
中山大学 https://link.zhihu.com/?target=https%3A//github.com/sysuexam/SYSU-Exam
B 站的公开课
https://link.zhihu.com/?target=https%3A//github.com/wenhan-wu/OpenCourseCatalog
https://link.zhihu.com/?target=https%3A//space.bilibili.com/12721139

龙果支付系统（roncoo-pay）是国内首款开源的互联网支付系统，拥有独立的账户体系、用户体系、支付接入体系、支付交易体系、对账清结算体系。目标是打造一款集成主流支付方式且轻量易用的支付收款系统，满足互联网业务系统打通支付通道实现支付收款和业务资金管理等功能。
https://github.com/roncoo/roncoo-pay
https://gitee.com/roncoocom/roncoo-pay
https://gitee.com/52itstyle/spring-boot-pay

Jcseg 是基于 mmseg 算法的一个轻量级中文分词器，同时集成了关键字提取，关键短语提取，关键句子提取和文章自动摘要等功能，并且提供了一个基于 Jetty 的 web 服务器，方便各大语言直接http调用，同时提供了最新版本的 lucene，solr和elasticsearch 的分词接口！
https://gitee.com/lionsoul/jcseg

### 摸鱼网站

小霸王游戏机 https://www.yikm.net/
鱼塘热搜 https://mo.fish/

### 虚拟机激活码

UG5J2-0ME12-M89WY-NPWXX-WQH88
GA590-86Y05-4806Y-X4PEE-ZV8E0
YA18K-0WY8P-H85DY-L4NZG-X7RAD
UA5DR-2ZD4H-089FY-6YQ5T-YPRX6
ZF582-0NW5N-H8D2P-0XZEE-Z22VA

VMware 2019 v15.x 永久许可证激活密钥
YG5H2-ANZ0H-M8ERY-TXZZZ-YKRV8
UG5J2-0ME12-M89WY-NPWXX-WQH88
UA5DR-2ZD4H-089FY-6YQ5T-YPRX6

v16.x
永久许可证激活密钥：
VMware激活密钥（通用批量永久激活许可）
16：ZF3R0-FHED2-M80TY-8QYGC-NPKYF
15：FC7D0-D1YDL-M8DXZ-CYPZE-P2AY6
12：ZC3TK-63GE6-481JY-WWW5T-Z7ATA
10：1Z0G9-67285-FZG78-ZL3Q2-234JG

VMware Workstation Pro 16 许可证密钥，批量永久激活
ZF3R0-FHED2-M80TY-8QYGC-NPKYF
YF390-0HF8P-M81RQ-2DXQE-M2UT6
ZF71R-DMX85-08DQY-8YMNC-PPHV8
ZF3R0-FHED2-M80TY-8QYGC-NPKYF

来战
https://job.alibaba.com/