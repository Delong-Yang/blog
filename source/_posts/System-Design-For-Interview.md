---
title: System Design For Interview
date: 2017-07-03 15:45:22
tags:
---

## 入门

* <1>[https://www.hiredintech.com/courses/system-design][1]
* <2>[https://www.youtube.com/watch?v=-W9F__D3oY4][2]

这是<[1]>里面提到的资料 是Harvard web app课的最后一节 讲scalability 里面会讲到很多基础概念比如Vertical scaling, orizontal scaling, Caching, Load balancing,Database replication, Database partitioning 还会提到很多基本思想比如avoid single point of failure
<!-- more --> 
* <3>[http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones][3]
<[1]> 里面提到的 Scalability for Dummies.

## 进阶 
这部分你会遇到很多新名词，每当你遇到一个不懂的概念时，多google一下，看看这个概念或者技术是什么意思，优点和缺点各是什么，什么时候用，这些你都知道以后，你就可以把他运用到面试中，让面试官刮目相看了。

* <4> [http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html][4]

Database Sharding是一个很重要的概念 建议看一看

* <5> [http://highscalability.com/all-time-favorites/][5]
这个里面会讲到很多非常流行的网站架构是如何实现的 比如Twitter, Youtube, Pinterest, Google等等.我的建议是看5-6个,然后你应该已经建立起了一些基本的意识,还有知道了某些技术和产品的作用和mapping 比如说到cache你会想到memcached和Redis,说到load balancer你会想到 Amazon ELB, F5一类的.

* <6> [http://www.infoq.com/][6]
<[5]>里面很多的文章都会有链接,其中有很多会指向这个网站,这里面有很多的tech talk

* <7> [https://www.facebook.com/Engineering/notes][7]
Facebook非常好的技术日志 会讲很多facebook的feature怎么实现的 比如facebook message:<https://www.facebook.com/notes/facebook-engineering/the-underlying-technology-of-messages/454991608919> 建议看看 尤其是准备面facebook的同学
这有一个facebook talk讲storage的<https://www.youtube.com/watch?v=5RfFhMwRAic>

* <8> 一些国内网站上的资料
[http://blog.csdn.net/sigh1988/article/details/9790337](http://blog.csdn.net/sigh1988/article/details/9790337)
[http://blog.csdn.net/v_july_v/article/details/6279498](http://blog.csdn.net/v_july_v/article/details/6279498)

* <9> 一些很有用的概念
    *  Distributed Hash Table
    * Eventual Consistency vs Strong Consistency
    * Read Heavy vs Write Heavy
    * Consistent Hashing
    * Sticky Sessions
    * Structured Data(uses DynamoDB) vs Unstructured Data(uses S3)
      http://smartdatacollective.com/michelenemschoff/206391/quick-guide-structured-and-unstructured-data http://stackoverflow.com/questions/18678315/amazon-s3-or-dynamodb

* <10> 给有兴趣深入研究的人看的
Mining Massive Datasets --讲很多big data和data mining的东西
Big Data: Principles and best practices of scalable realtime data systems(http://www.amazon.com/gp/product/1617290343) --
twitter的前员工讲述如何处理实时数据 目前市面上讲解big data最好的一本书

* <11> 其他资料
http://highscalability.com/blog/2013/10/28/design-decisions-for-scaling-your-high-traffic-feeds.html

## 小结
看多了以后，心里有了一个大框架,一个基本的distributed system是怎么搭起来的, 然后心里有很多if condition,如果要是满足这个条件,我应该用什么技术. 比如如果read heavy那么用cache会提升performance之类的. 同时知道应该避免什么东西, 比如避免single point of failure,再比如时间和空间的tradeoff在read heavy的时候应该倾向于时间,Write heavy的时候倾向于空间等等.

[1]: https://www.hiredintech.com/courses/system-design
[2]: https://www.youtube.com/watch?v=-W9F__D3oY4
[3]: http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones
[4]: http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html
[5]: http://highscalability.com/all-time-favorites/
[6]: http://www.infoq.com/
[7]: https://www.facebook.com/Engineering/notes