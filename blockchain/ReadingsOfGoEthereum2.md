**Console包**   
> console 包封装了GETH的命令行解析，它使用了开源Go嵌入JS库 [otto](https://github.com/robertkrimen/otto)   
> console 在 subcommand console、attach
> 

**Rpc包**
> 各种RPC连接，http、ipc、ws 等   
> _, isHTTP := conn.(*httpConn) 这行代码可以看到，程序通过类型转换是否成功来判断这是不是一个http连接    
> rpc/endpoints.go 是真正的初始化各个rpc终端的程序文件，其中 StartHTTPEndpoint 开启一个http终端    
> StartHTTPEndpoint 循环调用 RegisterName ，该函数通过反射技术将需要提提供的接口注册到http服务（或ws等其他服务）     
> 在用web3的接口时会发现所有的回调函数都是返回 (data,error) 这种，在这个 rpc/utils.go:suitableCallbacks 我们可以看到，这是以太坊程序对外提供rpc函数时对函数签名的限制     
> 我们在 RegisterName 加了打印看下具体提供rpc的函数都在哪里

```golang
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *ethapi.PublicEthereumAPI
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *ethapi.PublicBlockChainAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *ethapi.PublicTransactionPoolAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *ethapi.PublicAccountAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *eth.PublicEthereumAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *eth.PublicMinerAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *downloader.PublicDownloaderAPI 
INFO [11-21|22:13:05|rpc/server.go:86] lx:RegisterName is *filters.PublicFilterAPI 
```

**Node包**
> 执行默认的geth action 将会通过 utils.StartNode(stack) 这行代码启动节点服务   
> 一路跟踪到node/node.go:Start() 函数，该函数是节点启动逻辑，在这里可以看到许多启动节点时会打印的信息，n.startRPC(services) 这行开启所有配置的远程过程调用接口（http,ws,ipc....）    
> n.stop 这里设置了channel，正是在geth主入口node.Wait()处阻塞监听的stop通道   
> node/node.go:startHTTP(...) 开启http服务，geth通过 [cors](https://github.com/rs/cors) 实现的http服务   
