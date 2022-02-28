## 2003-SIGCOMM-Towards an Accurate AS-Level Traceroute Tool

方法类型：pair-matching

## 2004-INFOCOM-Scalable and Accurate Identification of AS-Level Forwarding Paths

方法类型：pair-matching

对前面SIGCOMM的文章进行了优化，将IP2AS映射问题转化为了path-pairs的一系列优化问题，使用动态规划和迭代更新提升准确度

## 2009-CoNEXT-Where the Sidewalk Ends: Extending the Internet AS Graph Using Traceroutes From P2P Users

提出了一系列启发式算法，用来从traceroute数据钟找到missing的AS级别的链路

## 2010-PAM-Toward Topology Dualism: Improving the Accuracy of AS Annotations for Routers

方法类型：IP-router-AS

首先使用别名解析将不同的interface解析到同一个路由器上，得到路由器级别拓扑，然后利用五步启发式算法推断AS级别拓扑：

![image-20220228213849177](C:\Users\WhileBug\AppData\Roaming\Typora\typora-user-images\image-20220228213849177.png)

1. **single**:如果一个路由器前后的路由器都属于一个AS，那么判定该路由器属于该AS
2. **election**：如果一个路由器前后的路由属于多个AS，那么判定该路由器属于前后路由器所属最多的那个AS，但是可能出现AS A和AS B都有同样最多个所属前后路由器的情况
3. **neighbor**：根据邻居路由器所属的AS进行推断
4. **customer**
5. **degree**

## 2010-PAM-Quantifying the Pitfalls of Traceroute in AS Connectivity Inference

方法类型：pair-matching

## 2011-JSAC-A Framework to Quantify the Pitfalls of Using Traceroute in AS-Level Topology Measurement

方法类型：pair-matching

## 2011-CoNEXT Workshop-Revisiting IP-to-AS mapping for AS-level traceroute 

IP2AS可以分成两种：

IP-路由器-AS（IP-router-AS）：首先将IP地址映射到路由器，然后将路由器映射到AS（别名解析）

配对（pair-matching）：基于traceroute和BGP AS路径相同的假设，修改IP2AS映射以最大化配对成功的traceroute-BGP路径对

这篇文章总结了11年之前关于pair matching的研究

第一篇关于pair-matching的工作（INFOCOM 2004）在2004提出，之后一直没有被改进过，直到PAM10和JSAC11使用pair matching方法评断使用traceroute的pitfall

现有的最佳配对匹配方法是INFOCOM 2004。 以 /24 前缀为粒度来修改 IP 到 AS 的映射。 在现实世界中并非如此：相同/24前缀的IP地址可能不属于同一个AS，尤其是对于路由器的IP地址。 原因如下： 

(1)边界AS路由器。 它可以使用分配给相邻 AS 的 IP 地址；

(2) 互联网交换点 (IXP)。 它用于在不同的 AS 之间交换流量。 

相同IXP前缀的IP地址被不同的AS使用。 尽管 [2] 允许将前缀映射到一组 AS，但它会产生歧义。 因此我们将[2]的方法称为带歧义的前缀粒度方法（PGMA）。 正如我们所知道的，一个 IP 地址仅供一个 AS 使用，因此 IP 到 AS 的映射应该是确定性的，而不是模棱两可的。

## 2016-IMC-bdrmap: Inference of Borders Between IP Networks

方法类型：IP-router-AS

## 2016-IMC-MAP-IT: Multipass Accurate Passive Inferences from Traceroute

方法类型：IP-router-AS

## 2016-IMC-Pushing the Boundaries with bdrmapIT: Mapping Router Ownership at Internet Scale

方法类型：IP-router-AS

## 总结