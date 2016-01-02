---
layout: post
title:  "Docker&容器在IT行业应用现状"
date:   2015-12-22 21:50:15
categories: docker
excerpt: 
---

* content
{:toc}


### 序

整理下之前分析的容器(Docker)在IT领域的应用现状，主要看了几个领域：公有云、PaaS、混合云、私有云和存储领域。容器技术本质上还算是技术领域，这里把存储考虑进来主要因为个人工作在存储领域，所以更多关注存储在容器生态的现状。

---

### 公有云领域

#### 公有云运营商已普遍提供了基于VM的容器服务，微服务可能成为未来IT架构新趋势。


1. **公有云容器服务**，AWS、微软、Google等主要公有云服务商，积极在VM之上推出其容器服务。

    ![ruby-gems]({{ "/css/pics/20160102/CW-ECS-diagram.png"}})  
	AWS的ECS在Amazon EC2 实例集群中轻松运行和管理支持Docker的应用程序，并且提供了容器的编排、集群管理、任务调度能力，而且还提供了LB、HA、故障恢复等高级运维能力。

2. **公有云微服务**，以微软Azure为代表的新生代公有云服务提供了基于容器的微服务：

    ![ruby-gems]({{ "/css/pics/20160102/azure_service_fabric.png"}})

    Azure发布的Service Fabric属于PaaS层服务，目前支持与其visual studio对接，Service Fabric目前使用的是自家的容器引擎，后面宣称要支持Docker，但前提是Docker可以运行于其Windows Server下面，而目前还仅仅是宣称2016年计划实现而已。更多信息参考：
    [http://www.informationweek.com/cloud/infrastructure-as-a-service/microsoft-offers-azure-service-fabric-for-distributed-apps/d/d-id/1320059](http://www.informationweek.com/cloud/infrastructure-as-a-service/microsoft-offers-azure-service-fabric-for-distributed-apps/d/d-id/1320059 "http://www.informationweek.com/cloud/infrastructure-as-a-service/microsoft-offers-azure-service-fabric-for-distributed-apps/d/d-id/1320059")

### PaaS

#### PaaS是容器技术当前最主要使用的场景，Docker很有可能取代传统容器技术，取代IaaS。

近两年PaaS层框架层出不穷，传统和新生的IT厂商都在打造其PaaS栈，如vmware的Cloud Foundry,Google的App Engine，Radhat的OpenShift，IBM的Bluemix等等，其针对DevOps的部署和资源隔离工具多使用容器技术，而这些厂商也在积极的支持Docker。

   ![ruby-gems]({{ "/css/pics/20160102/cloudfoundry.png"}})

Cloud Foundry的DEA原生支持其容器引擎Garden，但为了迎合Docker生态，目前也兼容了Docker引擎。


### 虚拟化、私有云、混合云

#### 私有云厂商积极打造云原生软件栈和基础平台，并助理其混合云完善软件定义企业战略。

虚拟化、私有云领域，还是看看vmware在容器领域的动作。

1. **打造原生云应用软件栈**，vmware 2015年推出其Photon平台，积极打造其原生云软件栈。

    ![ruby-gems]({{ "/css/pics/20160102/photon_platform.png"}})  
	Photon是vmware 2015年推出的Container OS,用于部署其ESXi之上，来融合容器和VM的优势。目前其战略是基于Photon构筑混合云和云原生栈的双平台。

2. **助力混合云**，vSphere引入了Docker和Google的k8s，使应用更容易部署于其vCloud Air平台之上。

    ![ruby-gems]({{ "/css/pics/20160102/vrealize_docker.png"}})  
	
### 存储领域

#### 聚焦云形态存储，积极融入Docker生态
我们主要看下EMC在容器领域的动作

1. **弹性云存储+SDS**，EMC 发布其弹性云存储ECS，之前从ViPR。2.0剥离出来的对象和分布式文件服务，定位于云形态下存储场景以及第三平台应用。
	
	ECS架构：
    ![ruby-gems]({{ "/css/pics/20160102/ECS2.png"}})  
	
	ViPR结合ECS，提供Docker数据卷和私有仓库构建：
    ![ruby-gems]({{ "/css/pics/20160102/docker_vipr.png"}})  
	具体参考[http://www.recorditblog.com/post/how-to-create-a-web-scale-infrastructure-based-on-docker-coreos-vulcand-and-kubernetes-and-why-object-storage-becomes-the-de-facto-data-repository/](http://www.recorditblog.com/post/how-to-create-a-web-scale-infrastructure-based-on-docker-coreos-vulcand-and-kubernetes-and-why-object-storage-becomes-the-de-facto-data-repository/ "http://www.recorditblog.com/post/how-to-create-a-web-scale-infrastructure-based-on-docker-coreos-vulcand-and-kubernetes-and-why-object-storage-becomes-the-de-facto-data-repository/")

2. **开源卷管理插件**，EMC在Docker积极投入Flocker和其自研的HEX-Ray卷管理组价，希望在容器卷管理领域获得更多话语权。

    ![ruby-gems]({{ "/css/pics/20151220/REX-Ray architecture.png"}}) 

### 总结

#### IT领域中Docker主要用于云环境下的容器计算服务，以及新型PaaS层的服务构建，存储领域则主要以迎合生态为主。

** 原创，引用请注明出处，谢谢。