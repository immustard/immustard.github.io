# 湖仓一体


&lt;!--more--&gt;



## 数据仓库与数据湖的区别

说到湖仓一体, 就要先了解一下数据仓库和数据湖的区别是什么. 

下面这个表格就是AWS([传送门](https://aws.amazon.com/cn/big-data/datalakes-and-analytics/what-is-a-data-lake/))上的对比: 

| 特性         | 数据仓库                                         | 数据湖                                                       |
| ------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| **数据**     | 来自事务系统、运营数据和业务线应用程序的关系数据 | 来自IoT设备、网站、移动应用程序、社交媒体和企业应用程序的非关系和关系数据 |
| **Schema**   | 设计数据仓库实施之前(写入型Schema)               | 写入在分析时(读取型Schema)                                   |
| **性价比**   | 更快查询结果会带来较高存储成本                   | 更快查询结果只需较低存储成本                                 |
| **数据质量** | 可作为重要事实依据的高度监管数据                 | 任何可以或无法进行监管的数据(例如原始数据)                   |
| **用户**     | 业务分析师                                       | 数据科学家、数据开发人员和业务分析师(使用监管数据)           |
| **分析**     | 批处理报告、BI和可视化                           | 机器学习、预测分析、数据发现和分析                           |



从上面这个表就能看出来数据仓库和数据湖的差别还是很明显的. 在企业中, 两者的作用是互补的, 所以**不应该认为数据湖的出现是为了取代数据仓库, 毕竟两者的作用是截然不同的**. 



## 湖仓一体是怎么诞生的

对于数据而言, 数据仓库就像是一个大型图书馆, 里面的数据需要按照规范放好, 可以按照类别找到想要的信息. 而数据湖就像是一个大型仓库, 可以存储任何形式(结构化, 半结构化和非结构化)和任何格式(文本, 图像, 音频, 视频)的原始数据. 



在产品的角度上来说, 数据仓库一般是独立标准化产品, 数据湖更像是一种架构指导, 需要配合着系列周边工具来实现业务需要. 也就是说, 数据湖的灵活性对于前期开发和前期部署是友好的; 数据仓库的规范性对于大数据后期的运行和长期发展是友好的. 那有没有一种新架构能兼具数据仓库和数据湖的优点? 



然后, 湖仓一体就诞生了. 



依据DataBricks公司对Lakehouse 的定义, 湖仓一体是一种结合了数据湖和数据仓库优势的新范式, 在用于数据湖的低成本存储上, 实现与数据仓库中类似的数据结构和数据管理功能. 湖仓一体是一种更开放的新型架构, 有人把它做了一个比喻, 就类似于在湖边搭建了很多小房子, 有的负责数据分析, 有的运转机器学习, 有的来检索音视频等, 至于那些数据源流, 都可以从数据湖里轻松获取. 



&gt;  需要注意的是: 
&gt;
&gt; 数据湖 &#43; 数据仓库 ≠ 湖仓一体



湖仓一体诞生的目的, 总结起来就是:

1. 打通数据的存储与计算
2. 灵活性与成长性兼得



&lt;center&gt;
    &lt;img style=&#34;border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34; 
    src=&#34;https://miro.medium.com/max/1280/1*Nv28a3S6zQFAHZd4iVy8Xw.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;
    &lt;br&gt;
    &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;&#34;&gt;
      灵活性与成长性
  	&lt;/div&gt;
&lt;/center&gt;



## 湖仓一体的好处

* **数据重复性**：如果一个组织同时维护了一个数据湖和多个数据仓库，这无疑会带来数据冗余。在最好的情况下，这仅仅只会带来数据处理的不高效，但是在最差的情况下，它会导致数据不一致的情况出现。湖仓一体的结合，能够去除数据的重复性，真正做到了唯一。

* **高存储成本**：数据仓库和数据湖都是为了降低数据存储的成本。数据仓库往往是通过降低冗余，以及整合异构的数据源来做到降低成本。而数据湖则往往使用大数据文件系统和Spark在廉价的硬件上存储计算数据。湖仓一体架构的目标就是结合这些技术来最大力度降低成本。

* **报表和分析应用之间的差异**：数据科学倾向于与数据湖打交道，使用各种分析技术来处理未经加工的数据。而报表分析师们则倾向于使用整合后的数据，比如数据仓库或是数据集市。而在一个组织内，往往这两个团队之间没有太多的交集，但实际上他们之间的工作又有一定的重复和矛盾。而当使用湖仓一体架构后，两个团队可以在同一数据架构上进行工作，避免不必要的重复。

* **数据停滞**：在数据湖中，数据停滞是一个最为严重的问题，如果数据一直无人治理，那将很快变为数据沼泽。我们往往轻易的将数据丢入湖中，但缺乏有效的治理，长此以往，数据的时效性变得越来越难追溯。湖仓一体的引入，对于海量数据进行治理，能够更有效地帮助提升分析数据的时效性。

* **潜在不兼容性带来的风险**：数据分析仍是一门兴起的技术，新的工具和技术每年仍在不停地出现中。一些技术可能只和数据湖兼容，而另一些则又可能只和数据仓库兼容。湖仓一体的架构意味着为两方面做准备。


---

> 作者:   
> URL: https://buli-home.cn/data_lakehouse/  
