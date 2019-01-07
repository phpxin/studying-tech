**目录**   
> cmd: 命令行工具入口，包括 geth,bootnode 等节点必备工具，从这里开始解读，可以线性的方式解读以太坊源代码      
> ethclient: 以太坊RPCAPI实现    
> 


**Eth包**
> 以太坊将功能模块封装到各个目录中通过init方法实现各模块初始化操作  
> geth 工具是创建节点，启动各种服务（RPC，IPC，Miner等）的工具     
> cmd/geth/main.go init()，首先将 geth函数作为默认 action 传递给 app（当没有说明要执行子指令时，程序默认运行geth方法，也就是开启节点服务器）， app.Commands 声明了geth的子指令，其中包括创建一个私有节点的initCommand，app.Flags 依次将支持的参数（--xx）添加到解析    
> main() 函数执行 app.run 开始运行程序     
> app 是一个开源应用程序管理框架 "gopkg.in/urfave/cli.v1" [cli.v1 文档](https://godoc.org/gopkg.in/urfave/cli.v1)，go-eth使用 [toml](https://github.com/toml-lang/toml) 管理配置文件     
> app.run 会自动调用在上文已经挂载到默认Action的geth方法 main.go:235   
> 函数geth是主入口，调用 startNode 并将命令行上下文传入,开启一个节点服务并且启动所有已注册的服务（rpc，ipc等），node.Wait 很简单就是阻塞并监听node.stop（channel）当有中断时channel会返回，这时退出block模式结束进程   
> Ethereum 结构体(backend.go) 是以太坊核心节点服务实现，可以说是业务逻辑层的大Boss   
> ProtocolManager 结构体(handler.go) 是节点的管理员，这个家伙是 Ethereum 的副手，他的主要工作就是把收发新的区块和事务 (handleMsg)，或者给其他节点提供区块的具体内容      


``` golang
// geth is the main entry point into the system if no special subcommand is ran.
// It creates a default node based on the command line arguments and runs it in
// blocking mode, waiting for it to be shut down.
func geth(ctx *cli.Context) error {
	node := makeFullNode(ctx)
	startNode(ctx, node)
	node.Wait()
	return nil
}
```     
> 配置文件加载：geth->makeFullNode->makeConfigNode 一路跟过来看到 cmd/geth/config.go:112 行在加载配置文件   
> eth.DefaultConfig 我们最关注的geth启动时配置（比如我们在创建本地私有节点，geth init ./genesis.json 这个配置文件也是这样被解析的，只是这里是默认我们geth不加任何参数时的默认配置）   
> geth console,geth attach 在 cmd/geth/consolecmd.go 中定义，这两个是命令行客户端交互方式   
> 我们可以看到 subcommand console 启动一个嵌入式JS解析器的同时也会启动节点    

```golang   
// Otherwise print the welcome screen and enter interactive mode    
console.Welcome()    
console.Interactive()    
```   

> download 子包是负责区块链数据同步的，cmd/utils/falgs.go:RegisterEthService 这里判断是 light 还是 full，然后启动对应的同步模式 les.New 或 eth.New   
> 同步模式：eth/backend.go:New()，调用 NewProtocolManager()，NewProtocolManager() 调用 downloader.New 开始同步 















