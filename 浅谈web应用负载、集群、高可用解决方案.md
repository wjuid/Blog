# 浅谈web应用负载、集群、高可用解决方案

​					[![img](https://s5.51cto.com//wyfs02/M02/9C/0F/wKioL1lrgz7CXyqaAAARf5fUuxg290_middle.jpg)](https://blog.51cto.com/anfishr) 			

[一百个小排](https://blog.51cto.com/anfishr)0人评论[5658人阅读](javascript:;)[2017-10-09 22:12:31](javascript:;)

**1、熟悉几个组件**

**1.1、apache**
   ——  它是Apache软件基金会的一个开放源代码的跨平台的网页服务器，属于老牌的web服务器了，支持基于Ip或者域名的虚拟主机，支持代理服务器，支持安 全Socket层(SSL)等等，目前互联网主要使用它做静态资源服务器，也可以做代理服务器转发请求(如：图片链等)，结合tomcat等 servlet容器处理jsp。
**1.2、ngnix**
   —— 俄罗斯人开发的一个高性能的  HTTP和反向代理服务器。由于Nginx 超越 Apache 的高性能和稳定性，使得国内使用 Nginx 作为 Web  服务器的网站也越来越多，其中包括新浪博客、新浪播客、网易新闻、腾讯网、搜狐博客等门户网站频道等，在3w以上的高并发环境下，ngnix处理能力相当 于apache的10倍。
   参考：apache和tomcat的性能分析和对比(http://blog.s135.com/nginx_php_v6/)
**1.3、lvs**
   —— Linux Virtual  Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。由毕业于国防科技大学的章文嵩博士于1998年5月创立，可以实现 LINUX平台下的简单负载均衡。了解更多，访问官网：http://zh.linuxvirtualserver.org/。

**1.4、HAProxy**

   —— HAProxy提供**高可用性**、**负载均衡**以及基于TCP和HTTP应用的代理，**支持虚拟主机**， 它是免费、快速并且可靠的一种解决方案。HAProxy特别适用于那些负载特大的web站点，  这些站点通常又需要会话保持或七层处理。HAProxy运行在当前的硬件上，完全可以支持数以万计的并发连接。并且它的运行模式使得它可以很简单安全的整 合进您当前的架构中， 同时可以保护你的web服务器不被暴露到网络上.
**1.5、keepalived**
   —— 这里说的keepalived不是apache或者tomcat等某个组件上的属性字段，它也是一个组件，可以实现web服务器的高可用(HA  high  availably)。它可以检测web服务器的工作状态，如果该服务器出现故障被检测到，将其剔除服务器群中，直至正常工作后，keepalive会自 动检测到并加入到服务器群里面。实现主备服务器发生故障时ip瞬时无缝交接。它是LVS集群节点健康检测的一个用户空间守护进程，也是LVS的引导故障转 移模块（director  failover）。Keepalived守护进程可以检查LVS池的状态。如果LVS服务器池当中的某一个服务器宕机了。keepalived会通过一 个setsockopt呼叫通知内核将这个节点从LVS拓扑图中移除。

Keepalived详解：https://my.oschina.net/piorcn/blog/404644
**1.6、memcached**
   —— 它是一个高性能分布式内存对象缓存系统。当初是Danga  Interactive为了LiveJournal快速发展开发的系统，用于对业务查询数据缓存，减轻数据库的负载。其守护进程(daemon)是用C写 的，但是客户端支持几乎所有语言(客户端基本上有3种版本[memcache client for  java;spymemcached;xMecache])，服务端和客户端通过简单的协议通信；在memcached里面缓存的数据必须序列化。
**1.7、terracotta**
   ——  是一款由美国Terracotta公司开发的著名开源Java集群平台。它在JVM与Java应用之间实现了一个专门处理集群功能的抽象层，允许用户在不 改变系统代码的情况下实现java应用的集群。支持数据的持久化、session的复制以及高可用(HA)。详细参 考：http://topmanopensource.iteye.com/blog/1911679

**2、关键术语**
**2.1、负载均衡（load balance）**
 
在互联网高速发展的时代，大数据量、高并发等是互联网网站提及最多的。如何处理高并发带来的系统性能问题，最终大家都会使用负载均衡机制。它是根据某种负载策略把请求分发到集群中的每一台服务器上，让整个服务器群来处理网站的请求。
公司比较有钱的，可以购买专门负责负载均衡的硬件（如：F5）,效果肯定会很好。对于大部分公司，会选择廉价有效的方法扩展整个系统的架构，来增加服务器的吞吐量和处理能力，以及承载能力。

**2.2、集群（Cluster）**

 用N台服务器构成一个松耦合的多处理器系统(对外来说，他们就是一个服务器)，它们之间通过网络实现通信。让N台服务器之间相互协作，共同承载一个网站的请求压力。

**2.3、高可用（HA）**

 在集群服务器架构中，当主服务器故障时，备份服务器能够自动接管主服务器的工作，并及时切换过去，以实现对用户的不间断服务。ps：这里我感觉它跟故障转移(failover)是一个意思，看到的网友给个解释，谢谢？

**2.4、session复制/共享**

 在 访问系统的会话过程中，用户登录系统后，不管访问系统的任何资源地址都不需要重复登录，这里面servlet容易保存了该用户的会话(session)。 如果两个tomcat(A、B)提供集群服务时候，用户在A-tomcat上登录，接下来的请求web服务器根据策略分发到B-tomcat，因为B- tomcat没有保存用户的会话(session)信息，不知道其登录，会跳转到登录界面。
这时候我们需要让B-tomcat也保存有A-tomcat的会话，我们可以使用tomcat的session复制实现或者通过其他手段让session共享。

**3、常用web集群**

**3.1、tomcat集群方案**
 apache+tomcat；ngnix+tomcat；lvs+ngnix+tomcat； 大家比较熟悉的是前两种。(lvs负责集群调度，nginx负责静态文件处理，tomcat负责动态文件处理[最优选择])。  以apache+tomcat集群为例，简单说一下：
 1、他们之间的通信有三种方式：ajp_proxy、mod_jk链接器、http_proxy。具体参考：http://www.ibm.com/developerworks/cn/opensource/os-lo-apache-tomcat/
 2、apache的分发策略有4种。权重(默认)、流量(bytraffic)、请求次数(byRequests)、繁忙程度(byBusyness根据活跃请求数的多少)
 3、apache支持stickysession(粘性session)，即为：访问用户访问了A-tomcat，那么他的所有请求都会转发到A-tomcat，而不会到B-tomcat。[这样的负载均衡效果不好，适用于小型网站，下面说非粘性session]
 4、它们之间的架构如图1：


![img](http://dl2.iteye.com/upload/p_w_upload/0096/4084/bb559448-9c44-3aef-9a6c-82c6b7d0ffe3.jpg)

问题1：只有一个web服务器，明显的单点故障。如果该apache出现问题，整个网站就会瘫痪。

 

**3.2、session复制**


   如果不采用stickysession(粘性session)，那么我们可以采用tomcat的session复制使所有节点tomcat的会话相 同，tomcat使用组播技术，只要集群中一个tomcat节点的session发生改变，会广播通知所有tomcat节点发生改变。

问题2：据网友测试，当tomcat节点数达到4个以上时候，集群性能呈线性下滑；另外当用户访问量大到一定程度，会话内容随之增多，tomcat节点相互之间通信产生大量的网络消耗，产生网络阻塞，整个集群的吞吐量不能再上升。


**4、高可用(HA)和session共享(解决上面提到的两个问题)**


**4.1、使用lvs+keepalive实现集群高可用，达到更健壮的LB**
 我们可以做前端使用lvs来做负载均衡，根据lvs的8种调度算法(可设置)，分发请求到对应的web服务器集群上。lvs做双机热备，通过keepalived模块能够达到故障自动转移到备份服务器，不间断提供服务，结构如图2：

![img](http://dl2.iteye.com/upload/p_w_upload/0096/4662/5389ba90-3091-3b3b-a149-fbf47fc6f736.jpg)
 
 说 明：据查询了解，一般在WEB端使用的负载均衡比较多的是HAProxy+keepalived+nginx；数据库mysql集群使用 Lvs+keepalived+mysql实现。因为HAProxy和nginx一样是工作在网络7层之上，并且前者弥补了nginx的一些缺点如 session的保持，cookie的引导等，且它本身是个负责均衡软件，处理负载均衡上面必然优于nginx；lvs比较笨重，对于比较庞大的网络应用 实施比较复杂，虽然它运行在网络4层之上，仅做分发没有流量产生，但是它不能做正则处理也不能也不能做动静分离，所以一般用lvs+keepalived 或heatbeat做数据库层的负载均衡。

​		



现在网站发展的趋势对网络负载均衡的使用是随着网站规模的提升根据不同的阶段来使用不同的技术：

一种是通过硬件来进 行进行，常见的硬件有比较昂贵的NetScaler、F5、Radware和Array等商用的负载均衡器，它的优点就是有专业的维护团队来对这些服务进 行维护、缺点就是花销太大，所以对于规模较小的网络服务来说暂时还没有需要使用；另外一种就是类似于LVS/HAProxy、Nginx的基于Linux 的开源免费的负载均衡软件策略,这些都是通过软件级别来实现，所以费用非常低廉，所以我个也比较推荐大家采用第二种方案来实施自己网站的负载均衡需求。

PV达到了亿级/日的访问量，最前端用的是HAProxy+Keepalived双机作的负载均衡器  /反向代理，整个网站非常稳定；这让我更坚定了以前跟老男孩前辈聊的关于网站架构比较合理设计的架构方案：即Nginx  /HAProxy+Keepalived作Web最前端的负载均衡器，后端的MySQL数据库架构采用一主多从，读写分离的方式，采用 LVS+Keepalived的方式。

在这里我也有一点要跟大家申明下：很多朋友担心软件级别的负载均衡在高并发流量冲击下的稳定情况， 事实是我们通过成功上线的许多网站发现，它们的稳  定性也是非常好的，宕机的可能性微乎其微，所以我现在做的项目，基本上没考虑服务级别的高可用了。相信大家对这些软件级别的负载均衡软件都已经有了很深的 的认识，下面我就它们的特点和适用场合分别说明下。

LVS：使用集群技术和Linux操作系统实现一个高性能、高可用的服务器，它具有很好的可伸缩性（Scalability)、可靠性（Reliability)和可管理性（Manageability)，感谢章文嵩博士为我们提供如此强大实用的开源软件。

**LVS的特点是：
**

1. 抗负载能力强、是工作在网络4层之上仅作分发之用，没有流量的产生，这个特点也决定了它在负载均衡软件里的性能最强的；
2. 配置性比较低，这是一个缺点也是一个优点，因为没有可太多配置的东西，所以并不需要太多接触，大大减少了人为出错的几率；
3. 工作稳定，自身有完整的双机热备方案，如LVS+Keepalived和LVS+Heartbeat，不过我们在项目实施中用得最多的还是LVS/DR+Keepalived；
4. 无流量，保证了均衡器IO的性能不会收到大流量的影响；
5. 应用范围比较广，可以对所有应用做负载均衡；
6. 软件本身不支持正则处理，不能做动静分离，这个就比较遗憾了；其实现在许多网站在这方面都有较强的需求，这个是Nginx/HAProxy+Keepalived的优势所在。
7. 如果是网站应用比较庞大的话，实施LVS/DR+Keepalived起来就比较复杂了，特别后面有Windows Server应用的机器的话，如果实施及配置还有维护过程就比较复杂了，相对而言，Nginx/HAProxy+Keepalived就简单多了。站长教学网 eduyo.com

**Nginx的特点是：
**

1. 工作在网络的7层之上，可以针对http应用做一些分流的策略，比如针对域名、目录结构，它的正则规则比HAProxy更为强大和灵活，这也是许多朋友喜欢它的原因之一；
2. Nginx对网络的依赖非常小，理论上能ping通就就能进行负载功能，这个也是它的优势所在；
3. Nginx安装和配置比较简单，测试起来比较方便；
4. 也可以承担高的负载压力且稳定，一般能支撑超过几万次的并发量；
5. Nginx可以通过端口检测到服务器内部的故障，比如根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点，不过其中缺点就是不支持url来检测；
6. Nginx仅能支持http和Email，这样就在适用范围上面小很多，这个它的弱势；
7. Nginx不仅仅是一款优秀的负载均衡器/反向代理软件，它同时也是功能强大的Web应用服务器。LNMP现在也是非常流行的web架构，大有和以前最流行的LAMP架构分庭抗争之势，在高流量的环境中也有很好的效果。
8. Nginx现在作为Web反向加速缓存越来越成熟了，很多朋友都已在生产环境下投入生产了，而且反映效果不错，速度比传统的Squid服务器更快，有兴趣的朋友可以考虑用其作为反向代理加速器。

**HAProxy的特点是：
**

1. HAProxy是支持虚拟主机的，以前有朋友说这个不支持虚拟主机，我这里特此更正一下。
2. 能够补充Nginx的一些缺点比如Session的保持，Cookie的引导等工作
3. 支持url检测后端的服务器出问题的检测会有很好的帮助。
4. 它跟LVS一样，本身仅仅就只是一款负载均衡软件；单纯从效率上来讲HAProxy更会比Nginx有更出色的负载均衡速度，在并发处理上也是优于Nginx的。
5. HAProxy可以对Mysql读进行负载均衡，对后端的MySQL节点进行检测和负载均衡，不过在后端的MySQL slaves数量超过10台时性能不如LVS，所以我向大家推荐LVS+Keepalived。
6. HAProxy的算法现在也越来越多了，具体有如下8种：
      ① roundrobin，表示简单的轮询，这个不多说，这个是负载均衡基本都具备的；
      ② static-rr，表示根据权重，建议关注；
      ③ leastconn，表示最少连接者先处理，建议关注；
      ④ source，表示根据请求源IP，这个跟Nginx的IP_hash机制类似，我们用其作为解决session问题的一种方法，建议关注；
      ⑤ ri，表示根据请求的URI；
      ⑥ rl_param，表示根据请求的URl参数'balance url_param' requires an URL parameter name；
      ⑦ hdr(name)，表示根据HTTP请求头来锁定每一次HTTP请求；
      ⑧ rdp-cookie(name)，表示根据据cookie(name)来锁定并哈希每一次TCP请求。

 

###  

### Nginx和LVS作对比的结果

1、 Nginx工作在网络的7层，所以它可以针对http应用本身来做分流策略，比如针对域名、目录结构等，相比之下LVS并不具备这样的功能，所 以  Nginx单凭这点可利用的场合就远多于LVS了；但Nginx有用的这些功能使其可调整度要高于LVS，所以经常要去触碰触碰，由LVS的第2条优点  看，触碰多了，人为出问题的几率也就会大。
 2、Nginx对网络的依赖较小，理论上只要ping得通，网页访问正常，Nginx就能连得通，Nginx同时还能区分内外网，如果是同时拥有内外网的  节点，就相当于单机拥有了备份线路；LVS就比较依赖于网络环境，目前来看服务器在同一网段内并且LVS使用direct方式分流，效果较能得到保证。另 外注意，LVS需要向托管商至少申请多一个ip来做Visual  IP，貌似是不能用本身的IP来做VIP的。要做好LVS管理员，确实得跟进学习很多有关网络通信方面的知识，就不再是一个HTTP那么简单了。站长教学网 eduyo.com
 3、Nginx安装和配置比较简单，测试起来也很方便，因为它基本能把错误用日志打印出来。LVS的安装和配置、测试就要花比较长的时间了，因为同上所 述，LVS对网络依赖比较大，很多时候不能配置成功都是因为网络问题而不是配置问题，出了问题要解决也相应的会麻烦得多。
 4、Nginx也同样能承受很高负载且稳定，但负载度和稳定度差LVS还有几个等级：Nginx处理所有流量所以受限于机器IO和配置；本身的bug也还是难以避免的；Nginx没有现成的双机热备方案，所以跑在单机上还是风险较大，单机上的事情全都很难说。
 5、Nginx可以检测到服务器内部的故障，比如根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点。目前LVS中   ldirectd也能支持针对服务器内部的情况来监控，但LVS的原理使其不能重发请求。重发请求这点，譬如用户正在上传一个文件，而处理该上传的节点刚  好在上传过程中出现故障，Nginx会把上传切到另一台服务器重新处理，而LVS就直接断掉了，如果是上传一个很大的文件或者很重要的文件的话，用户可能 会因此而恼火。
 6、Nginx对请求的异步处理可以帮助节点服务器减轻负载，假如使用apache直接对外服务，那么出现很多的窄带链接时apache服务器将会占用大  量内存而不能释放，使用多一个Nginx做apache代理的话，这些窄带链接会被Nginx挡住，apache上就不会堆积过多的请求，这样就减少了相  当多的内存占用。这点使用squid也有相同的作用，即使squid本身配置为不缓存，对apache还是有很大帮助的。LVS没有这些功能，也就无法能 比较。
 7、Nginx能支持http和email（email的功能估计比较少人用），LVS所支持的应用在这点上会比Nginx更多。在使用上，一般最前端所 采取的策略应是LVS，也就是DNS的指向应为LVS均衡器，LVS的优点令它非常适合做这个任务。重要的ip地址，最好交由LVS托管，比如数据库的  ip、webservice服务器的ip等等，这些ip地址随着时间推移，使用面会越来越大，如果更换ip则故障会接踵而至。所以将这些重要ip交给  LVS托管是最为稳妥的，这样做的唯一缺点是需要的VIP数量会比较多。Nginx可作为LVS节点机器使用，一是可以利用Nginx的功能，二是可以利  用Nginx的性能。当然这一层面也可以直接使用squid，squid的功能方面就比Nginx弱不少了，性能上也有所逊色于Nginx。Nginx也 可作为中层代理使用，这一层面Nginx基本上无对手，唯一可以撼动Nginx的就只有lighttpd了，不过lighttpd目前还没有能做到  Nginx完全的功能，配置也不那么清晰易读。另外，中层代理的IP也是重要的，所以中层代理也拥有一个VIP和LVS是最完美的方案了。具体的应用还得  具体分析，如果是比较小的网站（日PV<1000万），用Nginx就完全可以了，如果机器也不少，可以用DNS轮询，LVS所耗费的机器还是比较 多的；大型网站或者重要的服务，机器不发愁的时候，要多多考虑利用LVS

​	






**4.2、使用terracotta或者memcached使session共享**
 

 **4.2.1、terracotta是jvm级别的session共享**

 它基本原理是对于集群间共享的数据，当在一个节点发生变化的时候，Terracotta只把变化的部分发送给Terracotta服务器，然后由服务器把它转发给真正需要这个数据的节点，并且共享的数据对象不需要序列化。

 

**4.2.2、通过memcached实现内存级session共享**

通过memcached-session-manager（msm）插件，通过tomcat上 一定的配置，即可实现把session存储到memcached服务器上。注意：tomcat支持tomcat6+，并且memcached可以支持分布 式内存，msm同时支持黏性session（sticky sessions）或者非黏性session（non-sticky  sessions）两种模式，在memcached内存中共享的对象需要序列化。结构如图3：



![img](http://dl2.iteye.com/upload/p_w_upload/0096/4664/ea0fdbe1-df9d-3014-9012-1c674bfdfc5c.jpg)
 

 通过一定的配置，可以实现故障转移(只支持对非粘性session)。如：

Xml代码  ![收藏代码](http://aokunsang.iteye.com/p_w_picpaths/icon_star.png)

1. <Context>  
2.    ...  
3.    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"  
4. ​    memcachedNodes="n1:host1.yourdomain.com:11211,n2:host2.yourdomain.com:11211"  
5. ​    failoverNodes="n1"  
6. ​    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"  
7. ​    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"  
8. ​    />  
9. </Context> 

 说明：failoverNodes：故障转移节点，对非粘性session不可用。属性 failoverNodes="n1"的作用是告诉msm最好是把session保存在memcached  "n2"节点上，只有在n2节点不可用的情况下才把session保存在n1节点。这样即使host2上的tomcat宕机，仍然可以通过host1上的 tomcat访问存放在memcached "n1" 节点中的session。
 
**4.2.3、其他方案**
通过cookie保存用户信息(一般是登录信息)，每一个请求到达web应用的时候，web应用从cookie中取出数据进行处理（这里尽量对cookie做加密处理）；
另外一种是把用户信息的关键属性保存到数据库，这样就不需要session了。请求过来从数据库查询关键属性数据，做相应处理。***缺点：加大了数据库的负载，使数据库成为集群的瓶颈\***