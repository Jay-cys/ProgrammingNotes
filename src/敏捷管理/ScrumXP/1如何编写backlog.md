
# 1 什么是Scrum

Nokia 用了几年时间，对上百个 Scrum 团队的工作进行了回顾，总结出了**迭代开发的基本需求**：

- 迭代要有固定时长（被称为“时间盒——timebox”），不能超过六个星期。
- 在每一次迭代的结尾，代码都必须经过 QA 的测试，能够正常工作。


Nokia的**Scrum标准**：

- Scrum 团队必须要有产品负责人，而且团队都清楚这个人是谁。
- 产品负责人必须要有产品 Backlog，其中包括团队对它进行的估算。
- 团队必须要有燃尽图，而且要了解他们自己的生产率。
- 在一个 Sprint 中，外人不能干涉团队的工作。


Scrum 和 XP 都关注如何把事情做好。这些团队承认在开发过程中会犯错，但是他们明白：**要投入实践中，动手去构建产品，这才是找出错误的最好方式；不要只是停留在理论层次上对软件进行分析和设计**。

- [http://agilemanifesto.org/](http://agilemanifesto.org/)
- [http://www.mountaingoatsoftware.com/scrum](http://www.mountaingoatsoftware.com/scrum)
- [http://www.xprogramming.com/xpmag/whatisxp.htm](http://www.xprogramming.com/xpmag/whatisxp.htm)




# 2 什么是Backlog

产品backlog是 Scrum 的核心，是一个**需求、user story或特性**等组成的列表，按照重要性的级别进行了排序。它里面包含的是客户想要的东西，并用客户的术语加以描述。它可以叫**User Story**或**Backlog条目**。

## 2.1 User Story的结构
主要由以下部分组成：

- ID：统一标识符，唯一标识一个user story
- Name（名称）：简短的、描述性的user story名。比如“查看你自己的交易明细”。它必须要含义明确，这样开发人员和产品负责人才能大致明白我们说的是什么东西，跟其他user story区分开。
- Importance（重要性）：产品负责人评出一个数值，指示这个user story有多重要。例如 10 或 150。分数越高越重要。
- Initial estimate（初始估算）：团队的初步估算，表示与其他user story相比，完成该user story所需的工作量。最小的单位是user story点（story point），一般大致相当于一个“理想的人天（man-day）”。
   - **不需要保证这个估值绝对无误，而是要保证相对的正确性**（即，两个点的user story所花费的时间应该是四个点的user story所需的一半）
> 补充：在Nokia，一个user story point代表半周事件，更具体的拆分用task标识（可以以天或小时为单位）

- How to demo（如何做演示）：它大略描述了这个故事应该如何在 sprint结束demo meeting上进行示范，本质就是一个简单的测试规范。“先这样做，然后那样做，就应该得到……的结果 ”。
   - 如果你在使用 TDD（测试驱动开发），那么这段描述就可以作为验收测试的伪码表示。
- Notes（注解） ——相关信息、解释说明和对其它资料的引用等等。



> 补充：这里我附加了Nokia的一份Backlog checlist的模板，里面是针对每个user story需要填充的字段。




## 2.2 停留在功能上，而不是技术细节
> 补充：User Story应该停留在用户功能上，完整描述一个端到端的用户使用场景，而不是堆砌一些技术代码细节可设计方案。要从用户角度要求功能，而不是开发角度。


只要发现面向技术的故事，我一般都会问产品负责人 “但是为什么呢”这样的问题，一直问下去，直到我们发现内在的目标为止。然后再用真正的目标来改写这个故事（“提高在后台系统中搜索并生成表单的响应速度”）。最开始的技术描述只会作为一个注解存在（“为事件表添加索引可能会解决这个问题”）。


## 2.3 A good example of User Story
| release | ID | name | description | state |
| --- | --- | --- | --- | --- |
| release 2 | ID-055 | 'Operator wants to
  have a CLI command to control the phase of activating UUS3 service | 'As a operator, I
  wants to use below CLI command to control the phase of activating service:CLI command 1:---- configure---- voice---- sip---- [no]
  allitf-isdn-cas-term---- if-index---- [no]
  user-info-profileCLI command 2:---- configure---- voice---- sip---- [no]
  user-info-prof---- prof-name---- [no]
  version-nbr---- [no]
  service-type---- [no]
  active-establish-call---- [no]
  active-stable-call1. I can use
  command 1 to assign a pre-created active-rule to the specific term2. I can get error
  notification with below two types of configurations:active-establish-call
  is true and active-stable-call true,active-establish-call
  is false and active-stable-call false,error info should
  be:"invalid configuration rule(need update)"3. I can configure
  the service-type to indicating the type of UUS service, now, I only need UUS3
  service.4. I can use below
  CLI command to show the detail configuration info which is sorted by profile
  index instead of profile name:CLI command:show voice sip
  user-info-prof [prof-name]show voice sip
  profilesinfo configure
  voice sip user-info-profinfo configure
  voice sip user-info-prof [prof-name]Note: if the value
  of service-type in DB is not UUS3, system shall assign UUS3 to service-type,
  this solution is designed for avoiding below issue:if one extend the
  range of service-type in newer release,such as Release 5, and does rollback
  from Release 5 to Release 3. one shall promise the extended service does not work
  in Release 3, and can configure to UUS3 again.FT test list:SIPSRV_CLI_01
  ~ SIPSRV_CLI_11Auto Test case :SIPSRV_CLI_01
  ~ SIPSRV_CLI_11 | WellAnalyzed |


