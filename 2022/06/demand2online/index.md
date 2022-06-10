# 从需求到上线全流程


## 从需求到上线全流程 | 青训笔记

这是我参与「第三届青训营 -后端场」笔记创作活动的的第4篇笔记。   

本篇主要内容是后端开发日常的整体流程，对从需求到上线全流程课程的一个总结。从why、what、how三个方面讲述了整个后端的开发流程，为什么需要流程，有哪些流程，怎样执行流程等等。  

### WHY | 为什么要有流程  
**团队规模和流程的关系**  
从想法到产品，如果是个人开发者，就不需要流程，自己按着思路做就好了。但如果是一个团队就需要进行协作，也就需要流程，而团队规模越大，流程中也会有新的问题出现。  
如果复杂的项目没有流程，那每个开发者都有自己的想法，都有自己的开发流程，如何协作交流共同完成一个项目就变得很重要。  
对于每个阶段：  
- 需求阶段：团队需要决策，从众多想法中确定需求
- 开发阶段：团队需要协作，各自安排，相互配合
- 测试阶段：团队商量如何交付，测试如何展开，bug如何修复
- 发布阶段：团队需要确保版本和流量如何控制，整个过程需要规范
- 运维阶段：对于线上问题，如何应急响应，及时处理用户反馈

**瀑布模型**  
需求--开发--测试--发布--运维，整个流程排成一条线，前一个阶段完成才进入下一个阶段。  
工作流程直观，是标准的研发阶段。流程十分严格，适用于银行，支付等公司，对待每个过程都十分严格要求，最大可能避免出错，因为一旦出现问题会造成很严重的影响。  
但整个流程，是十分低效的，每个步骤都是固定的，没有灵活性。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101919366.png)  
  
**敏捷开发**  
以“需求--开发--测试--发布--运维”进行快速迭代，是更现代的的流程模型。  
不断将整个流程迭代，能够以小团队进行快速迭代，团队成员的合作也更加紧密，能够以人为本，和用户进行沟通，增减需求等。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101920586.png)  

**实际的例子**   
SAFe：企业实施敏捷开发的一套方法论，是多个敏捷团队之间的配合。  
Scrum：  
- 敏捷教练：Scrum Master
- 产品负责人：Product Owner
- 敏捷团队：Scrum Team
- 敏捷发布火车：Agile Release Train

如果将scrum类比为一个战术小队，那敏捷教练就是队长，产品负责人就是负责联络指挥部和发布任务的人，其他的成员就是特种兵。大家不是按照一个方阵前进，而是用更敏捷的方式前进，也就是敏捷发布火车。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101922405.png)  

### WHAT | 有哪些流程
**需求阶段**  
不要浪费时间讨论不应该存在的问题。  
对于产品经理的需求，不需要的需求该砍就砍，同时也要站在用户的角度评估需求。  
需求评估方法：  
- MVP法  
  以MVP（最小化可行产品）思想进行需求评估，能够收集用户反馈，快速迭代。  
  可以先给用户一个简单的能用的产品，比如用户想要代步工具，我们可以先给个滑板车，然后根据用户的反馈逐步进行功能升级，自行车，小汽车等等，最终变成用户想要的产品。  
- 四象限法  
  如果有多重任务，按照四象限法进行分类，先判断事情的重要性，再判断紧急程度，一个高效的占比是大多数时间在处理重要但不紧急的事情，因为紧急的事情容易让我们犯错误。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101601342.png)  

**开发阶段**  
云原生改变了后端开发的工作，如果把虚拟主机的开发类比是用冷兵器决斗，那云原生就是让我们可以使用热武器了。  
云原生下的开发：  
传统虚拟机：  
- 在物理主机中虚拟多个虚拟机，每个虚拟机拥有自己的操作系统；
- 运维人员负责维护和交付虚拟机；
- 每个虚拟机都要安装相应的依赖环境。  

容器化：  
- 容器是在操作系统中虚拟出来的；
- 相互隔离，低开销；  
通过cgroup, namespace和Union Mount等技术实现了容器之间的相互隔离，同时容器的开销很低。  
- 应用和依赖作为一个整体，打包成镜像交付。  
不再依赖运维人员创建和维护运行时环境。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101613153.png)  

单体架构：  
- 多个模块共同组成一个服务，服务体量较大；
- 模块之间直接调用，不需要RPC通信；
- 服务整体扩缩容量；
- 多人开发一个代码仓库，需要充分集成测试。  

微服务架构：  
- 各功能在不同的服务中；
- 不同模块需要进行RPC通信；
- 不同模块可以独立扩缩容；
- 每个服务的代码仓库仅由少部分人维护。  

开发环境逐渐云原生化，FaaS，PaaS等技术让开发逐渐从本地IDE向线上转变，而入职搭建开发环境的情况也可以通过Web IDE等技术，在未来达到开箱即用。  

团队的分支策略，协作存在的问题：  
- 多个团队成员之间各自用什么分支开发
- 修改之间有冲突如何解决
- 出了问题的代码如何回退到之前的版本

比如可以专门用一个release分支，大家将代码合并到release分支，然后测试，发布，再把release合并到master；或者直接把开发的分支合入master，再用master上的commit进行发布等等。  

代码规范、自测和文档：  
代码规范：  
- 养成良好的注释习惯
- 不要有魔法数字，魔法字符串
- 重复的逻辑抽象成公共的方法，不要copy代码
- 正确使用IDE的重构功能，防止修改错误

自测：  
- 单元测试
- 功能环境测试
- 测试数据构造

文档：
- 大型改造需要有技术设计文档，方案评审
- 好的接口文档能更方便和前端进行沟通  

**测试阶段**  
越底层的测试粒度越细，需要越多的数量去覆盖所有场景，越顶层的测试越能用少量的case覆盖大多数场景，而且越早发现的缺陷解决成本越低。  
测试金字塔：  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101633451.png)  

实际工作中，三套测试环境：  
功能环境：  
- 需要一个能够模拟线上的环境进行开发和测试；
- 环境之间能够相互隔离，不影响其他功能的开发和测试。  

集成环境：  
- 不同人开发的功能合并在一起测试，相互之间的影响可能产生缺陷；
- 迭代发布的所有功能合并在一起测试，确保发布的所有功能之间的影响不产生缺陷。  

回归环境：  
- 确保新的功能不对老的功能产生影响；
- 回归测试一般会借助自动化测试脚本。  

**发布阶段**  
如果发布过程中，页面突然打不开，使用chrome的开发者工具，发现一个接口连接失败；使用curl命令请求接口，发现域名解析到127.0.0.1，定位到问题所在，一个请求就是一个链条，可以顺着链条一节一节去定位推断问题出在哪。  

发布过程中要做的事：  
- 发布负责人：通知相关人员，观察各个服务的发布状态；
- 变更服务的相关RD：按照规范检查日志，监控，响应线上的告警；
- 值班人员：关注用户反馈，关注发布过程中的监控和告警，如果有异常需要立刻判断是否由变更引起。  

五种发布模式：  
蛮力发布：  
- 简单粗暴，直接用新版本覆盖老版本；  
- 成本低；
- 发布过程中服务中断；
- 如果出问题会影响全部用户；
- 适用于测试环境部署，小公司或非核心的业务服务。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101932309.png)  

金丝雀发布：  
- 逐步发布，先小部分，再整体；
- 简单，能够用少量用户验证新版本功能；
- 发布过程中服务会中断；
- 发现不了随用户量增大才会暴露的问题；
- 适用于测试环境部署，小公司或非核心的业务服务。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101932187.png)  

滚动发布：  
- 每个实例都是金丝雀发布，逐步放大流量，对用户影响小，体验平滑；  
- 发布过程中用户体验不好中断；
- 可以充分验证服务功能；
- 流程比较复杂，对发布系统有比较高的要求；
- 发布速度慢；
- 新老版本不兼容的情况不能使用；
- 适用于发布系统能力比较强，可以平滑切换流量；
- 适用于发布自动化程度高，可以自动滚动。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101934462.png)  

蓝绿发布：  
- 服务分蓝绿两组，先把蓝组流量摘掉然后升级，只用绿组提供服务，之后切换全部流量，只用蓝组提供服务，然后升级绿组服务，最终两组全部升级；  
- 发布速度快，流程简单；
- 需要一半机器承担所有流量的能力；
- 出问题会影响全部用户；
- 适用于服务器资源丰富，新老版本不能兼容的情况，需要一次性升级到新版本。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101934367.png)  

红黑发布：  
- 与蓝绿发布类似，但发布会动态扩容出一组新的服务，不需要常备两组服务；
- 发布速度快，流程简单；
- 对机器数量有要求，需要扩容一倍；
- 出问题会影响全部用户；
- 适用于服务器资源丰富，新老版本不能兼容的情况，需要一次性升级到新版本。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101935742.png)  

没有强大发布系统和服务器资源不足的公司是一般使用蛮力发布或金丝雀发布；  
有强大发布系统和服务器资源充足的公司是一般使用滚动发布和蓝绿发布。  

**运维阶段**  
当故障发生以后，首先是进行止损，尽快让服务恢复功能；其次是让服务的上下游感知到问题；当上述两个动作做了以后才去定位和修复问题。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101830147.png)  

对于复杂的微服务体系，往往会在其中添加埋点采集Metrics、Logging、分布式Trace等多种数据，进行监控。  

总结：  
- 需求阶段：确定做什么不该做什么
- 开发阶段：按照规范实现产品
- 测试阶段：验证产品，修改缺陷
- 发布阶段：按照流程规范上线
- 运维阶段：观察线上监控和日志

### HOW | 怎样执行流程
**怎样让生活更美好**  
在重视质量的团队，效率往往比较低；在重视效率的团队，事故往往比较多，两者如何平衡，这是一个重要问题。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101837079.png)  
技术的发展会带来质量和效率的同时提高，将质量保障融入到流程，将流程自动化，从需求到上线全流程自动化，同时提高质量和效率。  

**DevOps**  
以上就是DevOps理念，从需求开始，写代码，编译，测试，发布，运维再到监控形成一个闭环。这样我们可以持续集成，持续交付，两者密不可分。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101842085.png)  

DevOps解决方案：  
- 代码管理
- 自动化测试
- 持续集成
- 持续交付

那在DevOps的基础上如何优化？可以通过效率竖井的概念进行优化。  
从需求到交付的过程中，产生价值的开发，测试等的动作占比是很低的，大量的时间在等待和传递，比如测试在等待开发把环境部署好，人和人的沟通也比较慢等等。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206101848365.png)  

**全流程自动化**  
- 通过效能平台串联各个阶段  
  需求发起研发流程的自动化；  
  写代码，测试环境部署的自动化；  
  自动化测试触发和报告分析；  
  发布过程可观测融入流程。  

- 减少无价值的等待  
  分析整个流程的耗时，计算真正产生价值的时间；  
  不断优化流程，让有价值的流程时间占比上升。  

### 总结  
本篇blog从why、what和how三个维度分析了需求到上线的整个流程。比如学习了敏捷开发和SAFe的方法论对于团队流程的开发有了新的认识；了解了研发的五大阶段：需求-开发-测试-发布-运维；知道了DevOps、效率竖井和全流程自动化等流程优化理念，对于整个后端的开发工作也进行了熟悉和了解。  
