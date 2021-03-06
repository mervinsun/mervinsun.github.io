---
layout: post
title:  "云存储管控面思考"
date:   2016-11-18 8:40:15
categories: Storage
excerpt:
---

* content
{:toc}


## 序

云是IT基础设施必然趋势，传统企业存储也在发展其云形态的存储，例如EMC的ECS，HPE的HCP，CleverSafe等，那么云存储的管控面应该关注哪些方向，整理些想法。

## 云存储管控面发展思考


   ![ruby-gems]({{ "/css/pics/20161119/storage_m_n_c.png"}})

---


### 说明：


1. **Service**，管控面的第一层，云存储的能力对外应该是以服务形式发布的，包括基础的块、文件、对象存储，也包括数据保护、SLA，这些都应该是以服务的方式发放出去，传统的设备管理配置已经不在试用，因为传统的设备管理是面向管理员的，而云上的用户是租户。
服务层关注存储的发放、云运营、运维系统的集成：
 - ***Variety***，存储服务需要考虑多样性，云存储的workload区别与传统企业存储，需要考虑新型应用workload，例如大数据、IoT、AI，以及传统的Oracle、SAP，典型存储有Robin System、Iguazio
 - ***Hybrid***, 基础设施需要支持混合形态，从物理机、到虚拟化，到公有云和混合云，多DC、多云、混合云是公有云发展的过程
 - ***Aware***，存储服务需要感知上层的计算，针对计算特点发放存储服务，例如VM-aware有VSAN、Tintri, 未来容器可能成为计算主流，真正的Container-aware的存储可能成为云存储的主要形态
2. **Store**，关注数据如何存储，存储如何调度和运维，是传统管控面关注的最多的一层。和传统企业存储不同的是，这里管控面需要调度优化存储，保证存储的SLA。
 - ***QoS***，指存储需要按照业务的QoS调度，并实现运行时的QoS保障，未来存储自身可能也是Cloud-native的，那么与其它Cloud-native App一起部署时，还要考虑存储与应用就近部署和执行
 - ***High Reliability***，云存储更重要的是数据需要高可用，复制、备份、自动故障恢复、弹性扩展都很重要
 - ***Cloud-based***，运维云化，对各Cloud、Region、AZ的存储集群集中运维管理，简化运维的人力成本
3. **Data**，最下层是数据，如文件、对象、Table粒度，传统企业存储仅感知到块、共享层面，用户无法感知到存储真正使用情况，冷热文件、活动用户，无法快速作出价值判断和运维决策，需要提供数据感知、洞察、规划能力。
 - ***Velocity***，数据感知能力，能够快速搜索数据，访问数据，并实现数据流动，典型的例子有DataGravity、PrimaryData
 - ***Insight***，数据洞察能力，能够分析用户行为，数据冷热，实施分析数据性能，快速定位性能瓶颈，分析数据使用率，数据成本。典型的例子有DataGravity,Qumulo
 - ***Planning***，给用使用计划，配额、策略，帮助用户决策

---

## 云存储管控面存储能力构筑设想：

   ![ruby-gems]({{ "/css/pics/20161119/storage_m_n_c_f.png"}})



-
