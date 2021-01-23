# Consul服务网络入门
Consul是一种网络工具，可提供功能齐全的**Service Mesh控制**和**服务发现**。 除此之外，Consul 还提供用于管理服务配置的**KV存储**。下面我们来学习如何在本地使用Consul的常见功能。

### 一  介绍
欢迎来到Consul学习平台，在这里，我们将通过示例来教你一步一步的执行Consul的一些常见功能。该入门教程将帮助你构建有关Consul的思维模型，以了解Consul如何运行的。 请你依次完成本教程的各个章节，其中一些章节的内容会依赖于前一个章节。

为了使入门教程内容更易于在本地计算机运行，你将在开发模式下使用Consul，该模式不适用于生产部署。 有关生产环境部署的指引，请跳转至Operations and Development(https://learn.hashicorp.com/consul/#operations-and-development)文档集。

继续下一个教程“**安装Consul**”，在本地计算机上安装Consul。
### 二 安装Consul

要使用Consul，需要做的第一件事就是安装它。 如果是在生产环境，你需要将Consul安装在要注册服务的每个节点上，但是在本教程中，你将在本地安装Consul，以便可以使用它来探索Consul的核心功能。 Consul以二进制文件或主要操作系统的软件包的形式发布。

如果你想要从源代码编译Consul，请参考该文档(https://www.consul.io/docs/install/index.html#compiling-from-source)。

#### 手动安装
要安装Consul，请找到适合你系统的软件包并下载。 Consul的安装包是一个zip文件。

下载Consul后，解压缩该软件包。 Consul会以名字为consul的二进制文件运行。 你可以安全删除软件包中的其他所有文件，Consul仍将可正常使用。

确保Consul二进制文件在你的环境变量PATH上可用。 你可以通过运行此命令来检查路径上可用的PATH。

	$ echo $PATH
	/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

上述命令输出的是由冒号分隔的path列表。 你可以通过将二进制文件移动到上述的path之一，或将Consul的path添加到PATH环境变量中来使Consul可用。

#### MacOx下安装

	brew install consul

无需手动配置PATH即可直接使用。
#### 验证安装
安装完Consul后，你可以通过在命令行执行`consul`命令来检查是否安装成功：

	$ consul


	Usage: consul [--version] [--help] <command> [<args>]
	
	Available commands are:
	    acl            Interact with Consul's ACLs
	    agent          Runs a Consul agent
	    catalog        Interact with the catalog
	    config         Interact with Consul's Centralized Configurations
	    connect        Interact with Consul Connect
	    debug          Records a debugging archive for operators
	    event          Fire a new event
	    exec           Executes a command on Consul nodes
	    force-leave    Forces a member of the cluster to enter the "left" state
	    info           Provides debugging information for operators.
	    intention      Interact with Connect service intentions
	    join           Tell Consul agent to join cluster
	    keygen         Generates a new encryption key
	    keyring        Manages gossip layer encryption keys
	    kv             Interact with the key-value store
	    leave          Gracefully leaves the Consul cluster and shuts down
	    lock           Execute a command holding a lock
	    login          Login to Consul using an auth method
	    logout         Destroy a Consul token created with login
	    maint          Controls node or service maintenance mode
	    members        Lists the members of a Consul cluster
	    monitor        Stream logs from a Consul agent
	    operator       Provides cluster-level tools for Consul operators
	    reload         Triggers the agent to reload configuration files
	    rtt            Estimates network round trip time between nodes
	    services       Interact with services
	    snapshot       Saves, restores and inspects snapshots of Consul server state
	    tls            Builtin helpers for creating CAs and certificates
	    validate       Validate config files/directories
	    version        Prints the Consul version
	    watch          Watch for changes in Consul

如果出现了类似于consul找不到的错误，说明你的PATH环境变量设置的有问题，请确保你的PATH环境变量包含consul的安装路径。

#### 下一步
在本章中，你在本机上安装了Consul。在下一章，我们将学习如何运行Consul agent。
### 三 运行Consul Agent

在安装完Consul之后，我们需要运行Consul agent。在本教程中，你将在开发模式下运行它，开发模式是一种不安全以及不可扩展的模式，但是可以让你不需要额外的配置就能够尝试Consul的大部分功能。你还将学习到如何优雅的关闭Consul agent。

#### Server agent 和 Client agent

在生产环境下，你应当以Server或Client模式运行每个Consul agent。每个Consul数据中心必须至少有一个Server，该Server负责维护Consul的状态，这包括有关其它Consul server和Consul client的信息，可用于发现的服务以及允许哪些服务与哪些其他服务进行通信的信息。

> 警告：我们强烈不建议使用单服务器在生产环境部署。


为了确保即使某个Consul Server agent发生故障，Consul的状态也会保留，你应该始终在生产环境中运行3~5台Server agent。 使用不超过5个并且奇数数量的Server agent可以在性能和容错能力之间取得平衡。 你可以在Consul的体系结构文档中了解有关这些要求的更多信息。


除了以Server模式运行的Consul agent外，还有以Client模式运行的。 Client agent是一个轻量级的进程，用于注册服务，运行状况检查并将服务查询转发到Server agent。 Client agent必须在Consul数据中心中运行服务的每个节点上运行，因为Client agent是有关服务运行状况的真实来源。

当你准备在生产环境中使用Consul的时候，你可以在我们的开发指南中找到有关Server agent和Client agent在生产环境部署的更多指南。现在，为了便于学习，让我们以开发模式在本地启动agent。

> 警告：永远不要在生产环境中运行  -dev 指令。


#### 启动Consul agent
以开发模式启动Consul agent

	$ consul agent -dev


	==> Starting Consul agent...
	           Version: '1.8.1'
	           Node ID: '7a82fb12-a621-004b-a8cd-98861b80d64c'
	         Node name: 'C02YM00WJG5H.local' //Consul默认使用主机名作为Nodename
	        Datacenter: 'dc1' (Segment: '<all>')
	            Server: true (Bootstrap: false)
	       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
	      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
	           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false, Auto-Encrypt-TLS: false
	
	==> Log data will now stream in as it occurs:
	
	    2020-08-07T13:43:09.554+0800 [DEBUG] agent: Using random ID as node ID: id=7a82fb12-a621-004b-a8cd-98861b80d64c
	    2020-08-07T13:43:09.561+0800 [WARN]  agent: Node name will not be discoverable via DNS due to invalid characters. Valid characters include all alpha-numerics and dashes.: node_name=C02YM00WJG5H.local
	    2020-08-07T13:43:09.568+0800 [INFO]  agent.server.raft: initial configuration: index=1 servers="[{Suffrage:Voter ID:7a82fb12-a621-004b-a8cd-98861b80d64c Address:127.0.0.1:8300}]"
	    2020-08-07T13:43:09.569+0800 [INFO]  agent.server.raft: entering follower state: follower="Node at 127.0.0.1:8300 [Follower]" leader=
	    2020-08-07T13:43:09.570+0800 [INFO]  agent.server.serf.wan: serf: EventMemberJoin: C02YM00WJG5H.local.dc1 127.0.0.1
	    2020-08-07T13:43:09.571+0800 [INFO]  agent.server.serf.lan: serf: EventMemberJoin: C02YM00WJG5H.local 127.0.0.1
	    2020-08-07T13:43:09.572+0800 [INFO]  agent.server: Handled event for server in area: event=member-join server=C02YM00WJG5H.local.dc1 area=wan
	    2020-08-07T13:43:09.572+0800 [INFO]  agent.server: Adding LAN server: server="C02YM00WJG5H.local (Addr: tcp/127.0.0.1:8300) (DC: dc1)"
	    2020-08-07T13:43:09.575+0800 [INFO]  agent: Started DNS server: address=127.0.0.1:8600 network=tcp
	    2020-08-07T13:43:09.575+0800 [INFO]  agent: Started DNS server: address=127.0.0.1:8600 network=udp
	    2020-08-07T13:43:09.575+0800 [INFO]  agent: Started HTTP server: address=127.0.0.1:8500 network=tcp
	    2020-08-07T13:43:09.576+0800 [INFO]  agent: Started gRPC server: address=127.0.0.1:8502 network=tcp
	    2020-08-07T13:43:09.576+0800 [INFO]  agent: started state syncer
	==> Consul agent running!
	    2020-08-07T13:43:09.635+0800 [WARN]  agent.server.raft: heartbeat timeout reached, starting election: last-leader=
	    2020-08-07T13:43:09.636+0800 [INFO]  agent.server.raft: entering candidate state: node="Node at 127.0.0.1:8300 [Candidate]" term=2
	    2020-08-07T13:43:09.636+0800 [DEBUG] agent.server.raft: votes: needed=1
	    2020-08-07T13:43:09.636+0800 [DEBUG] agent.server.raft: vote granted: from=7a82fb12-a621-004b-a8cd-98861b80d64c term=2 tally=1
	    2020-08-07T13:43:09.636+0800 [INFO]  agent.server.raft: election won: tally=1
	    2020-08-07T13:43:09.636+0800 [INFO]  agent.server.raft: entering leader state: leader="Node at 127.0.0.1:8300 [Leader]"
	    2020-08-07T13:43:09.636+0800 [INFO]  agent.server: cluster leadership acquired
	    2020-08-07T13:43:09.637+0800 [INFO]  agent.server: New leader elected: payload=C02YM00WJG5H.local
	    2020-08-07T13:43:09.637+0800 [DEBUG] agent.server: Cannot upgrade to new ACLs: leaderMode=0 mode=0 found=true leader=127.0.0.1:8300
	    2020-08-07T13:43:09.642+0800 [DEBUG] connect.ca.consul: consul CA provider configured: id=07:80:c8:de:f6:41:86:29:8f:9c:b8:17:d6:48:c2:d5:c5:5c:7f:0c:03:f7:cf:97:5a:a7:c1:68:aa:23:ae:81 is_primary=true
	    2020-08-07T13:43:09.657+0800 [INFO]  agent.server.connect: initialized primary datacenter CA with provider: provider=consul
	    2020-08-07T13:43:09.657+0800 [INFO]  agent.leader: started routine: routine="federation state anti-entropy"
	    2020-08-07T13:43:09.657+0800 [INFO]  agent.leader: started routine: routine="federation state pruning"
	    2020-08-07T13:43:09.657+0800 [INFO]  agent.leader: started routine: routine="CA root pruning"
	    2020-08-07T13:43:09.657+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local
	    2020-08-07T13:43:09.657+0800 [INFO]  agent.server: member joined, marking health alive: member=C02YM00WJG5H.local
	    2020-08-07T13:43:09.658+0800 [INFO]  agent.server: federation state anti-entropy synced
	    2020-08-07T13:43:09.716+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T13:43:09.716+0800 [INFO]  agent: Synced node info
	    2020-08-07T13:43:09.797+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T13:43:09.797+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T13:43:09.797+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T13:44:09.638+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local
	    2020-08-07T13:44:12.669+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T13:44:12.669+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T13:45:09.575+0800 [DEBUG] agent.server.router.manager: No healthy servers during rebalance, aborting
	    2020-08-07T13:45:09.639+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local
	    2020-08-07T13:45:45.883+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T13:45:45.883+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T13:46:09.640+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local

日志显式Consul agent已启动并且正在传输一些日志流数据。日志还显式该Consul agent正在以Server模式运行，并且被选举为Leader。此外，本地agent已被标记为数据中心的正常成员。从上面可以看出我们在后续查询服务时需要用到的端口信息：

    Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
    Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)

> OS X用户注意事项：Consul使用你的主机名作为默认节点名。 如果你的主机名包含句点，则对该节点的DNS查询将不适用于Consul。 为避免这种情况，请使用`-node`标志显式设置节点的名称。

#### 查找数据中心成员
通过在新的终端窗口中运行`consul menbers`命令，可以检查Consul数据中心的成员。 该命令的输出结果列出了数据中心中所有的agent。 稍后我们将介绍将其它Consul agent一起加入到数据中心的方法，但是目前数据中心中只有一个成员（你的计算机）。

	$ consul members
	
	Node                Address         Status  Type    Build  Protocol  DC  Segment
	C02YM00WJG5H.local  127.0.0.1:8301  alive   server  1.8.1  2         dc1  <all>

该输出展示了你在上一个窗口启动的Consul agent, 它的地址，健康状态，它在数据中心的状态以及一些版本信息。你也可以通过`-detailed`标志来展示更多元信息。

同时你也可以在另一个窗口看到一次查询日志：

	2020-08-07T13:56:57.969+0800 [DEBUG] agent.http: Request finished: method=GET url=/v1/agent/members?segment=_all from=127.0.0.1:55189 latency=265.761µs

`menbers`指令是针对Consul client运行的，通过流言协议获取client的信息。client拥有的信息最终是一致的，但是在任何时间点，其对全局状态的视角都可能与Server agent上的状态不完全匹配。 要获得与全局高度一致的视图，请调用HTTP API，该请求会将请求转发到Consul Server agent。

	$ curl localhost:8500/v1/catalog/nodes

#
	
	[
	    {
	        "ID": "7a82fb12-a621-004b-a8cd-98861b80d64c",
	        "Node": "C02YM00WJG5H.local",
	        "Address": "127.0.0.1",
	        "Datacenter": "dc1",
	        "TaggedAddresses": {
	            "lan": "127.0.0.1",
	            "lan_ipv4": "127.0.0.1",
	            "wan": "127.0.0.1",
	            "wan_ipv4": "127.0.0.1"
	        },
	        "Meta": {
	            "consul-network-segment": ""
	        },
	        "CreateIndex": 10,
	        "ModifyIndex": 12
	    }
	]

除了HTTP API之外，你还可以使用DNS接口来查找节点。 除非你启用了缓存，否则DNS接口会将你的查询请求发送到Consul Server。 要执行DNS查找，你必须指向Consul agent的DNS服务器，该服务器默认在**8600**端口上运行。 DNS参数的格式（例如Judiths-MBP.node.consul）将在后面详细介绍。

	$ dig @127.0.0.1 -p 8600 Judiths-MBP.node.consul
	
	; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 Judiths-MBP.node.consul
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7104
	;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2
	;; WARNING: recursion requested but not available
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;Judiths-MBP.node.consul.   IN  A
	
	;; ANSWER SECTION:
	Judiths-MBP.node.consul. 0  IN  A   127.0.0.1
	
	;; ADDITIONAL SECTION:
	Judiths-MBP.node.consul. 0  IN  TXT "consul-network-segment="
	
	;; Query time: 0 msec
	;; SERVER: 127.0.0.1#8600(127.0.0.1)
	;; WHEN: Mon Jul 15 19:43:58 PDT 2019
	;; MSG SIZE  rcvd: 104

#### 停止Consol agent
可以使用`consul leave`命令停止consul agent。该指令会优雅的停止Consul agent，使其离开Consul数据中心并关闭。

	$ consul leave
	Graceful leave complete

如果你切换回之前的那个带有Consul日志流输出的窗口，可以看到这些日志显示Consul agent已离开数据中心。

	2020-08-07T14:15:19.445+0800 [INFO]  agent.server: server starting leave
	    2020-08-07T14:15:19.445+0800 [INFO]  agent.server.serf.wan: serf: EventMemberLeave: C02YM00WJG5H.local.dc1 127.0.0.1
	    2020-08-07T14:15:19.445+0800 [INFO]  agent.server: Handled event for server in area: event=member-leave server=C02YM00WJG5H.local.dc1 area=wan
	    2020-08-07T14:15:19.445+0800 [INFO]  agent.server.router.manager: shutting down
	    2020-08-07T14:15:22.446+0800 [INFO]  agent.server.serf.lan: serf: EventMemberLeave: C02YM00WJG5H.local 127.0.0.1
	    2020-08-07T14:15:22.446+0800 [INFO]  agent.server: Removing LAN server: server="C02YM00WJG5H.local (Addr: tcp/127.0.0.1:8300) (DC: dc1)"
	    2020-08-07T14:15:22.446+0800 [WARN]  agent.server: deregistering self should be done by follower: name=C02YM00WJG5H.local
	    2020-08-07T14:15:23.665+0800 [ERROR] agent.server.autopilot: Error updating cluster health: error="error getting server raft protocol versions: No servers found"
	    2020-08-07T14:15:25.447+0800 [INFO]  agent.server: Waiting to drain RPC traffic: drain_time=5s
	    2020-08-07T14:15:25.666+0800 [ERROR] agent.server.autopilot: Error updating cluster health: error="error getting server raft protocol versions: No servers found"
	    2020-08-07T14:15:27.666+0800 [ERROR] agent.server.autopilot: Error updating cluster health: error="error getting server raft protocol versions: No servers found"
	    2020-08-07T14:15:29.666+0800 [ERROR] agent.server.autopilot: Error updating cluster health: error="error getting server raft protocol versions: No servers found"
	    2020-08-07T14:15:29.666+0800 [ERROR] agent.server.autopilot: Error promoting servers: error="error getting server raft protocol versions: No servers found"
	    2020-08-07T14:15:30.447+0800 [INFO]  agent: Requesting shutdown
	    2020-08-07T14:15:30.447+0800 [INFO]  agent.server: shutting down server
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopping routine: routine="federation state anti-entropy"
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopping routine: routine="federation state pruning"
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopping routine: routine="CA root pruning"
	    2020-08-07T14:15:30.447+0800 [ERROR] agent.server: error performing anti-entropy sync of federation state: error="context canceled"
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopped routine: routine="federation state pruning"
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopped routine: routine="CA root pruning"
	    2020-08-07T14:15:30.447+0800 [DEBUG] agent.leader: stopped routine: routine="federation state anti-entropy"
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: consul server down
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: shutdown complete
	    2020-08-07T14:15:30.448+0800 [DEBUG] agent.http: Request finished: method=PUT url=/v1/agent/leave from=127.0.0.1:59821 latency=11.002045748s
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Stopping server: protocol=DNS address=127.0.0.1:8600 network=tcp
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Stopping server: protocol=DNS address=127.0.0.1:8600 network=udp
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Stopping server: protocol=HTTP address=127.0.0.1:8500 network=tcp
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Waiting for endpoints to shut down
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Endpoints down
	    2020-08-07T14:15:30.448+0800 [INFO]  agent: Exit code: code=0

当发出`leave`命令的时候，Consul会通知其他成员该agent离开了数据中心。 agent离开后，在同一节点上运行的本地服务以及对其的检查将从目录中删除，并且Consul不会尝试再次与该节点联系。

如果强制终止agent进程则会向Consul数据中心中的其他agent指示该节点发生故障而不是正常离开。 当节点发生故障时，其运行状况将标记为“critical”，但不会从目录中删除。 Consul将自动尝试重新连接到发生故障的节点，前提是该节点由于网络原因可能暂时不可用，并且可能会恢复回来。

如果agent程序正在以Server模式运行，那么优雅离开就变得很重要，这样可以避免导致潜在的可用性中断进而而影响一致性（https://www.consul.io/docs/internals/consensus.html）。 查看“添加和删除服务（https://learn.hashicorp.com/consul/day-2-operations/servers）”教程，以获取有关如何安全添加和删除Server的详细信息。

#### 下一步

现在，你已经在开发模式下学习了启动和停止了Consul agent，请继续下一个教程，通过Consul 服务发现注册服务，你将在其中学习Consul是如何知道某服务在数据中心中是否存在及其所在位置的。
### 四 通过Consul服务发现注册一个Service
在上一章“**运行Consul agent**”教程中，你启动了本地Consul agent，并检查了数据中心的其他成员。 在本教程中，你将使用Consul来注册服务和执行健康检查。

Consul的主要使用场景之一是服务发现。 Consul提供了一个DNS接口，下游服务可以使用该接口查找其上游依赖项的IP地址。

Consul知道这些服务的位置，因为每个服务都向其本地Consul client注册。 操作者可以手动注册服务，配置管理工具可以在部署时注册服务，或者容器编排平台可以通过集成自动注册服务。

在本教程中，你将通过给Consul提供配置文件来手动注册服务和执行健康检查，并使用DNS接口和HTTP API查找服务信息。 手动注册服务将帮助你了解自动化工具最终需要提供给Consul哪些信息才能进行服务发现。

#### 服务定义
你可以通过提供服务定义（这是注册服务的最常用方法）或通过调用HTTP API来注册服务。 在这里，你将使用服务定义。

首先，为Consul配置创建目录。 Consul将所有配置文件加载到配置目录中，因此在Unix系统上，一个通用约定是将目录命名为**/etc/consul.d**（.d后缀表示“此目录包含一组配置文件”）。

	$ mkdir ./consul.d

接下来，编写服务定义配置文件。 假设有一个名为“`web`”的服务运行在80端口上。使用以下命令在配置目录中创建一个名为web.json的文件。 该文件将包含服务的一些属性定义：名称，端口和一个可选的tags，你可以在后续使用它们来查找服务。 （请复制下面除`$`之外的整个代码块，执行命令并创建文件。）

	$ echo '{
	  "service": {
	    "name": "web",
	    "tags": [
	      "rails"
	    ],
	    "port": 80
	  }
	}' > ./consul.d/web.json

现在，使用配置文件重启agent并在agent上启用脚本检查。

>安全警告：在某些配置中启用脚本检查可能会引入一个远程执行漏洞，众所周知该这种漏洞是恶意软件的紧盯的目标。 在生产环境中，我们强烈建议改为`-enable-local-script-checks`。

	$ consul agent -dev -enable-script-checks -config-dir=./consul.d
	==> Starting Consul agent...
	           Version: '1.8.1'
	           Node ID: '78b4e800-e24a-689e-2491-8fb091a3de24'
	         Node name: 'C02YM00WJG5H.local'
	        Datacenter: 'dc1' (Segment: '<all>')
	            Server: true (Bootstrap: false)
	       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
	      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
	           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false, Auto-Encrypt-TLS: false
	
	==> Log data will now stream in as it occurs:
	
	    2020-08-07T14:46:37.010+0800 [DEBUG] agent: Using random ID as node ID: id=78b4e800-e24a-689e-2491-8fb091a3de24
	    2020-08-07T14:46:37.017+0800 [ERROR] agent: [SECURITY] issue: error="using enable-script-checks without ACLs and without allow_write_http_from is DANGEROUS, use enable-local-script-checks instead, see https://www.hashicorp.com/blog/protecting-consul-from-rce-risk-in-specific-configurations/"
	    2020-08-07T14:46:37.017+0800 [WARN]  agent: Node name will not be discoverable via DNS due to invalid characters. Valid characters include all alpha-numerics and dashes.: node_name=C02YM00WJG5H.local
	    2020-08-07T14:46:37.023+0800 [INFO]  agent.server.raft: initial configuration: index=1 servers="[{Suffrage:Voter ID:78b4e800-e24a-689e-2491-8fb091a3de24 Address:127.0.0.1:8300}]"
	    2020-08-07T14:46:37.024+0800 [INFO]  agent.server.raft: entering follower state: follower="Node at 127.0.0.1:8300 [Follower]" leader=
	    2020-08-07T14:46:37.025+0800 [INFO]  agent.server.serf.wan: serf: EventMemberJoin: C02YM00WJG5H.local.dc1 127.0.0.1
	    2020-08-07T14:46:37.025+0800 [INFO]  agent.server.serf.lan: serf: EventMemberJoin: C02YM00WJG5H.local 127.0.0.1
	    2020-08-07T14:46:37.026+0800 [INFO]  agent.server: Handled event for server in area: event=member-join server=C02YM00WJG5H.local.dc1 area=wan
	    2020-08-07T14:46:37.026+0800 [INFO]  agent.server: Adding LAN server: server="C02YM00WJG5H.local (Addr: tcp/127.0.0.1:8300) (DC: dc1)"
	    2020-08-07T14:46:37.028+0800 [INFO]  agent: Started DNS server: address=127.0.0.1:8600 network=tcp
	    2020-08-07T14:46:37.029+0800 [INFO]  agent: Started DNS server: address=127.0.0.1:8600 network=udp
	    2020-08-07T14:46:37.029+0800 [INFO]  agent: Started HTTP server: address=127.0.0.1:8500 network=tcp
	    2020-08-07T14:46:37.029+0800 [INFO]  agent: Started gRPC server: address=127.0.0.1:8502 network=tcp
	    2020-08-07T14:46:37.030+0800 [INFO]  agent: started state syncer
	==> Consul agent running!
	    2020-08-07T14:46:37.078+0800 [WARN]  agent.server.raft: heartbeat timeout reached, starting election: last-leader=
	    2020-08-07T14:46:37.078+0800 [INFO]  agent.server.raft: entering candidate state: node="Node at 127.0.0.1:8300 [Candidate]" term=2
	    2020-08-07T14:46:37.078+0800 [DEBUG] agent.server.raft: votes: needed=1
	    2020-08-07T14:46:37.078+0800 [DEBUG] agent.server.raft: vote granted: from=78b4e800-e24a-689e-2491-8fb091a3de24 term=2 tally=1
	    2020-08-07T14:46:37.078+0800 [INFO]  agent.server.raft: election won: tally=1
	    2020-08-07T14:46:37.078+0800 [INFO]  agent.server.raft: entering leader state: leader="Node at 127.0.0.1:8300 [Leader]"
	    2020-08-07T14:46:37.078+0800 [INFO]  agent.server: cluster leadership acquired
	    2020-08-07T14:46:37.079+0800 [INFO]  agent.server: New leader elected: payload=C02YM00WJG5H.local
	    2020-08-07T14:46:37.079+0800 [DEBUG] agent.server: Cannot upgrade to new ACLs: leaderMode=0 mode=0 found=true leader=127.0.0.1:8300
	    2020-08-07T14:46:37.082+0800 [DEBUG] connect.ca.consul: consul CA provider configured: id=07:80:c8:de:f6:41:86:29:8f:9c:b8:17:d6:48:c2:d5:c5:5c:7f:0c:03:f7:cf:97:5a:a7:c1:68:aa:23:ae:81 is_primary=true
	    2020-08-07T14:46:37.096+0800 [INFO]  agent.server.connect: initialized primary datacenter CA with provider: provider=consul
	    2020-08-07T14:46:37.096+0800 [INFO]  agent.leader: started routine: routine="federation state anti-entropy"
	    2020-08-07T14:46:37.096+0800 [INFO]  agent.leader: started routine: routine="federation state pruning"
	    2020-08-07T14:46:37.096+0800 [INFO]  agent.leader: started routine: routine="CA root pruning"
	    2020-08-07T14:46:37.097+0800 [INFO]  agent.server: federation state anti-entropy synced
	    2020-08-07T14:46:37.097+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local
	    2020-08-07T14:46:37.097+0800 [INFO]  agent.server: member joined, marking health alive: member=C02YM00WJG5H.local
	    2020-08-07T14:46:37.119+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T14:46:37.119+0800 [INFO]  agent: Synced node info
	    2020-08-07T14:46:37.120+0800 [INFO]  agent: Synced service: service=web
	    2020-08-07T14:46:37.120+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T14:46:37.120+0800 [DEBUG] agent: Service in sync: service=web
	    2020-08-07T14:46:37.720+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T14:46:37.720+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T14:46:37.720+0800 [DEBUG] agent: Service in sync: service=web

你可以在输出的日志流中注意到Consul“同步”了`web`服务。 这意味着agent已从配置文件加载了服务定义，并已成功将其注册到服务目录中。

> 注意：在此示例中，我们从未真正启动过`web`服务。 Consul可以注册尚未运行的服务。 它基于服务的端口将每个正在运行的服务与其注册并关联。

在多agent的Consul数据中心中，每个服务都将向其本地Consul client注册，并且Consul client会将注册转发给Consul Server agent，后者维护服务目录。

如果要注册多个服务，则可以在Consul配置目录中创建多个服务定义文件。

#### 查询服务
一旦agent将服务加入到Consul的服务目录后，你可以使用DNS接口或HTTP API对其进行查询。

##### DNS接口
这里首先使用Consul的DNS接口查询前面注册的名为“`web`”的服务。 在Consul中注册的服务的DNS名称为`NAME.service.consul`，其中`NAME`是你用于注册服务的名称（在本例中为`web`）。 默认情况下，所有DNS名称都在consul命名空间中，尽管这是可配置的。

web服务的标准域名为`web.service.consul`。 可以使用下面的指令通过DNS接口（Consul的DNS接口默认在**8600**端口上运行）查询注册的服务。

	$ dig @127.0.0.1 -p 8600 web.service.consul
	
	; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 web.service.consul
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13887
	;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
	;; WARNING: recursion requested but not available
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;web.service.consul.                IN        A
	
	;; ANSWER SECTION:
	web.service.consul.        0        IN        A        127.0.0.1
	
	;; Query time: 0 msec
	;; SERVER: 127.0.0.1#8600(127.0.0.1)
	;; WHEN: Fri Aug 07 16:05:39 CST 2020
	;; MSG SIZE  rcvd: 63

上面有一个`QUESTION SECTION`以及一个 `ANSWER SECTION`。

可以通过输出进行验证，上述输出返回了一条`A`记录，其中包含注册该服务的IP地址。 `A`记录只能保存IP地址。

> 提示：由于我们是以最小的配置启动的consul，因此`A`记录将返回本地主机（127.0.0.1）。 如果你需要播发对数据中心中其他节点有意义的IP地址，请在服务定义中设置 `-advertise` 参数或`address`字段。

你还可以使用DNS接口检索整个ip/port对，作为`SRV`记录。

	$ dig @127.0.0.1 -p 8600 web.service.consul SRV
	
	; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 web.service.consul SRV
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36210
	;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 3
	;; WARNING: recursion requested but not available
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;web.service.consul.                IN        SRV
	
	;; ANSWER SECTION:
	web.service.consul.        0        IN        SRV        1 1 80 C02YM00WJG5H.local.node.dc1.consul.
	
	;; ADDITIONAL SECTION:
	C02YM00WJG5H.local.node.dc1.consul. 0 IN A        127.0.0.1
	C02YM00WJG5H.local.node.dc1.consul. 0 IN TXT        "consul-network-segment="
	
	;; Query time: 0 msec
	;; SERVER: 127.0.0.1#8600(127.0.0.1)
	;; WHEN: Fri Aug 07 16:11:55 CST 2020
	;; MSG SIZE  rcvd: 153

`SRV` 记录表明`web`服务运行在80端口，并且在`C02YM00WJG5H.local.node.dc1.consul`这个节点上。同时还有DNS返回的`A`记录作为附加部分返回。

最后，你还可以使用DNS接口按`tag`过滤服务。 基于tag的服务查询的格式为`TAG.NAME.service.consul`。 在下面的示例中，你将要求Consul提供所有带有“`rails`” tag的`web`服务。 由于你使用该tag注册了名为`web`的服务，因此你将获得成功的响应。

	$ dig @127.0.0.1 -p 8600 rails.web.service.consul
	
	; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 rails.web.service.consul
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13561
	;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
	;; WARNING: recursion requested but not available
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;rails.web.service.consul.        IN        A
	
	;; ANSWER SECTION:
	rails.web.service.consul. 0        IN        A        127.0.0.1
	
	;; Query time: 0 msec
	;; SERVER: 127.0.0.1#8600(127.0.0.1)
	;; WHEN: Fri Aug 07 16:18:40 CST 2020
	;; MSG SIZE  rcvd: 69

##### HTTP 接口
除了使用DNS接口外，还可以使用HTTP接口（Consul的HTTP接口默认在**8500**端口上运行）来查询服务。

	$ curl http://localhost:8500/v1/catalog/service/web
	
	[
	    {
	        "ID": "78b4e800-e24a-689e-2491-8fb091a3de24",
	        "Node": "C02YM00WJG5H.local",
	        "Address": "127.0.0.1",
	        "Datacenter": "dc1",
	        "TaggedAddresses": {
	            "lan": "127.0.0.1",
	            "lan_ipv4": "127.0.0.1",
	            "wan": "127.0.0.1",
	            "wan_ipv4": "127.0.0.1"
	        },
	        "NodeMeta": {
	            "consul-network-segment": ""
	        },
	        "ServiceKind": "",
	        "ServiceID": "web",
	        "ServiceName": "web",
	        "ServiceTags": [
	            "rails"
	        ],
	        "ServiceAddress": "",
	        "ServiceWeights": {
	            "Passing": 1,
	            "Warning": 1
	        },
	        "ServiceMeta": {},
	        "ServicePort": 80,
	        "ServiceEnableTagOverride": false,
	        "ServiceProxy": {
	            "MeshGateway": {},
	            "Expose": {}
	        },
	        "ServiceConnect": {},
	        "CreateIndex": 13,
	        "ModifyIndex": 13
	    }
	]

HTTP API列出了托管给定服务的所有节点。 正如你稍后将在讨论健康检查时所讲的那样，你通常只希望仅对运行状况良好的服务实例过滤查询，而DNS接口会在后台自动进行过滤。 可以通过下面带过滤参数的HTTP 接口来查询仅正常的实例。

	$ curl 'http://localhost:8500/v1/health/service/web?passing'
	
	[
	    {
	        "Node": {
	            "ID": "78b4e800-e24a-689e-2491-8fb091a3de24",
	            "Node": "C02YM00WJG5H.local",
	            "Address": "127.0.0.1",
	            "Datacenter": "dc1",
	            "TaggedAddresses": {
	                "lan": "127.0.0.1",
	                "lan_ipv4": "127.0.0.1",
	                "wan": "127.0.0.1",
	                "wan_ipv4": "127.0.0.1"
	            },
	            "Meta": {
	                "consul-network-segment": ""
	            },
	            "CreateIndex": 11,
	            "ModifyIndex": 12
	        },
	        "Service": {
	            "ID": "web",
	            "Service": "web",
	            "Tags": [
	                "rails"
	            ],
	            "Address": "",
	            "Meta": null,
	            "Port": 80,
	            "Weights": {
	                "Passing": 1,
	                "Warning": 1
	            },
	            "EnableTagOverride": false,
	            "Proxy": {
	                "MeshGateway": {},
	                "Expose": {}
	            },
	            "Connect": {},
	            "CreateIndex": 13,
	            "ModifyIndex": 13
	        },
	        "Checks": [
	            {
	                "Node": "C02YM00WJG5H.local",
	                "CheckID": "serfHealth",
	                "Name": "Serf Health Status",
	                "Status": "passing",
	                "Notes": "",
	                "Output": "Agent alive and reachable",
	                "ServiceID": "",
	                "ServiceName": "",
	                "ServiceTags": [],
	                "Type": "",
	                "Definition": {},
	                "CreateIndex": 11,
	                "ModifyIndex": 11
	            }
	        ]
	    }
	]

#### 更新服务
接下来，你将通过注册健康检查来更新这个名为`web`的服务。 请记住，因为你从未在80端口上启动注册的`web`服务，所以注册的健康检查将失败。

你可以通过更改服务定义文件并发送SIGHUP到agent或运行`consul reload`来更新服务定义，这样不会造成任何停机。 或者，你可以使用HTTP API动态添加，删除和修改服务。 在此示例中，我们将通过更新注册文件的方式。

首先，运行如下命令编辑文件，复制粘贴整个代码块(除`$`外)到你的终端。

	$ echo '{
	  "service": {
	    "name": "web",
	    "tags": [
	      "rails"
	    ],
	    "port": 80,
	    "check": {
	      "args": [
	        "curl",
	        "localhost"
	      ],
	      "interval": "10s"
	    }
	  }
	}' > ./consul.d/web.json



服务定义中的“`check`”字段添加了一个基于脚本的运行状况检查，该检查尝试每10秒通过curl连接到web服务。 基于脚本的健康检查将以与启动Consul流程相同的用户身份运行。

如果命令以`code> = 2`退出，则检查将失败，Consul将认为服务不健康。 而以`code = 1`退出则将被视为警告状态。

现在，重新加载Consul的配置，使其运行健康检查。

	$ consul reload
	Configuration reload triggered
	
请注意Consul日志中的以下几行，它们表明这个名为`web`的服务的检查结果为不健康。
	
	2020-08-07T16:46:35.373+0800 [INFO]  agent: Synced service: service=web
	    2020-08-07T16:46:35.373+0800 [DEBUG] agent: Check in sync: check=service:web
	    2020-08-07T16:46:42.170+0800 [WARN]  agent: Check is now critical: check=service:web
	    2020-08-07T16:46:49.121+0800 [DEBUG] agent: Skipping remote check since it is managed automatically: check=serfHealth
	    2020-08-07T16:46:49.121+0800 [DEBUG] agent: Node info in sync
	    2020-08-07T16:46:49.121+0800 [DEBUG] agent: Service in sync: service=web
	    2020-08-07T16:46:49.121+0800 [DEBUG] agent: Check in sync: check=service:web
	    2020-08-07T16:46:52.194+0800 [WARN]  agent: Check is now critical: check=service:web
	    2020-08-07T16:46:59.241+0800 [DEBUG] agent.server: Skipping self join check for node since the cluster is too small: node=C02YM00WJG5H.local
	    2020-08-07T16:47:02.221+0800 [WARN]  agent: Check is now critical: check=service:web
	    2020-08-07T16:47:12.248+0800 [WARN]  agent: Check is now critical: check=service:web

Consul的DNS服务只会返回健康的结果，使用DNS接口再次查询`web`服务，它将不会返回任何IP，因为`web`服务的健康检查失败了。

	$ dig @127.0.0.1 -p 8600 web.service.consul
	
	; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8600 web.service.consul
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 51197
	;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
	;; WARNING: recursion requested but not available
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;web.service.consul.                IN        A
	
	;; AUTHORITY SECTION:
	consul.                        0        IN        SOA        ns.consul. hostmaster.consul. 1596790201 3600 600 86400 0
	
	;; Query time: 0 msec
	;; SERVER: 127.0.0.1#8600(127.0.0.1)
	;; WHEN: Fri Aug 07 16:50:01 CST 2020
	;; MSG SIZE  rcvd: 97

注意到已经没有`ANSWER SECTION`了，因为Consul已经将这个名为`web`的服务标记为不健康了。

#### 下一步

在本教程中，你通过Consul注册了服务，并学习了如何使用HTTP 接口和DNS接口查询该服务。 你还为该服务添加了基于脚本的健康检查。 你可以在API文档（https://www.consul.io/api/agent/service.html）中找到有关服务注册的完整列表，或在健康检查（https://www.consul.io/docs/agent/checks.html）文档中了解有关健康检查的更多信息。

继续阅读《**使用Consul Service Mesh连接服务**》章节，以学习如何启用Consul的Service Mesh——Consul Connect，它可以保护和管控服务之间的网络流量，允许或拒绝服务间通信。

### 五 通过Consul Service Mesh连接服务

除了使用DNS接口或HTTP 接口直接提供服务的IP地址外，Consul还可以通过随每个服务实例在本地部署的Sidecar Proxy将服务彼此连接。 这种类型的部署（控制服务实例之间网络流量的本地代理）就是Service Mesh(服务网格)。 由于Sidecar Proxy可连接你的注册服务，因此在文档中有时将Consul的Service Mesh(服务网格)功能称为Consul Connect。 该术语是该功能的旧名称，在我们更新教程和文档时将不再使用该术语。
Consul Service Mesh(服务网格)使你可以在不修改服务代码的情况下保护和监控服务之间的通信。 Consul通过配置sidecar proxy以在你的服务之间建立mutual TLS（mTLS，双向TLS ），并根据其注册名称允许或拒绝它们之间的通信。 由于Sidecar Proxy控制着所有服务到服务的流量，因此它们可以收集有关它们的指标，并将其导出到Prometheus等第三方聚合器。

你还可以将应用程序与Consul Connect进行本地集成(https://www.consul.io/docs/connect/native.html)，以实现最佳性能和安全性。

使用Service Mesh(服务网格)注册服务与正常注册服务相似。 在本教程中，你将：

- 启动一个服务。
- 按常规注册它，但要附加一个connect字段。
- 再注册一个proxy以与服务进行通信。
- 启动Sidecar Proxy。
- 练习阻止与服务的连接。

#### 准备条件
##### 本地环境
本教程的唯一要求就是在本地环境中有安装Consul二进制文件。

> 安全警告：为简化起见，本教程以开发模式演示了Consul Connect Service Mesh(服务网格)功能，这不是在生产环境中推荐的部署Consul的安全方法。 请阅读Connect生产环境教程，以了解有关安全部署Consul Connect Service Mesh(服务网格)的信息。

#### 互动教程
我们还提供了涵盖本教程相同概念的交互式教程。 如果你不想在本地设置演示环境，请单击“显示教程”按钮以启动演示环境。

> 注意：交互式教程并未使用与本教程中列出的完全一致的指令。 你可以将交互式教程这个实验平台视为扩展本教程中介绍的概念的一种方法。

#### 启动一个对Consul无感知的服务

首先启动一个对Consul无感知的服务。 这里你将使用socat指令启动一个最基础的echo服务，该服务将在本教程中充当上游服务的角色。 在生产环境中，该服务可能是数据库，后端或其他服务依赖的任何服务。

Socat是一个具有数十年历史的Unix实用程序，它没有加密或TLS协议的概念。 你将使用它来证明Consul Connect Service Mesh(服务网格)为你解决了这些问题。 如果你的计算机上未安装socat，可以通过软件包管理器来安装使用。比如MacOS下可以：

	brew install socat

启动socat服务，并指定它侦听8181端口上的TCP连接：

	$ socat -v tcp-l:8181,fork exec:"/bin/cat"
	
	(base) c02ym00wjg5h:~ linyu$ socat -v tcp-l:8181,fork exec:"/bin/cat"
	> 2020/08/08 17:03:34.377599  length=6 from=0 to=5
	hello
	< 2020/08/08 17:03:34.378130  length=6 from=0 to=5
	hello
	> 2020/08/08 17:03:43.203669  length=12 from=6 to=17
	testing 123
	< 2020/08/08 17:03:43.204088  length=12 from=6 to=17
	testing 123

你可以直接通过`nc`（netcat）命令在对应的端口上连接到echo服务来验证它是否正常工作。 连接后，输入一些文本，然后按Enter。 你输入的文本将会回显在控制台：

	(base) C02YM00WJG5H:consul-learn linyu$ nc 127.0.0.1 8181
	hello
	hello
	testing 123
	testing 123

#### 注册该服务并通过Consul来代理它
接下来，就像在教程的前一个部分所做的那样，我们通过编写一个新的服务定义向Consul注册该服务。 这次，你的服务定义中将包含`connect`字段，该字段将在后端服务实例边上注册sidecar proxy， 用来处理此后端服务实例的流量。

添加一个叫做**socat.json**的文件到**consul.d**目录，文件中的内容如下（可以拷贝并粘贴除`$`以外的内容）:

	$ echo '{
	  "service": {
	    "name": "socat",
	    "port": 8181,
	    "connect": {
	      "sidecar_service": {}
	    }
	  }
	}' > ./consul.d/socat.json

现在运行`consul reload`或者发送一个SIGHUP信号给Consul从而使其读取新的服务定义文件。

可以看看刚刚添加的服务定义中的“`connect`”字段。 该空配置会通知Consul在动态分配的端口上为此进程注册一个Sidecar Proxy。 当你通过命令行启动时，它还会使用合理的默认值，Consul将使用该默认值来配置Proxy。 Consul不会自动为你启动 proxy 进程。 这是因为Consul Connect Service Mesh(服务网格)允许你选择要使用的proxy。

Consul原生拥有用于测试目的的L4 proxy，和对Envoy的一流支持——你应该将其用于生产环境部署，以及7层(L7)流量管理。 在本教程中，你将使用L4 proxy，因为与Envoy不同的是，它是Consul自带的，不需要任何额外的安装。

使用`consul connect proxy`命令在另一个终端窗口中启动proxy进程，并指定其对应的服务实例。

	$ consul connect proxy -sidecar-for socat
	
	(base) c02ym00wjg5h:~ linyu$ consul connect proxy -sidecar-for socat
	==> Consul Connect proxy starting...
	    Configuration mode: Agent API
	        Sidecar for ID: socat
	              Proxy ID: socat-sidecar-proxy
	
	==> Log data will now stream in as it occurs:
	
	    2020-08-08T17:08:19.089+0800 [INFO]  proxy: Proxy loaded config and ready to serve
	    2020-08-08T17:08:19.089+0800 [INFO]  proxy: Parsed TLS identity: uri=spiffe://f85d5c94-24e5-c805-3681-b87674570927.consul/ns/default/dc/dc1/svc/socat roots=[pri-12c9ah6t.consul.ca.f85d5c94.consul]
	    2020-08-08T17:08:19.089+0800 [INFO]  proxy: Starting listener: listener="public listener" bind_addr=0.0.0.0:21000


#### 注册一个依赖服务和proxy
接下来，注册一个称为“`web`”的下游服务。 类似于`socat`服务定义，用于web服务的配置文件将包含一个指定sidecar的`connect`字段，但是与socat定义不同的是，此处的配置不是空的。 相反，它指定`web`对`socat`服务的上游依赖性，以及proxy将侦听的端口。

	$ echo '{
	  "service": {
	    "name": "web",
	    "connect": {
	      "sidecar_service": {
	        "proxy": {
	          "upstreams": [
	            {
	              "destination_name": "socat",
	              "local_bind_port": 9191
	            }
	          ]
	        }
	      }
	    }
	  }
	}' > ./consul.d/web.json

同样，使用`consul reload` 或 SIGHUP 来重启Consul。这会为`web` 服务注册一个sidebar proxy。 该proxy将监听9191端口并建立到`socat`的mTLS(双向 TLS, **m**utual **T**ransport **L**ayer **S**ecurity)链接。

>**mTLS**(mutual Transport Layer Security)
>
>所有的请求和回复，都会进行双向TLS认证。在加密的情形下，请求与回复的内容无法被网络嗅探。

如果我们运行的是真正的Web服务，它将在回环地址上与其proxy进行通信。 proxy将对其流量进行加密，并通过网络将其发送到socat服务的sidecar proxy。 `socat`的sidecar proxy将解密流量并将其通过本地8181端口上的回环地址发送到`socat`服务。由于没有Web服务在运行，因此你将通过在我们指定的（9191）端口上与它的proxy进行通信来假装为真实的Web服务 。

在启动proxy进程之前，请确认你无法在9191端口上连接到`socat`服务。如果运行以下命令应立即退出，因为在9191端口上`socat`没有侦听任何内容（`socat`在8181上侦听）。

	$ nc 127.0.0.1 9191

现在使用sidecar注册的配置文件启动proxy

	$ consul connect proxy -sidecar-for web
	
	==> Consul Connect proxy starting...
	    Configuration mode: Agent API
	        Sidecar for ID: web
	              Proxy ID: web-sidecar-proxy
	
	==> Log data will now stream in as it occurs:
	
	    2019/07/24 13:32:10 [INFO] 127.0.0.1:9191->service:default/socat starting on 127.0.0.1:9191
	    2019/07/24 13:32:10 [INFO] Proxy loaded config and ready to serve
	    2019/07/24 13:32:10 [INFO] TLS Identity: spiffe://287133f6-3d1e-8fb0-a0c5-fb9d5a95d53c.consul/ns/default/dc/dc1/svc/web
	    2019/07/24 13:32:10 [INFO] TLS Roots   : [Consul CA 7]
	    2019/07/24 13:32:10 [INFO] public listener starting on 0.0.0.0:21001



>注意，在第一行日志中，proxy在端口9191上设置了本地侦听器，其会将请求代理到`socat`服务，就像我们在sidecar注册中配置的那样。 随后的日志行列出了从proxy加载的证书的身份URL（将其标识为“ `web`”服务）以及代理知晓的一组受信任的根证书。

尝试再次在端口9191上连接至`socat`。这一次它应该可以工作并回显你发送的文本。

	$ nc 127.0.0.1 9191
	
	hello
	hello

可以使用Crl+c关闭链接。

web proxy和socat proxy之间的通信通过mTLS连接进行加密和授权，而每个服务与其Sidecar proxy之间的通信未加密。 在生产环境中，服务应当仅接受回环连接。 机器进出的任何流量都应通过proxy来完成，因此始连接终会被加密。

>安全警告：Consul Connect Service Mesh安全模型在使用proxy时需要信任回环连接。 你可以使用网络命名空间之类的工具进一步保障回环连接的安全性。

#### 使用intentions指令来控制通信
`intention`定义允许哪些服务与其他服务进行通信。 上面的连接可以成功是因为在开发模式下，ACL系统默认情况下（以及默认的intention策略）为“全部允许”。

创建`intention`用于阻断从`web`服务到`socat`服务的访问，该指令指定了`intention`的策略、源服务和目标服务。

	$ consul intention create -deny web socat
	
	Created: web => socat (deny)

现在，再次尝试连接，将会出现失败：

	$ nc 127.0.0.1 9191

#### 删除intention：

	$ consul intention delete web socat
	Intention deleted.

再次尝试连接将会成功

	$ nc 127.0.0.1 9191
	
	hello
	hello

`intention`使你可以像传统防火墙一样对网络进行分隔，但是它们依赖于服务的逻辑名称（例如“ `web`”或“`socat`”），而不是每个服务实例的IP地址。 在文档中了解有关`intention` （https://www.consul.io/docs/connect/intentions.html）的更多信息。

> 注意：更改intention不会影响与Consul当前的现有连接。 你必须建立新的连接才能查看intention改变的影响。

#### 下一步
在本教程中，你在单个agent上配置了服务，并使用Consul Connect Service Mesh进行自动连接授权和加密。 想了解Consul Connect Service Mesh的其他功能，可查看Getting Started with Consul Service Mesh（https://learn.hashicorp.com/consul?track=gs-consul-service-mesh#gs-consul-service-mesh）。

接下来，将带你按照《**在Consul KV中存储数据**》的教程，探索如何使用Consul的键值（KV）进行服务配置存储。
### 六  在Consul KV中存储数据

除了提供服务发现、集成健康检查以及安全的网络链路外，Consul还包含kv存储，你可以将其用于动态地配置应用程序，协调服务，管理leader选举或充当Vault的数据后端，以及数不胜数的其他用途。

在本教程中，你将使用命令行探索Consul键值存储（Consul KV）。 本教程假定“启动Consul Agent”教程中的Consul agent仍在运行。 如果没有，你可以通过运行`consoul proxy -dev`启动一个新的agent。 Consul KV在Consul agent上自动开启； 你无需在Consul的配置中启用它。

有2种方式可以与Consul KV存储进行交互：HTTP API和命令行（CLI）。 在本教程中，你将使用命令行（CLI）。 请参阅HTTP API文档（https://www.consul.io/api/kv.html），以了解服务如何通过HTTP接口与Consul KV进行交互。

#### 添加数据
首先，使用consul kv的 `put`命令将一些value存入 KV存储中。 `put`命令后面的第一个参数是key，第二个参数是value。

	$ consul kv put redis/config/minconns 1
	Success! Data written to: redis/config/minconns

在上面这里，key 是 redis/config/maxconns ，value 是 25

	$ consul kv put redis/config/maxconns 25
	Success! Data written to: redis/config/maxconns

请注意，使用下面输入的key（“`redis/config/users/admin`”），你将设置一个值为`42`的flag。key支持设置64位整数flag, 该值不是在Consul内部使用，但是客户端可以使用该值将元数据添加到任何KV对中。

	$ consul kv put -flags=42 redis/config/users/admin abcd1234
	Success! Data written to: redis/config/users/admin

#### 查询数据
现在，查询你刚输入的任何一个key

	$ consul kv get redis/config/minconns
	1

Consul保留了一些有关键值对的其他元数据。 使用-detailed命令行标志可以检索一些元数据（包括你设置的flag）。

	$ consul kv get -detailed redis/config/users/admin
	
	CreateIndex      14
	Flags            42
	Key              redis/config/users/admin
	LockIndex        0
	ModifyIndex      14
	Session          -
	Value            abcd1234

可使用`recurse`选项列出存储中的所有key。 其结果将按字典顺序返回。

	$ consul kv get -recurse
	
	redis/config/maxconns:25
	redis/config/minconns:1
	redis/config/users/admin:abcd1234

#### 删除数据

使用`delete`指令从Consul KV存储中删除数据：

	$ consul kv delete redis/config/minconns
	Success! Deleted key: redis/config/minconns

Consul允许你以类似于文件夹的方式与key进行交互。 尽管KV存储中的所有key实际上都是扁平化存储的，但Consul允许你将具有相同前缀的key作为一组进行操作，就像它们在文件夹或子文件夹中。

可以使用`recurse`指令删除所有带`redis`前缀的key

	$ consul kv delete -recurse redis
	Success! Deleted keys with prefix: redis

#### 修改数据
更新某个key的value

	$ consul kv put foo bar
	Success! Data written to: foo

查询更新后的key

	$ consul kv get foo
	bar

#### 下一步
在本教程中，你在Consul的KV存储中添加，查看，修改和删除了一些键值对。

Consul可以使用Check-And-Set（CAS）操作执行原子键更新，并且包含写入键和值的其他复杂选项。 你可以在consul kv put命令的帮助页面上浏览这些选项。

	$ consul kv put -h

这些只是API支持的几个示例。 有关完整的文档，请参阅HTTP API文档（https://www.consul.io/api/kv.html）和CLI文档(https://www.consul.io/docs/commands/kv.html)。

现在你已经知道了Consul包含一些有趣的关于服务注册，键，值和intention的数据，请继续浏览**使用Consul Web UI**教程以在Consul Web UI中探索所有这些数据。

### 七 使用Consul Web UI

Consul的Web UI允许你通过图形用户界面查看Consul并与之交互，这可以降低新用户的进入门槛，并简化故障排除。

如果你在生产环境中运行Consul，则需要在Consul的配置文件中启用UI或使用`-ui`命令行标志，但是由于这里你的agent程序在开发模式下运行，因此会自动启用UI。


#### 导航到UI界面
如果你已经把前面教程中启动的consul节点关掉了，你也可以访问Consul Web UI 的这个在线demo来探索本节中的内容。

否则，如果你没有关掉前面启动的consul节点，你则可以更加真切的按照本教程走下去。现在打开你的浏览器，输入**http://localhost:8500/ui**即可以打开Cosnul Web UI 界面。

你将会看到一个顶部是粉色菜单的页面。
#### 查看服务

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNKnASnnTNp8Dc4tvtRoMfdL0YicDjwmmmARAbiaLvFqSSGiatJj3acPHZw/640?wx_fmt=png)

>注：最新版的已经不长上面那样了，而是下面这样👇

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNhsKerticHMmhCGB8E6atbPdZCyBiaRGnvrEogUOB4q0AibDE6QoNlq0Dw/640?wx_fmt=png)

首先进入的落地页是Service页面，该页面将所有注册的服务列了出来，包括他们的服务名，健康状况，tags，类型以及资源。你可以点击某个服务来查看有关其实例个数、每个实例健康状况以及该实例注册到哪个agent了等相关的更多信息。

你可以根据服务的名称，标签，状态或其他搜索条件过滤页面上可见的服务。

> 尝试一下：在搜索栏中输入socat并按Enter键，以筛选出socat服务。

你可以通过单击来了解各个服务的详细信息。

> 尝试一下：单击web、服务以浏览其相关信息。 现在，选择一个列出的web服务实例，以查看实例-实例的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNWI4kf6MVopicouiax5ddSFae1cetpiaibaqchgGsHyzF07GCHAkwYVTXcA/640?wx_fmt=png)

单击instance实例：

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNYvlrCfu3wibxZJMo68zEaVVQT5rJmd8GzCsGY64ypVJSOh3Hic96FTtA/640?wx_fmt=png)

Proxy Infos

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNMNXojicltwv7xLTmEShicJRo1x8dkCOrPMBQp4re5LhN4VCwGXNJaXDA/640?wx_fmt=png)


#### 查看节点
接下来，点击粉红色的顶部导航栏中“Nodes”菜单，跳转到节点页面。 你将在其中找到整个数据中心的概述，包括每个节点的运行状况。 你可以选择各个节点以了解其运行状况检查，注册的服务，往返时间和锁定会话。

你还可以按健康状态过滤节点，或在搜索栏中搜索它们。

> 尝试一下：从顶部菜单栏中选择节点页面，然后单击本地计算机这个节点。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNDoqpj4RgXAzdx4QfcjDgnY0ae0gnNDYpsFbTCSY3TlfGQZgmpOJXaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNFpadD7f7Y2vAo7BC2f5UmnHVpjEsliamIMib2Comjx5tbk99QRWTtPYw/640?wx_fmt=png)


![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNpiatQDnWoxFffFefLorpFryuicicCPFThAl9B2T3MMpMDF3zDUVeFrzIQ/640?wx_fmt=png)


#### 管理KV存储
在顶部导航中，单击“Key/Value”菜单以查看Consul KV的页面。 如果你使用的是与先前教程相同的代理，则应该看到一个key `foo`。

![](https://mmbiz.qlogo.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNKjSicrErkCs2gm0LkibEIDOaS0Fb5ZCeDx8yP6aSSwOChO4aOHk9pPkg/0?wx_fmt=png)

key页面具有类似文件夹的结构。根据其键前缀出现嵌套。 例如，你可以为每个应用程序，业务功能或两者的嵌套组合使用一个文件夹。

> 尝试一下：在主页上，单击蓝色的“Create”按钮以添加新的键值对。 用`Alice`作为key `redis/user`的value。 现在，使用key `redis/password` 和value `123`创建另一个键值对。在主页上，请注意只有一个新条目，称为“redis”，旁边是文件夹图标。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNvrum9vJonJzNibSia6hJ1UPQ00vTUvvOSMeJLslswiagtlibETefDrSXLA/640?wx_fmt=png)

当你单击文件夹时，Consul将自动在该文件夹下嵌套新key，而无需键入前缀。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNhfaGOTnQnyPZicTc8hPXL91Uk81IDfsYia7O5W0BGtqsXGAeZ1U2glhA/640?wx_fmt=png)

#### 管理访问控制列表

Consul使用访问控制列表（ACL）来保护Web UI，API，CLI，服务和sidecar proxy之间通信。 你需要在Consul数据中心中配置ACL以保护它在生产环境中的安全，但是，在开发模式下的代理上没有启用它们，因此目前在“ACL”页面上没有多少内容。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNQZS7HAxlr7Uh68431LKvBB6zXGlqviaRmRAiayibbvB53rAlCETylWHkw/640?wx_fmt=png)


通过限制UI中各个页面的读取，写入和更新权限，可以使用ACL保护UI本身。 为此，你可以创建具有适当权限的令牌，然后将其添加到ACL页面下的UI中。 要删除访问权限，只需从令牌列表中的令牌操作菜单中选择“停止使用”。


#### 管理intention
单击“intention”菜单项以导航到UI中的intention页面。 目前还没有任何intention，但是如果你仍在运行与先前教程相同的agent，则可以通过创建intention来阻止`web`服务与`socat`服务之间的通信。

比如，如果我们按之前的教程，创建一个阻止web到socat的连接：

	(base) c02ym00wjg5h:consul-learn linyu$ consul intention create -deny web socat

那么在intentions菜单下，你将会看到：

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNsZndV1jhrW2MyDdzRpYzlp6DNY8h5fycOYH2TyjibgUv11ZLicV9J7HA/640?wx_fmt=png)


而删除该intention之后，UI界面上也将看不到了：

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNRa6pyItV1eic6D9b8EgptYZ0Q2vyBV0fBmGv48ib2oFpV5j8HtcNfbaA/640?wx_fmt=png)

当然，你也可以使用UI提供的创建、删除、编辑功能完成同样的效果。

#### 调整UI配置

单击菜单栏最右侧的“Settings”。 你可以在此处编辑用户界面的设置。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNeWYXSc354TLwy0kXf18Ks2bWWSxM0o4ouhtG7PaicCwmR8RNSWUzZTg/640?wx_fmt=png)

如果已设置指标仪表板来监视服务，则可以在设置页面上添加一个链接，该链接将自动为该服务名称和数据中心填充占位符，并从其UI页面链接到每个服务的指标。

你还可以选择是否要设置使用阻塞式的查询来实时更新UI，而不是刷新时才更新。 默认情况下启用了此功能，你也可以将其禁用，因为它可能会影响页面性能。

#### UI任务表

你可能已经注意到，Web UI的某些页面是只读的，而其他页面是可交互的。 下表是每个页面的CRUD可操作性的列表。

![](https://mmbiz.qpic.cn/mmbiz_png/XsgEbl9EdmnNjHTMIhVzgVnA38r1y9ibNDgkNv4ZmSOqpoLpZwxqd8oF3S7tHHA6RMoOkmzZGbjjbo2EbgJxd3g/640?wx_fmt=png)

#### 下一步
现在你可以轻松查看和使用Web UI了，请尝试使用Consul 命令行工具完成此处列出的相同任务。

到目前为止，你已经探索了Consul的核心功能，包括服务发现，使用Service Mesh保护服务以及使用KV存储。 继续下一个教程“**创建本地Consul数据中心**”，以了解如何通过将多个Consul agent连接在一起来设置Consul数据中心。

>注意：下一个教程依赖VirtualBox和Vagrant一次在你的计算机上运行多个Consul agent。 如果你还没有下载它们，那么现在下载是个不错的选择。

### 八  创建本地Consul数据中心
既然你已经练习过使用Consul，现在是时候进一步了解Consul的操作方式了。 在本教程中，你将创建你的具有多个成员的第一个数据中心。

当新的Consul agent启动时，它对其他 agent一无所知。 它实际上它是一个只有一个成员的数据中心。 agent通过两种方式互相知道彼此。 要将新agent添加到现有数据中心，需要给它提供数据中心中任意其他agent（Server模式或Client模式）的IP地址，这将导致新agent加入数据中心。 agent成为新数据中心的成员后，它将通过流言协议自动了解其他agent。

在本教程中，你将把两个agent结合在一起以创建一个由两个成员组成的Consul数据中心。

#### 设置环境
Consul是一个分布式应用程序，其设计的是每个机器上只有一个agent。 要在同一台计算机上运行两个agent，你需要安装VirtualBox和Vagrant，它们将运行虚拟机来模拟分布式环境。

创建一个目录来存储本教程中Vagrant的配置文件。

	$ mkdir consul-getting-started-join

在名为Vagrantfile的目录中创建一个新文件，并将Consul的演示Vagrant文件的内容粘贴到其中。 然后保存文件。 该文件将告诉Vagrant在计算机上创建两个预安装Consul二进制文件的虚拟机。

启动你的两个虚拟机。 下载所需的所有内容可能需要一点时间。

	$ vagrant up
	
	Bringing machine 'n1' up with 'virtualbox' provider...
	Bringing machine 'n2' up with 'virtualbox' provider...
	...
	
一旦环境跑起来，SSH到第一个虚拟机中，配置你的第一个数据中心：

	$ vagrant ssh n1
	
	Linux n1 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u2 (2019-05-13) x86_64
	
	The programs included with the Debian GNU/Linux system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.
	
	Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
	permitted by applicable law.
	You have new mail.
	vagrant@n1:~$

#### 启动agent

在之前的教程中，你在开发模式下使用了一个agent来测试Consul的功能。 但是，永远不要在生产环境中运行开发模式的agent。 在本教程中，你将通过以下命令行标志将第一个Consul agent配置为在Server模式下运行。 （在生产中环境中，这些可以通过配置文件提供）

- `-server` 。如果提供了这个标志，表示你希望agent以Server模式启动
- `-bootstrap-expect`。告诉Consul Server数据中心总共应该有多少台Server。所有Server将在启动日志之前等待这些Server加入，从而使所有Server上的数据保持一致。因为这里你正在建立一个单Server数据中心，所以将这个值设置为 1。你可以在启动教程(https://www.consul.io/docs/guides/bootstrapping.html)中了解有关此过程的更多信息。
- `-node name`。数据中心中的每个节点必须具有唯一的名称。默认情况下，Consul使用计算机的主机名，但是这里你将手动覆盖它，并将其设置为agent-one。
- `-bind addres`。 这是该agent将监听来自其他Consul成员的通信的地址。数据中心中的所有其他节点都必须可以访问它。如果未设置绑定地址，Consul将尝试侦听所有IPv4接口，并且如果找到多个私有IP，则启动失败。由于生产环境下的Server通常具有多个接口，因此应始终提供绑定地址。在本教程情况下，它是172.20.20.10，你将其指定为Vagrantfile中第一个VM的地址。
- `-data-dir`。此标志告诉Consul agent他们应将状态存储在哪里，其中可以包括Server和Client的敏感数据（如ACL令牌）。在生产环境部署时，你应注意此目录的权限。可以在该文档(https://www.consul.io/docs/agent/options.html#_data_dir)中查找更多信息。这里你将数据目录设置为标准位置：`/tmp/consul`。
- `-config-dir`。此标志告诉Consul在哪里寻找其配置。你将其设置在标准位置: `/etc/consul.d`。

运行以下命令来启动你的第一个Consul agent。

	# vagrant@n1:~
	$ consul agent \
	  -server \
	  -bootstrap-expect=1 \
	  -node=agent-one \
	  -bind=172.20.20.10 \
	  -data-dir=/tmp/consul \
	  -config-dir=/etc/consul.d
	
	...

打开一个新的终端窗口，并将目录更改为`consul-getting-started-join`。 然后ssh进入第二个虚拟机。

	$ vagrant ssh n2

现在以第Client模式启动你的第二个Consul agent，绑定的IP地址将设置为第二个虚拟机的地址（`172.20.20.11`, Vagrantfile文件中）并且名字为`agent-two`。这里不要使用`-server`标志，这样第二个Consul agent将会以Client模式运行。

	# vagrant@n2:~
	$ consul agent \
	  -node=agent-two \
	  -bind=172.20.20.11 \
	  -enable-script-checks=true \
	  -data-dir=/tmp/consul \
	  -config-dir=/etc/consul.d
	...
	
现在我们已经有了2个Consul agent在运行：一个Client模式，一个Server模式。这2个agent此时此刻人然对彼此一无所知，他们各自组成自己的单节点数据中心。

通过ssh进入每个虚拟机，检查每个agent的成员信息来验证这一点。这里打开一个新的窗口，进入到consul-getting-started-join目录。

检查`agent-two`的成员信息：

	$ vagrant ssh n2
	
	Linux n2 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u2 (2019-05-13) x86_64
	
	The programs included with the Debian GNU/Linux system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.
	
	Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
	permitted by applicable law.
	You have new mail.
	Last login: Fri Aug  2 23:42:33 2019 from 10.0.2.2
	
#

	# vagrant@n2:~
	$ consul members
	
	Node       Address            Status  Type    Build  Protocol  DC   Segment
	agent-two  172.20.20.11:8301  alive   client  1.5.3  2         dc1  <default>


再打开一个新窗口，进入consul-getting-started-join目录。

检查`agent-one`的成员信息：

	$ vagrant ssh n1
	
#
	
	Linux n1 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u2 (2019-05-13) x86_64
	
	The programs included with the Debian GNU/Linux system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.
	
	Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
	permitted by applicable law.
	You have new mail.
	Last login: Fri Aug  2 20:37:46 2019 from 10.0.2.2

#

	# vagrant@n1:~
	$ consul members
	
#
	
	Node       Address            Status  Type    Build  Protocol  DC   Segment
	agent-one  172.20.20.10:8301  alive   server  1.5.3  2         dc1  <all>


#### 加入新的agent
现在你已经可以准备创建多agent的数据中心了。 再次回到第一个VM的终端窗口中，并在Consul Server上运行`consul join`命令，给它提供Consul Client的绑定地址。

	# vagrant@n1:~
	$ consul join 172.20.20.11
	Successfully joined cluster by contacting 1 nodes.

在同样的窗口再次运行`consul members`指令，你将看到会同时列出2个agent:

	# vagrant@n1:~
	$ consul members
#

	Node       Address            Status  Type    Build  Protocol  DC   Segment
	agent-one  172.20.20.10:8301  alive   server  1.5.3  2         dc1  <all>
	agent-two  172.20.20.11:8301  alive   client  1.5.3  2         dc1  <default>

切换到Consul Server在第一个VM上运行的终端窗口，你会注意到一些日志输出，日志显式`agent-two`加入了该窗口。

现在，切换到在第二台VM上运行的Client终端。 你会注意到Client agent一直在发出警告和显示错误，提示没有Server agent可用。 当Client agent感知到Server agent时，它将停止抛出错误并同步其节点信息。

	2019/08/03 00:09:25 [WARN] manager: No servers available
	    2019/08/03 00:09:25 [ERR] agent: failed to sync remote state: No known Consul servers
	    2019/08/03 00:09:54 [WARN] manager: No servers available
	    2019/08/03 00:09:54 [ERR] agent: failed to sync remote state: No known Consul servers
	    2019/08/03 00:10:10 [INFO] serf: EventMemberJoin: agent-one 172.20.20.10
	    2019/08/03 00:10:10 [INFO] consul: adding server agent-one (Addr: tcp/172.20.20.10:8300) (DC: dc1)
	    2019/08/03 00:10:10 [INFO] consul: New leader elected: agent-one
	    2019/08/03 00:10:11 [INFO] agent: Synced node info

没有Consul Server agent，Consul Client agent将无法运行。 所有数据中心必须至少具有一个在Server模式下运行的agent，Consul才能正常运行。

在具有多个Server agent的数据中心中，超过一半的Server agent必须始终保持相互通信，以使数据中心正常运行。 这称为维持仲裁。 你可以在Consul架构文档中了解有关Server agent法定数量的更多信息。

切换到第二个VM的窗口，然后在Client上运行`consul members`指令，你将看到Client同样会同时列出2个agent:

	# vagrant@n2:~
	$ consul members
#

	Node       Address            Status  Type    Build  Protocol
	agent-two  172.20.20.11:8301  alive   client  0.5.0  2
	agent-one  172.20.20.10:8301  alive   server  0.5.0  2

>提示：要加入数据中心，Consul agent只需了解另外一个现有成员，该成员可以是Client agent，也可以是Server agent。 加入数据中心后，agent会自动通过流言协议传播完整的成员信息。

#### 自动加入
在生产环境中，新的Consul agent应自动加入数据中心，而无需人工干预。 你可以通过在Consul配置文件中配置cloud auto join对象，将Consul配置为自动发现AWS，Google Cloud或Azure中的新agent。 这将允许新节点无需任何硬编码配置即可加入数据中心。

或者，你可以使用`-join`标志或`start_join`配置将已知Consul agent的硬编码地址提供给新agent。

#### 查询节点

你可以使用DNS接口或HTTP API查询Consul agent。

对于DNS API，名称的结构为`NAME.node.consul或NAME.node.DATACENTER.consul`。 如果省略`DATACENTER`，Consul将仅搜索本地数据中心。

切换到agent-one的终端窗口中，使用DNS接口中查询`agent-two`的地址。

	# vagrant@n1:~
	$ dig @127.0.0.1 -p 8600 agent-two.node.consul
	...
	
	;; QUESTION SECTION:
	;agent-two.node.consul. IN  A
	
	;; ANSWER SECTION:
	agent-two.node.consul.  0 IN    A   172.20.20.11

除了服务发现之外，查找服务之外的节点的功能对于系统管理也很有用。 例如，想知道要SSH的节点的地址就像使节点成为Consul数据中心的一部分并进行查询一样容易。

#### 停止agent
可以在运行它们的终端窗口中键入Ctrl+c或使用`consul leave`命令，以优雅地停止这两个agent。

#### 清理环境
Vagrant会自动停止并关闭其创建的虚拟机，从计算机中删除其硬盘，并释放它们消耗的所有磁盘空间和RAM。

它不会清理你创建的目录或其中包含的Vagrant文件，因此，如果你想重新运行本教程，你所需要做的就是从`consul-getting-started-join`目录中再次运行`vagrant up`。

可以在consul-getting-started-join目录中运行以下命令来清理虚拟环境。

	$ vagrant destroy
	
	n2: Are you sure you want to destroy the 'n2' VM? [y/N] y
	==> n2: Forcing shutdown of VM...
	==> n2: Destroying VM and associated drives...
	    n1: Are you sure you want to destroy the 'n1' VM? [y/N] y
	==> n1: Forcing shutdown of VM...
	==> n1: Destroying VM and associated drives...

#### 下一步
在本教程中，你通过连接两个Consul agent（Client agent和Server agent）设置了一个多agent 的Consul数据中心。 继续阅读下一个教程，以了解可帮助你将Consul投入生产环境的操作和开发教程集合。

