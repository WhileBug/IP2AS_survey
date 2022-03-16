## IP2AS文章总结

### 2003-SIGCOMM-Towards an Accurate AS-Level Traceroute Tool

方法类型：pair-matching



### 2004-INFOCOM-Scalable and Accurate Identification of AS-Level Forwarding Paths

方法类型：pair-matching

对前面SIGCOMM的文章进行了优化，将IP2AS映射问题转化为了path-pairs的一系列优化问题，使用动态规划和迭代更新提升准确度

### 2009-CoNEXT-Where the Sidewalk Ends: Extending the Internet AS Graph Using Traceroutes From P2P Users

提出了一系列启发式算法，用来从traceroute数据钟找到missing的AS级别的链路

### 2010-PAM-Toward Topology Dualism: Improving the Accuracy of AS Annotations for Routers

方法类型：IP-router-AS

使用数据集：CAIDA Ark（收集traceroute）

Routeviews提供BGP数据集

MIDAR：CAIDA的别名解析工具

kapar：别名解析工具

首先使用别名解析将不同的interface解析到同一个路由器上，得到路由器级别拓扑，然后利用五步启发式算法推断AS级别拓扑：

![image-20220228213849177](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20220228213849177.png)

1. **single**：如果一个路由器的所有接口 IP 地址都属于一个 AS，那么这个路由器就属于这个 AS
2. **election**：如果一个路由器绝大多数的接口 IP 地址都属于一个 AS，那么这个路由器就属于这个 AS
3. **neighbor**：考虑邻居路由器的归属 AS 情况，如果一个路由器绝大多数的邻居路由器都属于一个AS，那么这个路由器就属于这个 AS
4. **customer**：如果邻居路由器所属的 AS 之间为 provider to customer 的商业关系，那么就推测路由器属于 customer 的 AS，因为通常 provider 会对外宣告他的部分 customer 的 IP 地址空间
5. **degree**：考虑邻居路由器所属的 AS 的度数，度数较小的 AS 通常为 customer，因此路由器属于度数最小的邻居路由器所属的 AS。

### 2010-PAM-Quantifying the Pitfalls of Traceroute in AS Connectivity Inference

方法类型：pair-matching

### 2011-JSAC-A Framework to Quantify the Pitfalls of Using Traceroute in AS-Level Topology Measurement

方法类型：pair-matching

整个框架由以下四个步骤构成：

<img src="C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023204930573.png" alt="image-20211023204930573" style="zoom:50%;" />

1.**拓扑图构建：**收集从同一个AS到同一个目的地的数百万对traceroute路径和BGP路径

2.**定位不匹配片段（核心工作）：**定位不匹配的路径对中的不匹配的片段

3.**定位一对一AS替换：**将每个不匹配的片段转换为多个一对一的AS替换

4.**虚假链路关联：**将拓扑上的每个虚假链路与创建虚假链接的替代品相关联

**定位不匹配片段（核心工作）**的具体方法：

作者首先将AS路径转换为字符串，基于traceroute推断出的AS路径将尽可能与BGP推断出的AS路径吻合的猜想，将定位不匹配字符串的问题转换为了一个**LCS（longest common subsequence）**问题，并提出了系统性分析的5个步骤：

<img src="C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023211144708.png" alt="image-20211023211144708" style="zoom:50%;" />

- **Special tokens：**两个字符串匹配的结果中出现了特殊字符就必定存在路径不匹配
- **Tie-break：**存在多种替代方式时，使用在BGP路径中=符号出现得尽量比较早的那种
- **Loop：**不匹配的片段是内部循环
- **Missing tail：**AS路径尾部丢失
- **Omission：**丢弃出现的额外路径（出现+的）

最后对不匹配片段出现的原因进行了分析

### 2015-CoNEXT-Mapping Peering Interconnections to a facility

#### 1-Preparation of traceroute data

首先以多种不同的大型网络为target，在多个vantage point上收集traceroute数据

然后使用Team Cymru的IP2AS服务，基于longest prefix matching做IP2AS，并且使用别名解析（MIDAR System）技术检测IP2AS中的错误，规避掉

#### 2-Constrained Facility Search

这一步主要是借用RTT-based geolocation的思想，不过使用facility information data代替delay data

1. **识别public peering关系和private peering关系**

   - 当在traceroute中发现(IPa, IPe, IPb)这种IP序列的时候，进行分类判断：
     - 当IPe是某个IXP的IP地址的时候，标记（A，B）是public Peering关系
     - 反之，丢弃掉这个序列

   - 当观察到（IPa，IPb）这种序列的时候，标记（A，B）是private Peering关系（可以是cross-connect, tethering, remote)

2. **初始化facility search**

   分成public peering和private peering进行讨论

   - public peering：对于IP序列(IPa, IPe, IPb)，将AS A对应的facility集合{Fa}与IXP对应的facility集合{Fe}求交集

     - Resolved Interface：当这个交集中只存在一个facility的时候，就将IPa对应到这个facility

     - Unresolved local interface：当这个交集中有多个facility的时候，说明IPa对应到facility集合中的某个facility

     - No common：当这个交集为空的时候，有两种可能：（a）Unresolved remote interface：AS A通过remote peering reseller远程连接到IXP（b）Missing data：关于AS A和IXP的facility信息不全

       使用T4P的方法识别remote peering

   - private peering：对应IP序列(IPa, IPb)，我们的操作和上面的类似，不过用来求交集的集合变成Fa和Fb

3. **通过别名解析限制facility集合大小**

   实际上同一个接口的不同别名会属于同一个facility，于是就可以对不同别名的facility求交集以减少facility集合的大小

4. **使用targeted traceroute再限制facility集合**

   对于仍然unresolved的interface选取一系列target，需要满足这些target对应的facility是unresolved的interface的facility集合的子集，然后对这些target进行traceroute，然后重复上述步骤

#### 3-Facility search in the reverse direction

实际在上面的search过程中，存在这样的问题：

在remote peering，public peering at IXP中，可能存在多个facility，但是在我们上面的search算法中，由于traceroute只显示入口的router，不显示出口的router

因此我们再从相反的方向进行traceroute

#### 4-Proximity Heuristic

很多情况下，我们并没有办法获取反方向的traceroute path，因此我们引入基于最近原则的启发式，对于一个public peering Link (IPa,IPixp,b,IPb)，如果IPb有多个对应的facility，那么就选择距离IPa的facility最近的一个facility作为这个link的IPb的facility

进一步的，由于交换机拓扑并不是都是有的，因此针对没有详细交换机拓扑的情况，我们引入基于概率排名的IP-to-facility mapping：针对我们多个出口的情况，计算通往不同出口的频率作为概率

### 2016-IMC-bdrmap: Inference of Borders Between IP Networks

方法类型：IP-router-AS

使用数据：

Routerviews（RV）：公共BGP数据，用于推断/24的IP与AS之间的关系

RIPE RIS：公共BGP数据，用于推断/24的IP与AS之间的关系

RIR delegation数据：用于补充BGP数据中，部分AS没有broadcast的IP与AS之间的映射关系

PeeringDB：记录IXP的地址前缀

Packet Clearing House（PCH）：记录BGProuter的地址前缀

准确识别网络边界对于当前学术、工业界的很多问题都十分重要，但是这样的互联网路由级拓扑发现和推理是十分容易出错，主要由于以下几个原因：

- TCP/IP 体系结构没有提供检测域间边界的机制
- 未能准确解析同一路由器的此类别名将导致网络之间链路数量的夸大推断
- 运营商地址分配和路由实施实践限制了准确性
- Traceroute 反复采样部分处于中心位置的链接但忽略更远的链接
- 运营商担心向竞争对手透露他们的拓扑

因此，这篇文章开发了一种方法来准确判断网络边界，具体的贡献如下所示：

- 引入一种可扩展的方法来准确**推断给定网络的边界**，以及附加在每个边界上的其他网络
- 开发一个有效的系统，允许在资源有限的设备上部署他们的方法
- 使用来自四家网络运营商的真实情况以及 IXP 地址使用数据库来验证其算法的正确性
- 通过分析大型接入 ISP 的拓扑结构以了解现代互连协议，展示算法的效用
- 公开发布源码实现

分析框架如下：

![image-20211023212432133](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023212432133.png)

主要有三步，其中第三步是核心：

- 收集用于通知数据收集和分析的路由和寻址数据
- 部署一个高效的 traceroute，以跟踪从每个 VP 到全局 BGP 路由系统中观察到的每个路由前缀的路径，应用别名解析技术来推断用于域间互连的路由器和点对点链路，构建整个网络初步的拓扑结构
- 使用收集到的数据来组合约束，使用启发式算法以推断路由器所有权

第二步的主要步骤：

- 生成要探测的地址块列表
- 收集traceroute
- 解析路由地址的IP别名
- 推断点对点链接
- 限制虚假别名
- 构建路由器级别拓扑图

第三步的主要方法：

![image-20211023213450980](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023213450980.png)

最后对推理结果进行了验证，并对比了分别使用traceroute和BGP的推理结果

### 2016-IMC-MAP-IT: Multipass Accurate Passive Inferences from Traceroute

方法类型：IP-router-AS

这篇文章主要着眼于如何可靠的识别AS之间的链路的地址。

准确识别AS之间的链路地址主要有两个困难：

- 在两个AS之间的链路可能被分配来自同一个地址空间的IP地址
- Traceroute artifacts可能导致错误

这篇文章的主要贡献如下：

- 描述了一种新颖、稳健且高度精确的多通道算法MAP-IT，用于从 traceroute 推断用于 AS 间链路的接口（核心）
- 使用来自二级区域网络提供商 Internet2 的实际情况验证算法，达到 100% 的精确度和 96.9% 的召回率
- 使用从 ISP 级别 3 和 TeliaSonera 中的接口的 DNS 主机名得出的近似基本事实来确认这些结果
- 将 MAP-IT 与用于识别 AS 间链接接口的现有方法进行比较，展示出更高的准确性

MAP-IT算法更多地关注对算法效果的提升而不是使用更多的监测设施提供效果上的提升

- Discard and Sanitize Traces

  - 路由器的配置不当、负载均衡以及瞬时路由更改可能会导致traceroute伪影
  - 2步对污染进行缓解:
    - 删除由转发 TTL=1 数据包的有问题的路由器引起的虚假邻接
    - 确定在跟踪期间是否发生负载平衡或瞬时路由更改

- Determine Interface Other sides

  ![image-20211023215717170](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023215717170.png)

  - 点对点链接可以从 /30 或 /31 前缀分配地址
  - 使用启发式推断每个接口的另一侧
  - 提取任何Traceroute中看到的所有地址，包括丢弃的Traceroute
  - /30 前缀中的所有非主机地址都被分配到 /31 前缀的另一侧
  - 对于剩余的有效主机地址，检查数据集中是否出现了不同的地址，该地址将是其 /30 前缀中的保留地址。 如果是这样，将其分配给 /31 前缀的另一侧，否则假设它来自 /30 前缀。

- Extract Neighbors Sets

  - 使用清理后的Traceroute，创建第 3 节中描述的 NF 和 NB。接口的 Ns 包括所有跟踪中恰好在它之前 (NB) 或之后 (NF) 一跳看到的所有地址，不包括空跳和私有/共享地址 . 不在 Ns 中包含私有/共享地址，因为它们不是全局可路由的或唯一的，并且不应出现在跟踪路由中。
  - 在数据集中与至少一个其他地址相邻的 4,752,201 个接口地址中，449,602 个具有多个地址的 NF，1,139,087 个具有多个地址的 NB。 对于在第 4.4.1 节中对特定接口进行的直接推断，其 NF 或 NB 必须至少包含 2 个地址。

- Adding Inferences

  - 一个接口被确定用于跨域链接分4个步骤：

    •使用 Ns 和当前的 IP2AS 映射对 IH 进行直接推断（第 4.4.1 节）；

    •更新每个直接推理另一侧的映射（第 4.4.2 节）；

    •解决对同一接口向前和向后进行推断的矛盾（第 4.4.3 节）；

    •解决对相邻 IH 进行的逆推理6，保留一个推理并丢弃另一个（第 4.4.4 节）。

- Remove Inferences

  ![image-20211023220533971](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023220533971.png)

  - 当访问每个具有直接推理的 IH 时，该算法会根据当前的 IP2AS 映射检查连接的 AS 是否仍占其 N 的一半以上。
  - 如果不是，那么最初将推理从直接推理更改为间接推理，但保留其 IP2AS 映射。
  - 每次通过 IH 后，所有没有关联直接推理的间接推理以及它们的 IP2AS 更新都将被丢弃

- Overall Convergence

  ![image-20211023220558250](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20211023220558250.png)

  - 由于存在不确定推理，整个算法重复添加和删除步骤，可能永远不会达到不能添加推理和不能删除推理的地步。
  - 相反，它会收敛到不断添加和删除相同推论的点。
  - 因此，在移除步骤结束时寻找一个重复的状态作为停止标准，这表明不能做出更可靠的推断。
  - 在实验中，这发生在主 while 循环的 3 次迭代之后

- Traceroute Artifacts

  - 如果邻居用于 AS 间链路，则应该可以进行反向推断，因为提供商通常会从许多入口点为其客户接收数据包，这可能会暴露边界路由器上的多个接口
  - 如果隐藏 AS 中只有一跳，则第一个 AS 和末节 AS 之间可能存在隐藏 AS。 这会导致断开连接的 Ases 之间的推断无效
  - 第三方地址不会导致此步骤的错误推断。 存根 AS 返回的第三方地址将是其提供者之一

- Stub AS Heuristic

  - Traceroute伪影：
    - 错误
    - 输出接口
    - 临时路由更改
    - 每包负载均衡
  - 可能导致错误或阻止进行推断。 即使是旨在避免大多数类型的负载平衡的巴黎跟踪路由，也不能免受工件的影响

### 2018-IMC-Pushing the Boundaries with bdrmapIT: Mapping Router Ownership at Internet Scale

方法类型：IP-router-AS

**bdrmap**

使用来自特定网络的目标跟踪路由、别名解析探测技术和 AS 关系推断，来推断该特定网络的边界以及连接在每个边界上的其他网络

**MAP-IT**

解决了在从许多不同网络启动的大量跟踪路由存档集合中推断所有 AS 级网络边界的艰巨挑战 

**Contributions**如下：

- 开发并实现了 bdrmapIT，它使用复杂的 bdrmap 启发式算法作为 MAP-IT 边界定位算法的附加输入。 在第 2 节中，讨论了如何利用 bdrmap 和 MAP-IT 的优势，第 3 节概述了合成方法，第 4、第 5 和第 6 节详细介绍了算法。
- 展示了 bdrmapIT 优于之前任一方法的准确性和覆盖范围。 尽管没有在任何验证网络中使用 traceroute VP，还是根据来自第 1 层、大型访问和两个 R&E 网络的真实情况验证了 bdrmapIT，实现了 91.8%-98.8% 的准确率。 还证明了 bdrmapIT 的准确性与有利位置的数量无关：当将 VP 的数量从 80 减少到 20 时，性能是等效的。
- 发布实现和源代码以提高可重复性，以便其他人可以使用工具进行他们自己的分析。 将 bdrmapIT 纳入 CAIDA 的 ITDK生成过程。

### 2021-ICMLC-Inter-domain Link Inference with Confidence Using Naïve Bayes Classifier

方法：概率模型（贝叶斯推断）

这篇文章首先将跨域链路推断的问题重新归类为一个分类问题：一个链路是域间链路还是域内链路。然后构建了一个基于朴素贝叶斯的概率模型对这个分类模型进行推断

![image-20220306155606674](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20220306155606674.png)

步骤如下：

- 路径预处理（Path Preprocessing）：首先对获取到的traceroute Path进行一些预处理，干掉一些错误hop，比如hop loop，使用CAIDA Ark收集Traceroute
- 构建IP级别的拓扑图（Construct the network topology）：基于前面的结果建立IP级别的拓扑图
- IP2AS映射（IP-to-AS mapping）：将IP地址映射到AS，采用CAIDA RouteView数据，实际就是IP-node-AS方法
- 设计IP链路特征（Design IP link feature）：提取IP链路的一系列特征，包括：
  - 链路的AS商业关系
  - 统计链路的前向链路和后向链路的数量
  - IP向量的欧几里得距离
- 建立概率模型（Build probabilistic models）：建立贝叶斯概率推断模型
- IP链路分类（IP Link Classification）：通过建立的模型、收集的特征数据对链路进行分类

实验：

这篇论文只有两个对比工作：

- simple heuristic：两个分属于不同AS的node之间的link就是跨域链路
- conventional heuristic：在simple heuristic的基础上加上对根据商业关系的推断

使用数据集：

CAIDA Ark：Traceroute收集工具

RouteViews：CAIDA的IP2AS数据集

Team Cymru IP2AS mapping tool：IP2AS映射工具

CAIDA IXP Prefixes数据集：CAIDA收集的

CAIDA AS Relationship：CAIDA的商业关系

个人评价：这篇文章的思路很有意思，基本就是照搬NSDI'19的那篇ProbLink，然后将贝叶斯推断的迭代更新方法应用到域间链路推断这个问题上来。但是提取的特征感觉过于少了，不足以构成很强的特征集合，并且对比工作就两个，一个simple启发式一个conventional启发式，可能是作者也觉得自己的方法不完善或者不靠谱，所以没有敢和MAP-IT和BdrMap-IT等进行对比；不过我觉得可以对这篇文章的思路进行扩展一下，在提取特征的阶段进行更加严格的筛选和特征工程说明，应该能取得挺好的效果。并且这篇文章的推断是在IP2AS已经完成的基础上进行的，即对AS边界几跳的路由器进行推断，可以在他的思路上进行扩展。

## 数据集总结



## 文章总结

IP2AS可以分成两种：

- IP-路由器-AS（IP-router-AS）：首先将IP地址映射到路由器，然后将路由器映射到AS（别名解析），往往采用一系列启发式算法对路由器到AS之间的映射进行推断

- 配对（pair-matching）：基于traceroute和BGP AS路径相同的假设，修改IP2AS映射以最大化配对成功的traceroute-BGP路径对


| 发表时间 | 发表会议/期刊 | 论文名                                                       | IP2AS方法           |
| -------- | ------------- | ------------------------------------------------------------ | ------------------- |
| 2003     | SIGCOMM       | Towards an Accurate AS-Level Traceroute Tool                 | pair matching       |
| 2004     | INFOCOM       | Scalable and Accurate Identification of AS-Level Forwarding Paths | pair matching       |
| 2009     | CoNEXT        | Where the Sidewalk Ends: Extending the Internet AS Graph Using Traceroutes From P2P Users | pair matching       |
| 2010     | PAM           | Toward Topology Dualism: Improving the Accuracy of AS Annotations for Routers | IP-router-AS        |
| 2010     | PAM           | Quantifying the Pitfalls of Traceroute in AS Connectivity Inference | pair matching       |
| 2011     | JSAC          | A Framework to Quantify the Pitfalls of Using Traceroute in AS-Level Topology Measurement | pair matching       |
| 2016     | IMC           | bdrmap: Inference of Borders Between IP Networks             | IP-router-AS        |
| 2016     | IMC           | MAP-IT: Multipass Accurate Passive Inferences from Traceroute | IP-router-AS        |
| 2018     | IMC           | Pushing the Boundaries with bdrmapIT: Mapping Router Ownership at Internet Scale | IP-router-AS        |
| 2021     | ICMLC         | Inter-domain Link Inference with Confidence Using Naïve Bayes Classifier | Probabilistic Model |

其他相关研究的文章

| 发表时间 | 发表期刊/会议                           | 题目                                                         | 主要内容                                                     |
| -------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2021     | IMC                                     | Identifying ASes of State-Owned Internet Operators           | 准确识别全球国有互联网运营商及其自治系统编号 (ASN)           |
| 2021     | IMC                                     | ASdb: A System for Classifying Owners of Autonomous Systems  | 建立了一个ASdb系统，它使用自已建立的商业智能数据库和机器学习的数据来对 AS 进行大规模准确分类 |
| 2008     | SIGMETRICS                              | In Search of the Elusive Ground Truth: The Internet’s AS-level Connectivity Structure | 通过对一组 AS 的案例研究来评估推断的AS级别拓扑的质量         |
| 2010     | ToN                                     | The (In)Completeness of the Observed Internet AS-level Structure | 通过对一组 AS 的案例研究来评估推断的AS级别拓扑的质量（上面那篇文章的扩展版本） |
| 2007     | SIGCOMM                                 | Observing the Evolution of Internet AS Topology              | 制定了拓扑活性（liveness）问题，并基于对 BGP 数据的分析提出了解决方案。 |
| 2006     | INFOCOM                                 | The Internet Dark Matter – on the Missing Links in the AS Connectivity Map | 定位AS Map中的Missing Link                                   |
| 2007     | NSDI                                    | A Systematic Framework for Unearthing the Missing Links: Measurements and Impact | 定位AS Map中的Missing Link                                   |
| 2010     | IMC                                     | Towards an AS-to-organization map                            | AS到organization的映射                                       |
| 2014     |                                         | Who runs the Internet? Classifying Autonomous Systems into industries | AS到organization的映射                                       |
| 2006     | PAM                                     | Revealing the Autonomous System taxonomy: The machine learning approach | AS分类（机器学习方法）                                       |
| 2015     | IEEE Communications Surveys & Tutorials | A survey of techniques for Internet topology discovery       | 互联网拓扑发现的综述                                         |
| 2010     | The European Physical Journal           | Evolution of the Internet AS-level ecosystem                 |                                                              |

| 发表时间 | 发表期刊/会议 | 题目                                                         | 主要内容                                                     |
| -------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2002     | IMW           | Topology Inference from BGP Routing Dynamics                 | 通过BGP消息推断AS级别的拓扑图                                |
| 2003     | SIGCOMM       | Internet Connectivity at the AS-level: An Optimization-Driven Modeling Approach | 推断AS最佳的上层供应商AS                                     |
| 2005     | SIGCOMM CCR   | Collecting the Internet AS-level Topology                    | 从RouteViews，RIPE RIS和一系列Looking Glass，路由服务器和路由注册表构建AS级别拓扑图，并使用路由更新更新拓扑图 |
| 2006     | SIGCOMM       | Building an AS-Topology Model that Captures Route Diversity  | 基于模拟的启发式迭代构建与所有观察到的路径一致的 AS 模型     |
| 2008     | SIGMETRICS    | In Search of the Elusive Ground Truth: The Internet’s AS-level Connectivity Structure | 通过对一组 AS 的案例研究来评估推断的AS级别拓扑的质量         |
|          |               | Cyclops: The AS-level Connectivity Observatory               |                                                              |
| 2015     | CoNEXT        | Mapping Peering Interconnections to a Facility               | 将Peering的连接映射到具体的设施上                            |
| 2016     | IMC           | Detecting Unusually-Routed ASes: Methods and Applications    |                                                              |
|          |               | AS Relationships Inference from the Internet Routing Registries |                                                              |
|          |               | A Study on Traceroute Potentiality in Revealing the Internet AS-level Topology |                                                              |
| 2007     | GLOBECOM      | An AS Border Judgment Method based on IP Path Information    | 对比通过规则判断AS边界和通过别名解析判断AS边界两种方法       |
| 2021     | ISNCC         | Analysis of Autonomous System Level Internet Topology Graphs and Multigraphs | 通过比较 AS 级图和 AS 级多重图来分析 Internet 的结构         |
| 2009     | CMC           | Importance of IP Alias Resolution in Sampling Internet Topologies | 对采样偏差对 Internet AS 级拓扑推理的影响进行了系统的定性研究 |
| 2006     | JSAC          | Network Topology Inference Based on End-to-End Measurements  | 使用类似 traceroute 的端到端测量来推断一组主机的底层拓扑     |
|          |               | Subnet Level Network Topology Mapping                        |                                                              |
|          |               | The Internet: A System of Interconnected Autonomous Systems  | AS分类                                                       |
| 2019     | ToN           | On Mapping the Interconnections in Today’s Internet          |                                                              |
| 2021     | ToN           | O Peer, Where Art Thou? Uncovering Remote Peering Interconnections at IXPs |                                                              |
|          |               |                                                              |                                                              |
