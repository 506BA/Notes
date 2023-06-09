分片技术作为区块链扩容的主流方式之一，能够在不降低区块链去中心化程度的同时实现高性能的链上扩容，从而解决区块链可拓展性不足以及吞吐量较低的问题。  

分片技术将整个区块链网络分成不同分片，由各分片的节点负责处理所在分片的事务以及存储分片的状态 ，通过并行验证事务，整个区块链的吞吐量近似线性地提升。同时，随着节点数目的增加，整个网络的分片数量也 增多，全网的事务处理能力进一步提高。分片技术一般分为定义**分片配置、片内和跨片共识协议、重配置**等阶段，从而构成一个完整的分片区块链系统。 在区块链分片技术中， 为了保证各个分片的安全性并防止节点之间作恶行为发生，在一个分片纪元之后，从原有分片中取出一部分节点并与其他分片中的节点进行交换，当有新节点加入时，需要对新加入节点进行分配。  

区块链主要包括网络分片、交易分片、状态分片3种分片方式。  

- 网络分片通过一定的组织方式将整个网络分成不同分片，各个分片并行处理整个区块链中的部分交易，各部分交易完全不同，从而同时完成多笔交易验证。网络分片通常要考虑分片规模、分片安全性、分片方式等因素，分片过程通常使用随机函数来实现。  
- 交易分片使得各个网络分片对交易具有更强 的处理能力，其将客户端的跨片交易分成若干个相
关的子交易，不同分片的跨片交易可以并行处理。  
- 为了降低各个节点存储账本的压力，状态分片将各部分完全不同的账本分别存储在各个分片（分片内的节点往往存储同一版本的账本），整个分片网络组成一个完整的账本。  

在现行的技术条件下，网络分片和交易分片都具有比较理想的实现方式，而实现状态分片还存在很多技术问题，状态分片制约着分片技术的发展。  

共识协议    
在完成分片配置后，区块链需要进行交易共识，共识分为片内共识和跨片共识。片内共识要求同一分片内各个节点按照所在分片的协议进行共识和广播，最终得出整个分片的共识结果，片内共识协议主要分为基于PoW的片内共识协议和基于BFT的片内共识协议。 在片内共识时，片内各个节点经过彼此间的广播可直接进行交流。在跨片共识时，由于所存储的信息不相交，不同分片之间各个节点在交易验证过 程中需要交流账本状态，因此跨片交流的基本单位是各个分片，不同分片执行共同的跨片协议以实现共识。跨片共识的主要方式包括交易原子化、交易 集中化和采用类路由协议。