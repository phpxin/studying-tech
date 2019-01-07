**P2P**   
> node/node.go:Start() 开启节点服务时会打开一个p2p服务，running := &p2p.Server{Config: n.serverConfig} 这行代码配置一个p2p，然后在下文调用running.Start() 开始工作   
> p2p/server.go 是该模块的入口程序文件，Start 函数中初始化各个channel（添加peer，删除peer，退出等），srv.setupDiscovery() 开启节点发现服务   
>  

**Miner**
> worker.go:mainLoop 接收各种事件，其中包括提交新的工作   
> worker.go:taskLoop mainLoop 收到newWork通知，经过一系列处理最终会调用 worker::commit 方法，commit 方法会包装一个task，给 worker::taskCh, taskLoop 主要监听 taskCh 并做进一步处理   
> worker.go:resultLoop taskLoop 将task处理完成后会调用worker::engine (consensus/ ethhash/clique pow/pos) 的 seal 方法（见consensus 模块分析）进行“挖矿”，挖矿之后，Seal 会将 Block 放进 worker.resultCh 中，resultLoop 读到该channel后执行后续操作（如：提交区块和状态state到数据库）   

**Consensus**
> 共识算法模块：ethhash，POW共识；clique，POS共识（[仅用于testnet](https://ethfans.org/posts/Clique-Consensus-Algorithm)）   
> 具体使用哪个算法进行挖矿，是在geth初始化时在实例化eth/les时确定的（cmd/utils/flags.go/RegisterEthService）   
> ethhash 和 clique 算法都继承自 Engine 接口   
> 通过源码 params/config.go:RinkebyChainConfig 可以看到，Clique 算法只在Rinkeby测试网络使用   
> ethhash 中的 Seal 通过调用 ethhash::mine 进行挖矿（这里从源码可以看到节点根据配置的挖矿线程数量开启多个协程进行“挖矿”），mine 是真正的 POW 矿工，它计算区块hash（挖矿过程）并将区块返回    
> 每个区块的复杂度在 Block.Header 中的 Difficulty 保存    
