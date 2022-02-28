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

## 2016-IMC-bdrmap: Inference of Borders Between IP Networks

方法类型：IP-router-AS

## 2016-IMC-MAP-IT: Multipass Accurate Passive Inferences from Traceroute

方法类型：IP-router-AS

## 2018-IMC-Pushing the Boundaries with bdrmapIT: Mapping Router Ownership at Internet Scale

方法类型：IP-router-AS

## 总结

IP2AS可以分成两种：

IP-路由器-AS（IP-router-AS）：首先将IP地址映射到路由器，然后将路由器映射到AS（别名解析）

配对（pair-matching）：基于traceroute和BGP AS路径相同的假设，修改IP2AS映射以最大化配对成功的traceroute-BGP路径对

| 发表时间 | 发表会议/期刊 | 论文名                                                       | IP2AS方法     |
| -------- | ------------- | ------------------------------------------------------------ | ------------- |
| 2003     | SIGCOMM       | Towards an Accurate AS-Level Traceroute Tool                 |               |
| 2004     | INFOCOM       | Scalable and Accurate Identification of AS-Level Forwarding Paths | pair matching |
| 2009     | CoNEXT        | Where the Sidewalk Ends: Extending the Internet AS Graph Using Traceroutes From P2P Users | pair matching |
| 2010     | PAM           | Toward Topology Dualism: Improving the Accuracy of AS Annotations for Routers | IP-router-AS  |
| 2010     | PAM           | Quantifying the Pitfalls of Traceroute in AS Connectivity Inference | pair matching |
| 2011     | JSAC          | A Framework to Quantify the Pitfalls of Using Traceroute in AS-Level Topology Measurement | pair matching |
| 2016     | IMC           | bdrmap: Inference of Borders Between IP Networks             | IP-router-AS  |
| 2016     | IMC           | MAP-IT: Multipass Accurate Passive Inferences from Traceroute | IP-router-AS  |
| 2018     | IMC           | Pushing the Boundaries with bdrmapIT: Mapping Router Ownership at Internet Scale | IP-router-AS  |

