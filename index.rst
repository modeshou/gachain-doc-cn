政务链v2技术开发文档
=======================

该文档包含了政务区块链平台（GAChain Platform）的描述，政务链平台的基础研发于2011年开展，2016年正式启动政务链（GAChain）的研发，经过政务流程团队、电子政务专家顾问团队和技术团队的努力，至2016第三季度，完成了如下工作: 

1. 创建了主要网络协议和轻量级软件客户端；
2. 开发了智能合约的概念；
3. 发布了早期版本的接口界面和合约编程语言。

由于该平台最初是为了实施电子政务领域的项目而开发的，因此团队决定将项目命名为 **政务链** (*电子政务即服务*)。2017年秋季，创建了政务链第一版本（v1）的测试网络。至2017年年底，为广东省南沙自贸区电子服务中心、中央扶贫办、天津市市场监督管理局、香港机场服务管理公司等政企机构开发出相关概念验证项目和解决方案，如多部门电子政务自动协同办公和流转、社会化代收税电子发票系统、精准扶贫平台、供应链管理等。

也是在2017年冬季，政务链团队决定在现有平台的基础上建立一个公共区块链网络，与相关公共安全部门合作，将KYC（个人身份识别）程序整合到用户账户的注册阶段。

由于该项目将不再只适应于政务服务项目，还可广泛用于商业、个人等项目的开发、应用，因此团队将项目的品牌标识做了一定修改：

1. 面对政府、事业单位的项目，依然使用“政务链”，英文名为 **GAChain** ( Government Affairs Chain);
2. 面对商业机构、个人客户，中文使用“智乾链”，英文名依然为 **GAChain** 。

GAChain平台的软件客户端起名为Molis和编程语言：智语言（G Language）为合约语言和乾语言（Chain Language）为界面编辑语言。同时，启动了源代码重构过程：软件客户端的代码被完全重写（基于JavaScript React库），创建了REST API v2的新版本，智语言（G Language）和乾语言（Chain Language）得到显着改善。另外，开发了专门生态系统的概念和功能，开始了视觉页面设计器的工作，并且实施了一些其它相关性改变和改进。

到2018年初，政务链（GAChain）平台已经做好与当前火热的区块链平台Ethereum、NEO相竞争的准备，团队相信，政务链（GAChain）在具有颠覆性的产品规划设计、优秀性能等优势，必将对现有区块链产品带来从概念到实际应用的冲击。

区块链颠覆了现有常规技术和观念，政务链颠覆了现有常规区块链！

本文档包含平台功能的最新描述，并且在更改或添加新功能时不断更新。

目录
======

.. toctree::
   :maxdepth: 4

   /introduction/what-is-Gachain.rst
   /introduction/script.rst
   introduction/templates2.rst
   introduction/appexample.rst
   introduction/api2.rst 
   introduction/vm.rst
   introduction/install.rst
   introduction/thesaurus.rst
   introduction/contractsignatures.rst
   introduction/faq.rst
