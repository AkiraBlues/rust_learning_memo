## Arbitrum learning memo

专门介绍Arbitrum相关知识点。



#### 入门

Arbitrum是2021年开始的一项技术，是以太坊L2的一项解决方案，目标是实现快速高效且经济的交易。它没有构造侧链，而是通过部署一些去中心化的服务器，允许在链下高效完成交易，最后再批量提交到链上。

因为它大量的业务是在它自己的服务器上完成的，**相当于是对L1的业务进行代理**。

因为核心是业务代理，所以Arbitrum有几个特点：

- 基于L2思路，增加了拓展性
- 降低了GAS成本

- 安全性依赖于以太坊
- 可以像在以太坊转账那样在Arbitrum上转账或者开发部署智能合约

Arbitrum自身也提供去中心化网络的支持，比如它主要的Arbitrum One和Arbitrum Nova都是生产环境网络，当然也提供了测试环境网络，开发者也可以在本地搭建相关开发环境。



#### L2技术之Rollup链

由于Arbitrum就是L2的一种实现，因此它应用了其他的L2技术，其中最核心的就是，叫rollup，在区块链行业中一般也称为rollup链，它和L1链一样有区块，区块之间也和L1链一样通过parentHash指向上一个区块，它实际上就是去中心化的网络节点，也存储L2区块数据。

rollup技术在2019年由以太坊创始人提出概念，之后在2020年开始应用，目前有2套方案，Optimistic Rollups和ZK-Rollups。

Optimistic Rollup，即乐观Rollup，**它假设所有交易都是有效的，在被证明无效之前**。因为都是有效的，在链下完成交易后，会一视同仁地提交到以太坊L1链（也就是以太坊主链），之后给出一段挑战时间（一般是7天），在此期间任何人都可以提出证明，来否认交易的有效性，如果证明有效，则这批待确认的交易需要重新执行，负责提交的序列器（类似智能合约）会被检视和处罚，如果没有问题，则7天之后交易得到确认，**用户也只能在交易确认后提取加密货币**，即一切顺利的话链下交易后7天才能提现。

ZK-Rollup，即Zero Knowledge Rollup，零知识，**它假设所有交易都是有问题的，因此在提交到L1之前都需要被验证**，如果验证通过，提交到L1的数据除了交易本身，也包含了L2对交易有效性的背书（证明），之后这部分信息会转给L1的智能合约，如果合约验证了背书，则会把交易数据确认到L1链上。因为验证是在L2平台完成的，L1的智能合约只做审核，因此用户在交易确认后就可以立刻提现。但是显然，**这种背书会带来一定的中心化风险，特别是如果证明过程只有少数节点参与的话**。



#### AVM

Arbitrum的解决方案就是去中心化的服务器，而且它也参考了以太坊，设计了它自己的虚拟机，即AVM，在这些服务器上运行AVM，部署Arbitrum兼容的智能合约代码。Arbitrum Classic是它的早期网络，每个节点都运行了AVM，后续升级为Arbitrum Nitro，直接使用部署所在的OS能识别的机器码来执行，**因此抛弃了AVM**。

Arbitrum Nitro抛弃AVM是这样处理的（实际上是通过Stylus实现的，后续会提到），它通过一个编译器，来把正常的业务代码编译为当前OS的机器码，以执行正常的业务交易，当需要处理欺诈证明（后续会提到）时，又会把代码编译为WASM，以方便L1链的智能合约直接调用。因为正常的业务流程可以通过机器码完成，和L1链的交互可以通过WASM让L1链的VM去执行，因此Arbitrum Nitro自身可以抛弃AVM。



#### 相关网络

```
 - - - - - -             - - - - - - -             - - - - - - - 
|Arbitrum One|          |Arbitrum Nova|           |Arbitrum Orbit|
 - - - - - -             - - - - - - -             - - - - - - - 
      \                        |                         /
       \                       |                        /
        \                      |                       /
         \                     |                      /
          \                    |                     /
               ------------------------------------
              |Arbitrum Classic ===> Arbitrum Nitro|
               ------------------------------------
```

最初是Arbitrum Classic网络，之后升级为Aribitrum Nitro，然后基于这个网络构建了One，Nova和Orbit。后续开始先介绍Arbitrum One网络相关知识点，Nova在技术上和One是类似的，但是一些流程和协议和One不同。



#### Sequencer（序列器）

序列器是它的网络节点中的一类特殊节点，用于处理交易的时序问题，以及打包生成新的L2区块，从这个角度看序列器有点像L1的矿工，但是它们不依赖GAS，形成新的区块后也没有奖励，因为控制它们的是程序，所以**序列器就类似自动创建L2区块的智能合约**。Arbitrum的交易是按照时间顺序进行的（因为不需要依赖GAS费和矿工的个人偏好），也和以太坊一样会进入一个临时交易池，同样如果待太久也会被废弃。一般在整理好交易数据后，序列器会继续把这部分数据交给另一个程序，状态转换函数（State  Transition Function，STF）来进一步加工交易信息，加工完成后，序列器把结果提交到L1。

状态转换函数严格来说不是序列器内部的程序，但它一般会存在于序列器节点中，主要的任务就是进一步加工序列器提交的数据，具体有：

- 初步验证交易是否可执行，比如发起转账的人，它的余额是否足够
- 更新对应账户的状态

STF的工作过程可以完全预测的，也是可以本地运算的，因为只要已知所有的交易单，就可以计算出这一批交易执行后的当前区块的状态，**所以它更新区块或创建区块都不需要共识机制，因为结果就是纯数学运算的问题**。处理完成后，STF会直接更新当前L2区块的状态，甚至创建新的L2区块（有待确认），并把更新结果返还给序列器，最后序列器把结果提交到L1。

在提交到L1确认之前，交易属于软确认状态，提交到L1之后，就属于硬确认状态了。



#### Fraud Proof（欺诈证明）

Arbitrum的序列器和STF在处理完L2交易后，会批量提交给L1，此时就进入了欺诈证明环节，即在一个给定时间内，所有参与者需要给出这批交易内存在虚假的交易或者篡改的交易，并给出证明，**一旦某个参与者挑战并给出了证明，L1的智能合约会自动验证这个证明的有效性**，即参与者发起挑战后，后续的过程基本就是自动化的了，**如果一个批次内存在无效交易，则整个批次的交易都要被视为无效**，相关的L2数据也要回撤到提交这批交易之前的状态，有点类似于数据库的事务的设计了。

此外，L1智能合约实际上是通过Arbitrum Nitro的WASM代码来验证欺诈证明是否有效的，即Arbitrum提供了验证方法和验证程序，L1链的智能合约只负责调用这部分程序，程序是WASM格式。

此外，Arbitrum采用了改进的验证欺诈证明的流程，叫做交互式欺诈证明，以降低合约调用WASM代码验证欺诈证明的负担（传统的欺诈证明，完全依赖智能合约执行L2的WASM代码的效率）。在这个新流程中，质疑方和被质疑方（质疑方=挑战者，被质疑方=自证者）需要轮流提交一部分的证明，以解决质疑点，分歧点。比如挑战者先提交证明到L1智能合约，之后多轮流程开始，每轮都需要双方提供证明，当最后出现无法协调的分歧点时，才会触发智能合约的接入和验证，并最终判决。



#### Arbitrum Nova

这个Nova和One底层都是Nitro网络，但是上层有区别。简单来说Nova构建了一个小组，牺牲了部分去中心化，换来了效率。Nova内部有一个Data Availability Committees，DAC，数据可用性委员会，它是AnyTrust协议的产物。

AnyTrust更像是一个协议而非技术，它规定了一个DAC组织用于存储数据和提供数据，组织内有2票否决权，任何决策都需要N - 1个成员来配合，因此如果有2个成员是正直的，不配合错误的决策，则决策就无法执行。DAC管理和提供数据，因此数据不是去中心化的，这样可以降低成本。

DAC通过让N - 1个成员对数据进行签名，提供了DA Cert，数据可用性证书，来背书数据的正确完整性。所有证书都有对应的哈希值、过期时间和N - 1成员的签名证据。

AnyTrust协议不仅规定了DAC，还规定了委员会运行的软件，即DA Server，它一般提供2类API，针对序列器的API和REST API，前者智能由序列器访问，后者是全球开放的，可以通过哈希值查询出对应的数据块。序列器API有写入修改权限，即序列器通过向此API提交L2交易数据，以形成新的L2区块，REST API则只有查询权限。

注意，**不像Arbitrum One那样，序列器搜集的交易都会发布到L1链，在Nova网络下，L2数据由DAC管理，存储于DA Server内，DAC一般只把对应的DA Cert而非完整数据发布到L1链，只有在必要时（签名失败时）会把完整数据发布到L1链**。这个设计本身就是带有中心化风险的，所以通过N-1签名机制，让DAC主要成员为这些数据背书。当序列器搜集到足够的数据，准备写入DA服务器时，会需要N - 1签名，如果签名成功，则签名后的证书发布到L1链，如果签名失败（比如成员时间不够或者参与人数不够），则表示AnyTrust协议不成立，此时则回退到One的模式，即完整数据会被提交到L1链，然后进入欺诈证明环节，后续处理流程和One网络是一样的。

所以，Nova场景下，一般数据都是存储于DA服务器，而One场景下都是存于L1链上，数据可用性是两个网络的最大差异。Nova通过牺牲部分去中心化换来了效率，适用于对延时很敏感的场景，比如游戏，社交媒体等。



#### Arbitrum Orbit

Orbit是一个支持用户自定义的网络，2023年推出，它是L2和L3兼容的，如果选择了L3方案则可以和L2交互，如果选择了L2方案则需要自己去和L1交互。这种更适合企业级平台。它的特点有：

- 带宽是专用的，因为是L3网络，不需要受到L1的限制
- 支持EVM+，因此可以使用多种语言来开发智能合约
- 支持后续独立于以太坊的产品演化路线，因为它本质也是off-chain
- GAS价格更独立稳定，目前L1的GAS价格是受制于整个链的繁忙程度控制的，即使自身业务很清闲但如果整体很繁忙，GAS费用也会很高，使用Orbit可以让GAS价格更独立稳定，因为带宽和计算资源都是专用的，所以其他业务的繁忙程度不会影响到自身的GAS价格
- GAS代币可自定义，只要是符合ERC-20标准的就可以

备注一下，ERC是以太坊社区进行制定和维护的一系列标准，相当于以太坊社区的一套共识，其中ERC-20定义了什么是代币，和代币的一些必备能力，比如总供应量，转账，查询余额等等



#### Arbitrum Stylus

arbitrum stylus本身不是一个独立的分布式网络，类似one或者nova，而是在nitro技术上进一步升级，核心是恢复了nitro节点的虚拟机配置，但是把AVM改为了支持WASM的虚拟机，因此使得任何nitro节点既可以直接执行编译为机器码的智能合约，又可以执行编译为WASM的智能合约，前者给L2网络提供直接支持，后者用于跟L1链交互。

也可以理解为，**L1链通过EVM执行智能合约，arbitrum stylus则在L2链上增加了支持WASM的虚拟机，使得L1链也可以通过调用L2链节点来执行WASM，因此L1链节点也间接具有了执行WASM的能力，因此从外部看是对EVM的增强，即EVM+**。

同时由于这个增强，任何可以编译为WASM的语言都可以用于编写arbitrum的智能合约，这就是arbitrum虽然是EVM生态但是可以支持RUST的原因。此外stylus对RUST的支持会更优先于其他语言，因为它性能更好，更小的算力开销就可以完成任务，因此应用于链上可以降低GAS费用。



#### 以太坊智能合约介绍

这段主要参考以太坊的教程。

智能合约部署在区块链上的自动执行的程序，且一旦知道了输入参数，结果可以预测，因而执行过程不存在随机性。它在进行判断时，遵循的逻辑是预先写好的，因此不需要人来参与决策。

智能合约可以操作代币，即它可以接收代币，存储代币，发放代币，也可以调用其他智能合约。

一般会用自动售货机来表示智能合约，即它的商品是确定的，人可以发起购买流程，选择了商品后，售货机会接收金钱（代币），把人选择的商品从货架上推出，商品掉落到底部取货口，最后人取走商品，整个过程都是可以预测的，其他的异常场景也是可以预测的。

**智能合约是部署在区块链的世界状态树内，单独的区块只在区块头记录它作为最新区块时的世界状态树的状态根的哈希值，因此不包含智能合约代码**，而由于每个完整节点都保存了一份最新的世界状态树，因此每个完整节点也保存了所有部署的智能合约，而且它一旦部署就不能修改，因此每个完整节点运行的这份智能合约，代码都是一样的。

智能合约主要是用来处理区块链上的用户间的不信任问题，现实世界，不信任的双方进行货物交换，一般都要借助中立的三方平台，比如参考淘宝闲鱼，买家钱都是先打款到平台上，确认收货后平台才会把钱转给卖家。当然三方平台是否足够中立也是问题，比如P2P暴雷这种事情也是发生过的，平台也有可能跑路。

而在去中心化平台上，没有所谓的中立三方平台，因此交易双方就借助智能合约来完成交易，而因**为智能合约的所有行为是可以预测的，无法修改也无法销毁，生命周期和区块链保持同步，而且所有判断过程都是可预测的，判断过程没有真人参与，因此实际上会比三方平台更加中立也更加可靠**。

当然这样吹智能合约也不对，它也是有缺陷的，**因为智能合约可以保存代币，而且代码透明，因此理论上黑客可以通过研究智能合约的代码来制定攻击策略，从而盗取代币**，这种事情实际上发生过，以太坊历史上最严重的一次黑客攻击发生在2016年，黑客通过上述步骤攻击了一个众筹项目，导致了大量资金损失并最终迫使以太坊区块链发生分叉，成为了ETH（以太坊）和ETC（以太坊经典），之后以太坊社区就更加重视智能合约的安全性和代码审查，比如给开发者提供更加安全的开发SDK，或者安全编程范式，或者部署前的代码审计服务等等。

以太坊把智能合约视为世界状态树上的节点，每个智能合约都有2类存储数据的空间，Storage和Memory。

Storage是会存储在状态世界树内的，因此对它的修改会导致广播事件，从而触发所有完整节点去更新对应的世界状态树，所以修改Storage会导致极大的开销。此外，虽然以太坊对智能合约可以使用的Storage没有明确限制，但是它会影响GAS费用，即**如果一个智能合约在Storage内存储了大量数据，那么每次调用此智能合约的GAS费就会更高**，从而使得此智能合约在商业上更不受欢迎。

Memory是执行代码中的临时数据，只在每次执行合约代码时创建，执行结束后就会销毁，不会存储在世界状态树内，因此可以支持频繁地修改。

另外由于区块链在全球分布，不同节点上都跑着同一份智能合约，**当不同的EOA都发起了铸币操作时，区块链的事务特性就会确保，有多个铸币交易存在时，只会有一个交易得到确认，其他的都会回滚，从而解决类似线程竞赛的问题**。当然，新区块在某个节点诞生后，一般会在几十秒的时间内传播到全球其他节点，以降低线程竞赛产生的概率。



#### 用Stylus Rust SDK写智能合约--介绍

Arbitrum推出Stylus主要目的就是实现用高效的RUST语言可以写以太坊合约。在Stylus之前，以太坊生态和Solidity语言是高度绑定的，就像KOTLIN和安卓那样，javascript和前端开发那样。但是在Stylus推出之后RUST也可以用于以太坊智能合约的开发，扩宽了以太坊生态的面向开发人群，还有其他的增强。



#### 环境准备

目前结论是windows环境下搞arbitrum不现实，官方说rust sdk支持是支持RUST，可以写，但是编译构建会失败，因为底层的一些东西不够友好，所以windows下搞最佳策略还是通过虚拟机安装ubuntu，因为虚拟机可以实现完整的linux环境模拟，所以3月3日着重研究怎么用虚拟机装ubuntu然后搞linux系统。

所以先搞定WMware和LINUX系统，这里选择WMware wortstation pro 17，Ubuntu 22版本，参考其他教程安装。

安装完成后就当我有了一个Ubuntu电脑。



#### 虚拟机相关安装和配置

登录VMware官网，注册账号，然后获取workstation pro 17下载地址。

之后去下载一个Ubuntu的镜像，大概4G多，可以选清华的那个。

然后安装WMware，然后安装Ubuntu镜像，最后进入系统。

然后是非常麻烦的网络配置，要求是虚拟机可以使用宿主的代理，测试了多个方案，最后的结果是这样：

- clash verge取消tun模式，回到原始的系统代理模式（后面测试恢复到tun模式又可以了，配置按照系统代理模式的走就可以，tun代理适配器不用做任何共享）
- 虚拟机关机情况下，网络适配器设置为NAT，或者自定义然后选VMnet8，这个8就是适配NAT的，NAT本身可以理解为宿主和虚拟机共享的一套网络（也就是宿主也要处于这个网络的管理之下）
- 然后是进入VMware的虚拟网络编辑器，新建一个VMnet1，对应仅主机模式，然后新建一个VMnet8，对应NAT模式，其他IP配置的先不要动，然后点击左下角还原默认设置，一段时间后，去控制面板网络适配器里面看，会发现出现了net1和net8的适配器，不需要在LAN或者WLAN的适配器上配置共享
- 还原默认设置后，会发现IP地址发生变化了，重点记录net8的IP地址，可以在任务管理器的进程里面确认，通常是192.168.X.X，它会和宿主的网段区隔开，比如宿主的网段是192.168.31.X，对应家里的路由器网段，那么它这个肯定不会是31
- 然后配置clash的代理端口，允许局域网访问，然后开虚拟机进入Ubuntu系统，进入系统配置，把这个net8的IP地址输入进去，这个就是虚拟机通过net8网络找到宿主的IP地址，也就相当于是宿主开热点然后另外一台机器连接了，此时宿主的热点就是一个net8网络，里面也给宿主分配了一个新的IP地址
- 用火狐浏览器测试外网是否能访问，如果可以则成功，也可以从clash的日志里面看到虚拟机的访问情况
- 还要在终端里面去配置一下代理，系统设置的代理对终端的很多CURL命令无效，之后配置好后可以直接通过curl去安装rustup的linux版本了

然后是在linux环境下操作了，先直接安装DOCKER，因为这个对系统影响最大，另外不要在/etc/apt/souces.list里面去加清华的源，会导致包版本不一致，导致后续安装KVM虚拟化相关失败，另外建议最好搞一个磁盘清理的，比如bleachbit，时刻掌控磁盘空间大小，避免SDA3分区盛满的情况。

安装rustup，然后是VS CODE，之后是GIT，其他相关所需的，安装cargo-stylus时，先安装pkg-config，然后安装libssl-dev，再安装cargo-stylus，最后终于构建成功，有了CARGO-STYLUS工具链，之后再引入一个DEMO应用，然后编译构建，终于成功了。

之后流程参考Arbitrum官方的教程，流程是这样的：

- RUST开发环境，即RUST相关工具，环境，和IDE，这部分之前已经提到
- Docker，这部分后面会介绍
- Foundry CLI，用于和EVM合约进行交互
- Nitro开发节点，需要从GITHUB上下载一个开发节点项目并启动它，它的脚本会触发docker相关行为



#### Docker和NITRO开发节点相关

Docker介绍，它是一个公开的平台，可以用来开发应用。Docker可以在一个沙箱中构建和运行应用，甚至在多个沙箱中做同样的事情。沙箱非常轻量，它包含了运行应用所需的所有环境，因此不需要直接在宿主环境（一般指个人用开发电脑）上安装对应的开发环境。一个最简单的过程就是通过Docker搭建一个CI / CD环境，这样每次代码提交后都可以直接在Docker上进行自动代码审核，打包构建，测试，发布等一系列流程，而不用去在自己的电脑上直接搭建这样的环境。

Docker安装过程记录：

- 确认OS，linux下需要开启KVM虚拟化，由于我是通过虚拟机跑的Ubuntu，因此实际上是通过虚拟机的虚拟化去支持KVM虚拟化，宿主Windows11环境本身需要调整一下，然后是虚拟机开启对应虚拟化配置，然后才是Ubuntu自身安装一些软件去开启KVM虚拟化，并授权给当前用户
- 之后参考Docker官方教程去安装，还是通过下载deb包的形式安装
- 然后在设置里面的docker engine修改一下国内镜像源，写法是`"registry-mirrors": ["https://some-domain.com",
  "https://some-domain2.com"]`，这里要改成实际的镜像源
- resource里面改一下memory limit，最少要4G

然后就开始涉及DOCKER最重要的2个概念，container（容器）和image（镜像），首先要下载镜像，就是操作OS，它是静态的，不可修改的，然后基于这个静态的镜像可以创建多个运行实例，就是容器，每个容器都是动态的，可以修改的，而且各自独立。

DOCKER安装好之后可以在gitbash或者CMD使用`docker --help`来确认它安装好了，并且可以查询各种命令。WMware内安装Docker会有点兼容性问题，关闭之后无法再次打开，此时进入任务管理器杀一下docker.backend进程就可以了。

之后在DOCKER的主界面可以打开终端，然后执行以下命令，会开始下载镜像并创建容器：`docker run -d -p 8080:80 docker/welcome-to-docker`，会先尝试加载本地镜像，失败后从镜像源下载对应镜像，然后创建容器，如果一切顺利，运行后，打开浏览器，输入`localhost:8080`就可以看到一个页面，表示容器运行成功。

安装Foundry CLI，先启动gitbash，确保可以使用curl命令，然后执行`curl -L https://foundry.paradigm.xyz | bash`，这个是用来下载foundryup（类似rustup），如果出现握手错误或者其他网络连接错误，设置一下VPN，确保`foundry.paradigm.xyz`要走代理，然后安装，成功后重启一下gitbash，然后执行`foundryup`，如果成功会看到FOUNDRY标题和下载安装进度条，安装完成后执行一下`foundryup -v`来查一下版本号，如果有就是真的装好了，或者执行`cast --help`确认cast命令是否可以用。

安装Nitro开发节点，这个需要在一个文件夹里面打开gitbash，因为需要把对应代码下载到这个位置，执行`git clone https://github.com/OffchainLabs/nitro-devnode.git .`，注意最后这个点，它表示把代码下载到当前所在文件夹，而不是创建新文件夹，之后执行`./run-dev-node.sh`，这个脚本会尝试通过docker相关命令去调用DOCKER去下载镜像创建容器，然后部署一个样例智能合约。如果一切顺利，可以从DOCKER那里看到有一个容器正在运行，说明脚本执行成功。

还有几个工具需要安装，先安装cargo-stylus，它是一个CLI，用于辅助开发stylus智能合约，注意以下命令行只能在Linux环境下运行：

```
RUSTFLAGS="-C link-args=-rdynamic" cargo install --force cargo-stylus
```

如果安装失败，提示rustc版本过低，则执行`rustup self update`和`rustup update`更新一下所有的工具。

安装完成后执行`cargo stylus --version`，确认是否能出结果。

然后可以创建一个hello world项目了，在项目空间里面执行`cargo stylus new <PROJECT_NAME>`。创建项目后会下载模板代码，然后不着急运行，先在项目内执行`rustup target add wasm32-unknown-unknown`，来往当前项目的RUST的构建目标内增加对WASM的支持，这里不指定rustup工具链为具体数字版本，采用默认的stable版本，如果后续出现兼容性问题，再修改。

之后启动Nitro开发节点，即先打开DOCKER，然后执行开发节点项目的`./run-dev-node.sh`，确保容器在运行。如果运行成功，浏览器输入`localhost:8547`应该会触发一个下载，实际是部署了一个Cache Manager智能合约。

之后再执行`cargo stylus check`，检查项目的合约情况，Ubuntu环境下应该能编译通过并进行检查，实际结果如下：

```
project metadata hash computed on deployment: "436d2e4d83312a02448a2d81feaf16b83e841a51a54f16a355d5fab4fe91644d"
stripped custom section from user wasm to remove any sensitive data
contract size: 7.2 KB
wasm data fee: 0.000077 ETH (originally 0.000064 ETH with 20% bump)
```

这样就启动了DOCKER的开发节点并验证了智能合约的格式是有效的。



#### 在开发节点上部署智能合约

然后就执行本地部署了，通过`deploy`命令进行部署，先预估一下GAS费用：

```
cargo stylus deploy --endpoint='http://localhost:8547' --private-key="0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659" --estimate-gas
```

如果执行成功，会看到以下字样，表示计算出来了预估的GAS费用：

```
contract size: 7.2 KB
wasm data fee: 0.000077 ETH (originally 0.000064 ETH with 20% bump)
estimates
deployment tx gas: 67217224
gas price: "0.100000000" gwei
deployment tx total cost: "0.006721722400000000" ETH
```

之后就是执行实际的本地部署，移除掉预估GAS的参数`--estimate-gas`：

```
cargo stylus deploy --endpoint='http://localhost:8547' --private-key="0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659"
```

如果本地部署成功，会看到：

```
wasm data fee: 0.000077 ETH (originally 0.000064 ETH with 20% bump)
deployed code at address: 0xa6e41ffd769491a42a6e5ce453259b93983a22ef
deployment tx hash: 0x64461d51642f89a024fb4ddd2939ef69c1bd1ff693e0a9de239435ce4828a6ed
contract activated and ready onchain with tx hash: 0x902f8491b74bf1bc0125636ecede9c69104a78c7408baf904320f4c11b9499e0
```

保存下这个合约地址，保持nitro-devnode运行，后续用命令行可以进行其他操作了，比如调用刚刚部署的智能合约对外暴露的number方法：

```
cast call --rpc-url 'http://localhost:8547' --private-key 0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659 0xa6e41ffd769491a42a6e5ce453259b93983a22ef "number()(uint256)"
```

结果会返回一个0，表示目前这个number函数返回值就是0。

再执行另一个方法，调用increment公开方法：

```
cast send --rpc-url 'http://localhost:8547' --private-key 0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659 0xa6e41ffd769491a42a6e5ce453259b93983a22ef "increment()"
```

返回值如下：

```
blockHash            0xd20e9c513fd140939d2fcfddd10200a676a580272f8dd4e607702ba7b1353a1e
blockNumber          6
contractAddress      
cumulativeGasUsed    993485
effectiveGasPrice    100000000
from                 0x3f1Eae7D46d88F08fc2F8ed27FCb2AB183EB2d0E
gasUsed              993485
logs                 []
logsBloom            0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root                 
status               1 (success)
transactionHash      0x1ff001e124097dc0d9c40bcc4bf3b28ef7b3874f8c6c0d66dfa6ad58f38b6652
transactionIndex     1
type                 2
blobGasPrice         
blobGasUsed          
authorizationList    
to                   0xA6E41fFD769491a42A6e5Ce453259b93983a22EF
gasUsedForL1         936000
l1BlockNumber        0
```

status值是1表示成功，之后再调用cast call指令可以发现，计数值变了，这就说明链上的节点的值被修改了。

到此就实现了开发智能合约并且在本地测试节点上进行调试的整个过程。

另外foundry的cast send是用于有**交易行为**的操作，即操作会导致区块链状态更改，而cast call是用于**查询行为**的操作，不会导致区块链状态更改。

此外，cast send/call的**函数签名不需要包括返回值**，foundry会自动处理，一般入参如果RUST里面是U256，则传入uint256即可。

**另外注意使用export-abi查看智能合约的方法的时候，RUST的snake_case写法会被自动转化为camelCase写法，比如源码里面写的是`some_function`，转为WASM之后会变成`someFunction`**，如果后续使用cast调用时，方法名称搞错了，就会直接报错，而没有任何明显的提示。



#### FOUNDRY和智能合约交互命令行格式

格式写法：

```
cast call/send --rpc-url <合约部署的链地址> --private-key <当前账户私钥> <合约地址> "<函数签名>" <函数入参>
```

举例：

```
cast call --rpc-url 'http://localhost:8547' --private-key 0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659 0xa6e41ffd769491a42a6e5ce453259b93983a22ef "number()"

cast send --rpc-url 'http://localhost:8547' --private-key 0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659 0xa6e41ffd769491a42a6e5ce453259b93983a22ef "setAsset(address)" 0xa6e41ffd769491a42a6e5ce453259b93983a22ef

cast call --rpc-url 'http://localhost:8547' --private-key 0xb6b15c8cb491557369f3c7d2c287b053eb229daa9c22138887752191c9520659 0xa6e41ffd769491a42a6e5ce453259b93983a22ef "addBalance(uint256)" 2
```

函数签名注意是camelCase，即使源码是snake_case。

如果函数入参有多个，用空格区分。



#### Stylus RUST SDK智能合约入门

RUST SDK是基于Alloy构建的，后者是RUST以太坊生态内的一组库集合。

这里先给出一个最简单的计数器智能合约的stylus写法：

```rust
use stylus_sdk::{alloy_primitives::U256, prelude::*};

// Generate Solidity-equivalent, Rust structs backed by storage.
sol_storage! {
  #[entrypoint]
  pub struct Counter {
    uint256 number;
  }
}

#[external]
impl Counter {
  // Gets the number value from storage.
  pub fn number(&self) -> Result<U256, Vec<u8>> {
    Ok(self.number.get())
  }

  // Sets a number in storage to a user-specified value.
  pub fn set_number(&mut self, new_number: U256) -> Result<(), Vec<u8>> {
    self.number.set(new_number);
    Ok(())
  }
}
```

智能合约通过Storage和Memory存储数据，其中Storage在SDK内对应了2个宏，分别是`#[storage]`宏和`#[sol_storage]`宏，它们2个都是用于定义合约使用Storage的具体数据格式，区别是前者采用纯RUST风格，后者采用以太坊原生的Solidity风格，从可读性上后者更好一些，而且也能直接对应部署到以太坊后的数据格式。

```rust
#[storage]
pub struct Contract {
    owner: StorageAddress,
    active: StorageBool,
    sub_struct: SubStruct,
}

#[storage]
pub struct SubStruct {
    // types implementing the `StorageType` trait.
}

sol_storage! {
    pub struct Contract {
        address owner;                      // becomes a StorageAddress
        bool active;                        // becomes a StorageBool
        SubStruct sub_struct,
    }

    pub struct SubStruct {
        // other solidity fields, such as
        mapping(address => uint) balances;  // becomes a StorageMap
        Delegate delegates[];               // becomes a StorageVec
    }
}
```

之所以推荐使用`#[sol_storage]`宏，是因为它贴近solidity编写的智能合约，使得从对方迁移过来比较方便。

**对Storage进行CRUD操作需要通过方法实现，不能直接赋值**。以下是一个智能合约增发代币的例子：

```rust
sol_storage! {
    pub struct Erc20<T> {
        mapping(address => uint256) balances;
        mapping(address => mapping(address => uint256)) allowances;
        uint256 total_supply;
        PhantomData<T> phantom;
    }
}

impl<T: Erc20Params> Erc20<T> {
    pub fn mint(&mut self, address: Address, value: U256) { // 增发代币到指定账户，因为代币变动了，也要修改智能合约自身的状态，所以必须使用&mut self
        let mut balance = self.balances.setter(address); // 表示获取目标地址的代币余额，用setter很奇怪
        let new_balance = balance.get() + value; // 计算目标地址的代币增发之后的余额
        balance.set(new_balance); // 更新目标地址的代币余额
        self.total_supply.set(self.total_supply.get() + value); // 把增发的部分计算到代币总供应量上
    }
}
```

RUST SDK定义了一组可以存储在Storage内的数据格式，都以`StorageXXX`表示，比如`StorageBool`表示bool类型，因此实际上是基本类型。对于所有Storage的非集合类型（包括StorageString）的数据格式，都可以通过`get`和`set`方法来获取和修改，举例：

```rust
impl Contract {
    /// Gets the owner from storage.
    pub fn owner(&self) -> Result<Address, Vec<u8>> {
        Ok(self.owner.get())
    }

    /// Updates the owner in storage
    pub fn set_owner(&mut self, new_owner: Address) -> Result<(), Vec<u8>> {
        if msg::sender() == self.owner()? {  // we'll discuss msg::sender later
            self.owner.set(new_owner);
        }
        Ok(())
    }
}
```

对于集合类型的，比如`StorageVec`和`StorageMap`，可以参考它们对标的原来的集合类型，采用push，insert，replace等方法操作，举例：

```rust
sol_storage! {
    pub struct SubStruct {
        mapping(address => uint) balances;  
        Delegate delegates[];              
    }
}

impl SubStruct {
    pub fn add_delegate(&mut self, delegate: Delegate) -> Result<(), Vec<u8>> {
        self.delegates.push(delegate);
    }

    pub fn track_balance(&mut self, address: Address) -> Result<U256, Vec<u8>> {
        self.balances.insert(address, address.balance());
    }
}
```

Storage存储的清除，合约销毁也是可以做到的，这样可以降低GAS费，举例：

```
sol_storage! {
    #[derive(Erase)]
    pub struct Contract {
        address owner;        
        uint256[] hashes;   
    }
}

impl Contract {
    fn remove(mut contract: Contract) {
        contract.owner.erase();
        contract.hashes.erase();
    }
}
```

实际上Storage消除就是把所有字段值都改为0或者类似的值。

合约方法的部分，合约本身按照结构体，有数据使用，当然也会有方法。RUST SDK提供了对方法的描述，比如对内对外，一个方法如果是内部的则只有合约本身可以调用，如果是外部的则其他智能合约可以调用。和RUST一样所有方法默认就是内部的，如果需要外部方法，通过`#[external]`再写一个impl，举例：

```rust
#[external]
impl Counter {
    pub fn number(&self) -> Result<U256, Vec<u8>> {
        Ok(self.number.get())
    }

    pub fn set_number(&mut self, new_number: U256) -> Result<(), Vec<u8>> {
        self.number.set(new_number);
        Ok(())
    }

    pub fn increment(&mut self) -> Result<(), Vec<u8>> {
        let number = self.number.get();
        self.set_number(number + U256::from(1))
    }
}

impl Counter {
    pub fn other_internal_op(&mut self) -> Result<(), Vec<u8>> {
        ...
    }
}
```

还有一个宏，`#[entrypoint]`，表示这个合约是有入口的，即允许和外部进行沟通的，如果没有这个宏则编译会失败，相当于告诉编译器，当前这个智能合约的根结构体是什么，因为一个合约可能包含多个结构体，而且RUST允许自引用或者循环引用，因此必须由开发者显式告诉编译器哪个结构体是最外侧的。

另外关于以太币ETH的处理问题，需要用`#[payable]`宏标记在函数上，表示此函数支持处理ETH，不然会失败。可以理解为ETH和其他代币相比具有更高的内部权限，它的处理也是走L1链的内部协议。

智能合约之间也有继承关系，因为可能已经有了一些基础功能的智能合约，开发者只需要继承它们就可以，不用重复造轮子，比如：

```rust
#[external]
#[inherit(Erc20)]
impl Token {
    pub fn mint(&mut self, amount: U256) -> Result<(), Vec<u8>> {
        ...
    }
}

#[external]
impl Erc20 {
    pub fn balance_of() -> Result<U256> {
        ...
    }
}

sol_storage! {
    #[entrypoint]
    pub struct Token {
        #[borrow]
        Erc20 erc20;
        ...
    }

    pub struct Erc20 {
        ...
    }
}
```

ERC20代币，如果要构造一个具体的代币，比如发币，那么就要通过智能合约完成，为此就要在智能合约内定义一个ERC20代币，为此就需要实现IERC20接口，这个接口定义了mint，burn，transfer，approve，即发币，销毁，转账，授权等行为的定义。

一般来说自己写的智能合约，如果要实现SDK里面的接口，不用做额外的事情。但是如果需要增加接口导出功能，比如导出接口供别人使用，则需要这样配置：

```rust
#![cfg_attr(not(feature = "export-abi"), no_main)]
```

然后cargo.toml配置导出能力：

```toml
[features]
export-abi = ["stylus-sdk/export-abi"]
```

最后通过这个命令导出接口：`cargo stylus export-abi`，这样如果一个智能合约需要别人来调用，就可以通过这个方式让别人知道它提供了哪些接口函数。

调用其他智能合约，可以通过其他智能合约暴露的接口来调用，如果对方没有直接支持导出接口的能力，则也可以通过类似JAVA反射的能力，即通过`call`方法指定目标合约地址，然后把需要调用的函数，传参等封装到call_data内，最后完成调用，举例：

```rust
// stylus_sdk::call::call

let return_data = call(Call::new(), target_contract, call_data)?;

// 一个更具体的例子
pub fn deposit(&mut self, amount: U256) -> Result<(), Vec<u8>> {
    let selector = function_selector!("transferFrom(address,address,uint256)");
    let data = [
        &selector[..],
        &msg::sender().into_array(),
        &self.recipent.get().into_array(),
        &amount.to_be_bytes::<32>(),
    ].concat();
    call(Call::new(), self.target.get(), &data);

    // ...

    Ok(())
}
```

还有一种调用方式，叫委托调用，使用`delegate_call`实现，就是类似A委托B调用C，然后C修改B的状态。一般来说用于合约升级，由于合约一旦部署就不能修改，而区块链支持修改合约的状态Storage，因此有人想出了一种办法，即把一个完整的合约拆分为2部分，数据部分和逻辑部分，其中数据部分就是存储于Storage的部分，内部再加一小段委托调用的逻辑，并保存逻辑部分的地址，之后逻辑部分再单独部署，这样每次调用的都是数据部分，数据部分通过委托调用去实际执行逻辑部分合约，如果需要升级，先部署新合约，然后修改数据部分的Storage存储的逻辑部分的地址，指向新合约地址即可。



#### 条件编译属性

在RUST里面有一个配置，语法是`#![cfg_attr(...)]`，叫条件编译，即可以设定满足某种条件才进行后续操作，主要作用是通过设置条件来控制是否编译某部分代码，有点类似前端vite构建时设置的环境变量（可以在代码或者vite的html模板中设置，如果环境变量符合某个范围或者等于某个值，就允许某些代码的存在，否则就不允许），一般会通过一个--feature命令行参数来进行控制，即通过传入不同的feature打包为不同的代码，这个就是适应了Arbitrum的EVM+，即一套源码既可以构建为目标节点的代码，用于处理节点业务，也可以构建为智能合约的代码，以部署到链上，也可以供外部调用以生成智能合约的接口文档。

另外注意`#[]`是局部属性，它只能影响它下一行的代码结构，比如下一行是函数，结构体，它可以影响。`#![]`是全局属性，可以影响整个库，一般放在库的入口文件开头部分。



#### 引入外部库

使用`extern crate some_crate;`语法来引入外部库，一般和`use foo::bar`一样都是写在文件开头。

从RUST2018之后也可以直接使用`use outer_foo::bar`来直接引入外部库。

比如`extern crate alloc;`一般来说引入外部库，也需要同步在Cargo.toml里面添加对应依赖，但是alloc是RUST标准库的一部分，因此一般会通过`use std::alloc;`来直接使用，但是在编写智能合约时一般都会通过条件编译禁止标准库以降低构建大小，此时就需要显式引入标准库内部的alloc功能，所以也就需要这样的extern写法，**但是因为它是标准库，所以不需要在Cargo.toml里面写额外依赖**。



#### 编写自己的ERC20代币



##### ERC20代币的规范

ERC20规范是以太坊的委员会定义的代币规范，即以太坊不仅支持ETH交易，也支持在L1链上发行其他的遵循ERC20规范的代币，所以，以太坊生态可以支持多种加密货币存在。

由于以太坊生态默认使用Solidity语言，因此它的ERC20代币规范也是用这个语言来定义的，有点接近JS，前端应该能看懂：

```solidity
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount)
        external
        returns (bool);
    function allowance(address owner, address spender)
        external
        view
        returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount)
        external
        returns (bool);
}
```



##### 开发语法准备

由于发行代币需要部署到EVM+节点上，因此需要把同一套代币编译为目标节点的可执行文件，以及支持和ETH主链进行交易的WASM格式，这就需要通过`#![cfg_attr(...)]`来实现多环境构建，语法是`#![cfg_attr(<condition>, <result1>, <result2>...)]`

比如`#![cfg_attr(not(feature = "export-abi"), no_main, no_std)]`，这里的条件是`not(feature = "export-abi")`，即`feature != "export-abi"`，表示如果构建时的入参命令中，feature不等于export-abi，就添加no_main和no_std属性，换言之只有是export-abi这种生成接口的命令时，才会移除no_main和no_std属性，至于它们具体的含义是：

- no_main，没有main函数入口，编译器也不应该去找一个main入口
- no_std，不会用到RUST的标准库，只会用到核心库core，因为区块链节点通常不会是完整的LINUX系统，所以不把标准库打包到代码内可以减少构建后体积

除了上述提到的条件编译，禁用标准库，显式引入alloc等外，还有其他的惯例操作，目标都是为了在对性能要求较高（低性能平台）的EVM+节点上执行智能合约：

- `#[global_allocator]`，并编写自己的内存分配器，以代替默认的内存分配器，因为RUST默认会采用目标OS的内存分配器，改成自定义的或者三方库提供的高性能内存分配器会更好，之后使用`static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;`声明一个全局静态常量ALLOC，它的类型显式声明，并赋值为INIT
- 项目模块化，比如发行某个代币就把相关功能抽离为一个模块，当然底层能力可以抽离到util模块内



##### 引入依赖STYLUS-SDK

从零开始搭建项目，不使用cargo-stylus的模板项目。

tool-chain最好是stable版本的，因为2024特性在stable版本才会有。

Cargo.toml内配置：

```toml
[package]
name = "erc20_demo"
version = "0.1.0"
edition = "2024"

[dependencies]
stylus-sdk = { version = "0.8.0", features = ["export-abi"]}
alloy-primitives = "0.8.20"
alloy-sol-types = "0.8.20"
wee_alloc = "0.4.5"

[features]
export-abi = ["stylus-sdk/export-abi"]

[lib]
crate-type = ["lib", "cdylib"]

[profile.release]
codegen-units = 1
strip = "debuginfo"
lto = true
panic = "abort"
opt-level = "s"
debug = false
rpath = false
debug-assertions = false
incremental = false 
```

注意profile.release必须配置，因为WASM体积是比较大的，必须要进行优化，而cargo stylus xxx相关命令都会默认采用release模式进行构建，会降低WASM文件体积。

注意VS CODE默认状态下需要启用export-abi特性。

然后是main.rs还是要保留，因为执行cargo stylus export-abi的时候需要有一个bin入口，即二进制文件main.rs入口，所以这样写：

```rust
#![cfg_attr(not(any(test, feature = "export-abi")), no_main)]

#[cfg(feature = "export-abi")]
fn main() {
    erc20_demo::print_abi("MIT-OR-APACHE-2.0", "pragma solidity ^0.8.23;");
}
```

然后构造一个模块erc20用于存放代币相关定义和行为：

```rust
use alloc::{string::String, vec::Vec};
use core::marker::PhantomData;
use stylus_sdk::{
    alloy_primitives::{Address, U256},
    alloy_sol_types::sol,
    prelude::*,
    stylus_core::log
};

pub trait Erc20Params {
    
    /// name of the token
    const NAME: &'static str;

    // symbol of the token
    const SYMBOL: &'static str;

    /// token digit precision, usally is 18
    const DECIMALS: u8;
}

sol_storage! {
    /// Erc20 implements all ERC-20 methods.
    pub struct Erc20Token<T> {
        /// Maps users to balances
        mapping(address => uint256) balances;

        /// Maps users to a mapping of each spender's allowance
        mapping(address => mapping(address => uint256)) allowances;

        /// The total supply of the token
        uint256 total_supply;

        /// Used to allow [`Erc20Params`]
        PhantomData<T> phantom;
    }
}

// declare events & errors
sol! {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    error InsufficientBalance(address from, uint256 have, uint256 want);
    error InsufficientAllowance(address owner, address spender, uint256 have, uint256 want);
}

#[derive(SolidityError)]
pub enum Erc20Error {
    InsufficientBalance(InsufficientBalance),
    InsufficientAllowance(InsufficientAllowance)
}

enum ValueEnoughMode {
    BALANCE,
    ALLOWANCE
}

// define basic operations, some methods can be used within project, but not to outside accounts
impl<T: Erc20Params> Erc20Token<T> {
    pub fn _mint(&mut self, target_addr: Address, value: U256) -> Result<(), Erc20Error> {
        let mut target_balance = self.balances.setter(target_addr); // get balance of target address
        let new_balance = target_balance.get() + value;
        target_balance.set(new_balance);

        let old_supply = self.total_supply.get();
        self.total_supply.set(old_supply + value);
        
        log(self.vm(),Transfer { // new log function needs host addr
            from: Address::ZERO, // zero addr means it's coming from smart contract
            to: target_addr,
            value: value
        });

        Ok(())
    }

    fn _valus_is_enough(&mut self, addr: Address, value: U256, mode: ValueEnoughMode) -> Result<(), Erc20Error> {
        let remain_value = match mode {
            ValueEnoughMode::ALLOWANCE => {
                let msg_sender = self.vm().msg_sender();
                self.allowances.setter(addr).setter(msg_sender).get()
            },
            ValueEnoughMode::BALANCE => self.balances.setter(addr).get()
        };
        if remain_value < value {
            match mode {
                ValueEnoughMode::ALLOWANCE => {
                    let err = InsufficientAllowance {
                        owner: addr,
                        spender: self.vm().msg_sender(),
                        have: remain_value,
                        want: value
                    };
                    Err(Erc20Error::InsufficientAllowance(err))
                },
                ValueEnoughMode::BALANCE => {
                    let err = InsufficientBalance {
                        from: addr,
                        have: remain_value,
                        want: value
                    };
                    Err(Erc20Error::InsufficientBalance(err))
                }
            }
        } else {
            Ok(())
        }

    }

    fn _transfer(&mut self, from: Address, to: Address, value: U256) -> Result<(), Erc20Error> {
        self._valus_is_enough(from, value, ValueEnoughMode::BALANCE)?;
        let mut sender_balance = self.balances.setter(from);
        let sender_old = sender_balance.get();
        sender_balance.set(sender_old - value);

        let mut receiver_balance = self.balances.setter(to);
        let receiver_old = receiver_balance.get();
        receiver_balance.set(receiver_old + value);
        log(self.vm(), Transfer { from, to, value });
        Ok(())
    }

    pub fn _burn(&mut self, addr: Address, value: U256) -> Result<(), Erc20Error> {
        let mut balance = self.balances.setter(addr);
        let old_val = balance.get();
        if old_val < value {
            let err = InsufficientBalance {
                from: addr,
                have: old_val,
                want: value
            };
            return Err(Erc20Error::InsufficientBalance(err));
        }

        balance.set(old_val - value);
        self.total_supply.set(self.total_supply.get() - value);
        log(self.vm(), Transfer {
            from: addr,
            to: Address::ZERO,
            value
        });
        Ok(())
    }
}

// public methods implementing erc20 interface
#[public]
impl<T: Erc20Params> Erc20Token<T> {
    pub fn name() -> String {
        T::NAME.into()
    }
    pub fn symbol() -> String {
        T::SYMBOL.into()
    }
    pub fn decimals() -> u8 {
        T::DECIMALS
    }
    pub fn total_supply(&self) -> U256 {
        self.total_supply.get()
    }
    pub fn balance_of(&self, addr: Address) -> U256 {
        let balance = self.balances.setter(addr);
        balance.get()
    }

    /// returns the token value which owner deposited at spender
    pub fn allowance(&self, owner: Address, spender: Address) -> U256 {
        self.allowances.getter(owner).get(spender)
    }

    pub fn approve(&mut self, spender: Address, value: U256) -> bool {
        let msg_sender = self.vm().msg_sender();
        self.allowances.setter(msg_sender).insert(spender, value);
        log(self.vm(), Approval {
            owner: msg_sender,
            spender,
            value
        });
        true
    }

    pub fn transfer(&mut self, to: Address, value: U256) -> Result<bool, Erc20Error> {
        self._transfer(self.vm().msg_sender(), to, value)?;
        Ok(true)
    }

    /// check if from's allowance(not balance) is enough to complete transfer
    pub fn transfer_from(&mut self, from: Address, to: Address, value: U256) -> Result<bool, Erc20Error> {
        self._valus_is_enough(from, value, ValueEnoughMode::ALLOWANCE)?;
        let msg_sender = self.vm().msg_sender();
        let mut remain_value = self.allowances.setter(from);
        let mut remain_value2 = remain_value.setter(msg_sender);
        let old_value = remain_value2.get();
        remain_value2.set(old_value - value);
        self._transfer(from, to, value)?;
        Ok(true)
    }
}
```

这个ERC20TOKEN结构体就相当于是一个代币的整体结构，包含了余额表，零花钱表，总供应量和额外数据，可支持Storage存储。然后定义它的私有行为，比如发币，转账，销毁，再基于这个私有行为去定义ERC20规范内的行为，最后就构成了一个原始的ERC20代币的工厂，后续再去lib.rs内定义具体的代币结构体，直接继承它就可以。

lib.rs：

```rust
#![cfg_attr(not(feature = "export-abi"), no_main)] // 注意这里要保留std，一些功能比如panic是依赖std的
extern crate alloc;

mod erc20;
use crate::erc20::{Erc20Token, Erc20Params, Erc20Error};
use stylus_sdk::{alloy_primitives::{Address, U256}, prelude::*};
// 不用声明全局内存对象处理，stylus-sdk内部已经处理了，不要乱改features配置就可以

pub struct StylusErc20Params;

impl Erc20Params for StylusErc20Params {
    const NAME: &'static str = "erc20_demo_token";
    const SYMBOL: &'static str = "EDT";
    const DECIMALS: u8 = 18;
}

sol_storage! {
    #[entrypoint]
    pub struct MyErc20Token {
        #[borrow]
        Erc20Token<StylusErc20Params> token;
    }
}

#[public]
#[inherit(Erc20Token<StylusErc20Params>)]
impl MyErc20Token {
    pub fn mint(&mut self, value: U256) -> Result<(), Erc20Error> {
        let msg_sender = self.vm().msg_sender();
        self.token._mint(msg_sender, value)
    }

    pub fn mint_to(&mut self, to: Address, value: U256) -> Result<(), Erc20Error> {
        self.token._mint(to, value)?;
        Ok(())
    }

    pub fn burn(&mut self, value: U256) -> Result<(), Erc20Error> {
        let msg_sender = self.vm().msg_sender();
        self.token._burn(msg_sender, value)?;
        Ok(())
    }
}
```

在lib.rs内实现了Erc20Params的常量，声明了代币名称，符号和小数点最小位数，然后直接使用组合+继承的方式把之前写好的代币工厂对象传入，这样这个入口代币就具有了代币工厂的所有功能，然后再简单封装一下，就可以作为export-abi的导出信息了。



##### missing import pay_for_memory_grow，问题解决记录

使用`cargo stylus check`后一直报错，提示missing import pay_for_memory_grow，尝试了很多解决办法，最后推测可能是WASM编译后文件过大导致的，等下来看一下怎么优化，但是后面分析之后发现和这个无关，尝试用AI给的例子编写了一个最简单的WASM，只有300 BYTES的，还是提示missing import pay_for_memory_grow，说明还是需要一个内存修改器来处理。

问题复现：下载一个全新的cargo stylus项目，改一下lib.rs，改为一个简化版的直接导出一个C语言的函数，然后进行测试，还是会出现上述问题，此时WASM很小，之后恢复到默认状态，执行check，编译出来的WASM是23KB，可以正常执行。因此推测是直接导出一个C语言函数，这种写法不符合STYLUS。

之后尝试一步步修改，比如先声明一个结构体入口，然后尝试编译，继续出现类似问题。怀疑是第一行条件编译导致的，尝试修改。把正常的lib.rs第一行条件编译换成有问题的lib.rs，没有问题可以通过，因此怀疑是结构体声明不完整导致的。即问题还在编码上。

然后回到ERC20项目编译，还是老问题，因此怀疑是cargo.toml配置问题，因此把ERC20的配置文件放到没问题的项目上替换看看。果然出问题了，因此问题就是cargo.toml的配置，之后就一项一项恢复看看。

测试结果：

- 删除dev-dependencies，正常
- 版本改到0.8.1，删除mini-alloc，正常
- package只保留name version edition，正常
- 删除hex，删除dotenv，正常
- features加了一个default，export-abi，不正常了，所以问题就出在这里！！！！

**结论，features尽量不要用default = [XXX]，会修改stylus-sdk的默认配置，导致缺少一些东西，导致missing import pay_for_memory_grow，非常坑爹。另外WASM的体积还是在58KB的样子，好像也可以部署。**



##### 部署测试

最后还是一样部署测试，部署之前算一下GAS，然后部署，然后用测试节点的私钥去cast send调用一下发币的功能，如果返回成功就表示给私钥所在账户发了我们定义的ERC20代币了，它来自固定的智能合约，有金额。

当热为了验证代币真的发出去了，可以修改一下智能合约，因为开发节点保存了世界状态树，因此钱包的代币金额是可以查询到的，这里就给ERC20代币结构体增加一个查询功能，中间遇到了一些问题，比如`self.vm().msg_sender()`指向的是当前调用者的地址，一般就是EOA外部账户，但是它必须显式赋值，而不能直接在其他函数中调用，因为如果智能合约调用其他智能合约，那么这个地址就会指向智能合约自己，从而在某些场景下不被允许，所以要先显式赋值保存。然后写了一个查看总供应链，查看当前账户，查看当前账户地址的几个方法，都可以，所以最后能看到结果，只不过U256是默认16进制的。



#### 发行自己的ERC-721规范的NFT代币

NFT和ERC20不同的地方在于每个代币都会有一个独一无二的标记，所以不存在等价交换的概念，有点类似游戏王的卡牌，青眼白龙的价值就是更高。

ERC721和ERC20的代币，在数据结构上基本类似，不同在于ERC721每次通过mint铸造代币的时候，都要给铸造出来的代币做一个唯一标记，并且结构体内部还要维持一个计数器，这个计数器实际上会作为代币的ID，以确保每个代币的ID都是不一样的（感觉区块链是单线程的）。

此外NFT不可拆分，因此在设计参数的时候，不需要考虑小数点位数的问题。

每个代币是独一无二的，因此设计参数的时候需要考虑给出一个方法来实现这个独一无二性。

代码样例，和ERC20的差不多，也是一个main.rs一个lib.rs一个erc721.rs，具体来说：

```rust
use alloc::{string::String, vec::Vec};
use core::{marker::PhantomData, borrow::BorrowMut};
use stylus_sdk::{stylus_core::log, abi::Bytes, msg, prelude::*};
use alloy_primitives::{Address, U256, FixedBytes};
use alloy_sol_types::sol;

pub trait Erc721Params {
    const NAME: &'static str;
    const SYMBOL: &'static str;
    fn token_uri(token_id: U256) -> String;
}

sol! {
    event Transfer(address indexed from, address indexed to, uint256 indexed token_id);
    event Approval(address indexed owner, address indexed approved, uint256 indexed token_id);
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
    
    error InvalidTokenId(uint256 token_id);
    error NotOwner(address from, uint256 token_id, address real_owner);
    error NotApproved(address owner, address spender, uint256 token_id);
    error TransferToZero(uint256 token_id);
    error ReceiverRefused(address receiver, uint256 token_id, bytes4 returned);
}


#[derive(SolidityError)]
pub enum Erc721Error {
    InvalidTokenId(InvalidTokenId),
    NotOwner(NotOwner),
    NotApproved(NotApproved),
    TransferToZero(TransferToZero),
    ReceiverRefused(ReceiverRefused),
}

sol_interface! {
    interface IERC721TokenReceiver {
        function onERC721Received(address operator, address from, uint256 token_id, bytes data) external returns(bytes4);
    }
}

const ERC721_TOKEN_RECEIVER_ID: u32 = 0x150b7a02;

sol_storage! {
    pub struct Erc721<T: Erc721Params> {
        mapping(uint256 => address) owners;
        mapping(address => uint256) balances;
        mapping(uint256 => address) token_approvals;
        mapping(address => mapping(address => bool)) operator_approvals;

        uint256 total_supply;
        PhantomData<T> phantom;
    }
}

pub type Erc721ResultWrapper<T> = Result<T, Erc721Error>;

// private methods, not public to EOAs
impl<T: Erc721Params> Erc721<T> {
    fn _owner_of(&self, token_id: U256) -> Erc721ResultWrapper<Address> {
        let owner = self.owners.get(token_id);
        let r: Erc721ResultWrapper<Address> = if owner.is_zero() {
            Err(Erc721Error::InvalidTokenId(InvalidTokenId { token_id }))
        } else {
            Ok(owner)
        };
        r
    }
    pub fn _req_auth_to_spend(&self, from: Address, token_id: U256) -> Erc721ResultWrapper<()> {
        let owner = self._owner_of(token_id)?;
        if from != owner {
            return Err(Erc721Error::NotOwner(NotOwner {from, token_id, real_owner: owner}));
        }
        let msg_sender = self.vm().msg_sender();
        if msg_sender == owner {
            return Ok(());
        } else if self.operator_approvals.getter(owner).get(msg_sender) {
            return Ok(());
        } else if self.token_approvals.get(token_id) == msg_sender {
            return Ok(());
        } else {
            return Err(Erc721Error::NotApproved(NotApproved {
                owner, spender: msg_sender, token_id
            }));
        }
    }

    pub fn _transfer(&mut self, token_id: U256, from: Address, to: Address) -> Erc721ResultWrapper<()> {
        let mut owner = self.owners.setter(token_id);
        let old_owner = owner.get();
        if old_owner != from {
            return Err(Erc721Error::NotOwner(NotOwner {from, token_id, real_owner: old_owner}));
        }

        owner.set(to);
        let mut from_balance = self.balances.setter(from);
        let from_balance_old = from_balance.get();
        from_balance.set(from_balance_old - U256::from(1));

        let mut to_balace = self.balances.setter(to);
        let to_balance_old = to_balace.get();
        to_balace.set(to_balance_old + U256::from(1));

        self.token_approvals.delete(token_id);
        log(self.vm(), Transfer { from, to, token_id });
        Ok(())                            
    }

    fn _call_receiver<S: TopLevelStorage>(
        storage: &mut S,
        token_id: U256,
        from: Address,
        to: Address,
        data: Vec<u8>
    ) -> Erc721ResultWrapper<()> {
        if to.has_code() {
            let receiver = IERC721TokenReceiver::new(to);
            let msg_sender = msg::sender();
            let received = receiver
                .on_erc_721_received(&mut *storage, msg_sender, from, token_id, data.into())
                .map_err(|_e| Erc721Error::ReceiverRefused(ReceiverRefused {
                    receiver: receiver.address,
                    token_id,
                    returned: FixedBytes(0_u32.to_be_bytes())
                }))?.0;
            if u32::from_be_bytes(received) != ERC721_TOKEN_RECEIVER_ID {
                return Err(Erc721Error::ReceiverRefused(ReceiverRefused {
                    receiver: receiver.address,
                    token_id,
                    returned: FixedBytes(received)
                }));
            }
        }
        Ok(())
    }

    pub fn _safe_transfer<S: TopLevelStorage + BorrowMut<Self>>(
        storage: &mut S,
        token_id: U256,
        from: Address,
        to: Address,
        data: Vec<u8>
    ) -> Erc721ResultWrapper<()> {
        storage.borrow_mut()._transfer(token_id, from, to)?;
        Self::_call_receiver(storage, token_id, from, to, data)
    }

    pub fn _mint(&mut self, to: Address) -> Erc721ResultWrapper<()> {
        let new_token_id = self.total_supply.get();
        self.total_supply.set(new_token_id + U256::from(1u8));
        self._transfer(new_token_id, Address::default(), to)?;
        Ok(())
    }

    pub fn _burn(&mut self, from: Address, token_id: U256) -> Erc721ResultWrapper<()> {
        self._transfer(token_id, from, Address::default())?;
        Ok(())
    }
}

#[public]
impl<T: Erc721Params> Erc721<T> {
    pub fn name() -> Erc721ResultWrapper<String> {
        Ok(T::NAME.into())
    }
    pub fn symbol() -> Erc721ResultWrapper<String> {
        Ok(T::SYMBOL.into())
    }
    #[selector(name = "tokenURI")]
    pub fn token_uri(&self, token_id: U256) -> Erc721ResultWrapper<String> {
        match self.owner_of(token_id) {
            // this leaves to the implementation side
            Ok(_) => Ok(T::token_uri(token_id)),
            Err(e) => Err(e)
        }
    }
    pub fn owner_of(&self, token_id: U256) -> Erc721ResultWrapper<Address> {
        self._owner_of(token_id)
    }
    pub fn balance_of(&self, owner: Address) -> Erc721ResultWrapper<U256> {
        Ok(self.balances.get(owner))
    }
    pub fn safe_transfer_from_with_data<S: TopLevelStorage + BorrowMut<Self>>(
        storage: &mut S,
        from: Address,
        to: Address,
        token_id: U256,
        data: Bytes
    ) -> Erc721ResultWrapper<()> {
        if to.is_zero() {
            return Err(Erc721Error::TransferToZero(TransferToZero { token_id }));
        }
        storage.borrow_mut()._req_auth_to_spend(from, token_id)?;

        Self::_safe_transfer(storage, token_id, from, to, data.0)
    }

    pub fn transfer_from(&mut self, from: Address, to: Address, token_id: U256) -> Erc721ResultWrapper<()> {
        if to.is_zero() {
            return Err(Erc721Error::TransferToZero(TransferToZero { token_id }));
        }
        self._req_auth_to_spend(from, token_id)?;
        self._transfer(token_id, from, to)?;
        Ok(())
    }

    pub fn approve(&mut self, approved: Address, token_id: U256) -> Erc721ResultWrapper<()> {
        let owner = self.owner_of(token_id)?;
        let msg_sender = self.vm().msg_sender();
        if msg_sender != owner && !self.operator_approvals.getter(owner).get(msg_sender) {
            return Err(Erc721Error::NotApproved(NotApproved {owner, spender: msg_sender, token_id}));
        }

        self.token_approvals.insert(token_id, approved);
        log(self.vm(), Approval { approved, owner, token_id });
        Ok(())
    }

    pub fn set_approval_for_all(&mut self, operator: Address, approved: bool) -> Erc721ResultWrapper<()> {
        let owner = self.vm().msg_sender();
        self.operator_approvals.setter(owner).insert(operator, approved);
        log(self.vm(), ApprovalForAll { owner, operator, approved });
        Ok(())
    }

    pub fn get_approved(&mut self, token_id: U256) -> Erc721ResultWrapper<Address> {
        Ok(self.token_approvals.get(token_id))
    }

    pub fn is_approved_for_all(&mut self, owner: Address, operator: Address) -> Erc721ResultWrapper<bool> {
        Ok(self.operator_approvals.getter(owner).get(operator))
    }

    pub fn supports_interface(interface: FixedBytes<4>) -> Erc721ResultWrapper<bool> {
        let interface_slice_arr: [u8; 4] = interface.as_slice().try_into().unwrap();
        if u32::from_be_bytes(interface_slice_arr) == 0xFFFFFFFF {
            return Ok(false);
        }

        const IERC165: u32 = 0x01FFC9A7;
        const IERC721: u32 = 0x80AC58CD;
        const IERC721_METADATA: u32 = 0x5B5E139F;
        Ok(matches!(u32::from_be_bytes(interface_slice_arr), IERC165 | IERC721 | IERC721_METADATA))
    }
}
```

lib.rs：

```rust
#![cfg_attr(not(feature = "export-abi"), no_main)]
extern crate alloc;

mod erc721;
use erc721::{Erc721, Erc721Params, Erc721ResultWrapper};
use stylus_sdk::prelude::*;
use alloy_primitives::{U256, Address};

struct NFTParams;
impl Erc721Params for NFTParams {
    const NAME: &'static str = "StylusNFT";
    const SYMBOL: &'static str = "SNFT";
    fn token_uri(token_id: stylus_sdk::alloy_primitives::U256) -> String {
        format!("{}{}{}", "https://my-nft-metadata.com/", token_id, ".json")
    }
}

sol_storage! {
    #[entrypoint]
    struct MyStylusNFT {
        #[borrow]
        Erc721<NFTParams> token;
    }
}

#[public]
#[inherit(Erc721<NFTParams>)]
impl MyStylusNFT {
    pub fn mint(&mut self) -> Erc721ResultWrapper<()> {
        let minter = self.vm().msg_sender();
        self.token._mint(minter);
        Ok(())
    }

    pub fn mint_to(&mut self, to: Address) -> Erc721ResultWrapper<()> {
        self.token._mint(to);
        Ok(())
    }

    pub fn burn(&mut self, token_id: U256) -> Erc721ResultWrapper<()> {
        let msg_sender = self.vm().msg_sender();
        self.token._burn(msg_sender, token_id);
        Ok(())
    }

    pub fn total_supply(&mut self) -> Erc721ResultWrapper<U256> {
        Ok(self.token.total_supply.get())
    }
}
```

main.rs：

```rust
#[cfg(feature = "export-abi")]
fn main() {
    nft_demo::print_abi("MIT-OR-APACHE-2.0", "pragma solidity ^0.8.23;");
}
```

另外注意依赖库的配置，虽然stylus-sdk内部包含了后面2个，但是不能直接引入，还是要直接依赖后面这2个才可以：

```toml
[dependencies]
stylus-sdk = "0.8.0"
alloy-primitives = "0.8.20"
alloy-sol-types = "0.8.20"
```

验证，部署完成后再测试一下，可以发币，可以查询总量增加，就证明合约部署成功了。

NFT的一些要素，玩法，规范等，后面再补充。



#### 实现A合约调用B合约

合约之间互相调用是非常常见的行为，因为智能合约之间互相协作可以扩大功能，产生新的能力，从而支持在链上完成更复杂的行为。

合约之间调用需要注意几个细节：

- GAS问题，如果A合约调用B合约，那么也是需要产生GAS的，即EOA发起交易，触发A合约，需要支付A合约的GAS，然后A再调用B，B也需要GAS才能完成交易，这样EOA用户就需要同时支付A+B的GAS费用
- 目标合约的ABI获取，一定要确认目标合约支持的能力，有几个方法，一是直接拿到目标合约的开源源码，然后本地生成ABI，或者用区块链浏览器去查询这个合约公开的ABI，或者获取到这个合约的WASM，然后反编译拿到ABI，总之一定要确认目标合约的ABI，如果不确认，也就谈不上调用了，这里后续都假设开发者拿到了目标合约的ABI
- 环境问题，即delegate_call，委托调用，这个是必要的，因为支持合约升级，委托调用表面上看，是B合约在A合约的环境内执行，这样支持修改A合约的状态，实际上，是A合约负责存储状态，B合约负责执行业务，这样B合约可以不断升级（通过修改A合约指向的B合约的地址，修改到升级后的B合约的地址），这样B合约升级后依然只负责业务，A合约还是负责状态。一般来说合约调用应该是A合约调用B合约时，各个业务在各个合约的环境内执行就可以。此外A合约还需要拿到B合约的执行结果，以便继续往下执行，这就需要B合约的公开方法一定要有返回值，即使是单元类型的

官方推荐的做法是使用`sol_interface!`宏定义当前合约会涉及的目标合约的ABI，目前研究到的做法是：

- 通过sol_interface!宏定义外部合约的ABI，此处是solidity语法
- 自己合约的结构体需要有一个方式保存外部合约的地址
- 每次调用外部合约的时候，通过外部合约接口（实际上是结构体，里面有一个address字段）构造外部合约实例，然后通过这个实例去调用方法
- 如果涉及到GAS和代币转移，外部合约一般都会在ABI内声明payable，此时可以在内部通过`Call::new_in()`设置最大支付的GAS费用和转移给对方合约的GAS费用，这块还没有研究清楚，后面可以具体看一下

view函数，pure函数和write函数，这些是合约开发的时候为了降低GAS费用而给函数做的区分，简单来说**任何调用合约的行为都会产生GAS费用**，那么为了降低GAS开销，SDK已经自动帮开发者优化了。首先通过impl结构体可以给合约添加函数或者方法，如果是方法，第一个入参如果是&self，那么编译出来就视为view函数，不修改状态，如果修改状态，那么第一个入参是&mut self，就会导致更高的GAS，pure函数就是RUST里面的关联函数，没有&self，就是客观理解的纯函数。



#### 实现ERC4626智能金库

ERC4626也是以太坊社区指定的规范，用于代币的金库（vaults）设计，它是基于ERC20的拓展，即保留了基础的代币发行转移能力，此外还支持储蓄，质押，投资等操作。

简单来说是这样的，既然加密货币也是货币，那么也就应该具有一般货币都有的特点，即可以存入银行获取利息收入，参与到借贷系统中。ERC4626就是对标传统的借贷业务的规范，它的核心是金库。

金库是一个链上的团队和其他EOA进行交互的中介，一般都会通过智能合约完成，相当于在链上可以开银行进行借贷投资服务。ERC4626只是规定了整个银行系统的玩法中，非常大类和确定的行为，比如代币转为金库债权，收益结算，赎回等。但是作为传统银行一般都会要做投资来扩大收益，**ERC4626并没有明确规定金库必须进行投资**。实际上商业化的金库一定会有投资环节，即使不做任何事情，维持智能合约的运转也需要GAS，因此现实中一旦存入代币，总资产一定会随着基础运营不断缩水，所以不去投资是不行的。

所以一个完整的ERC4626智能合约，需要包括以下功能：

- 接收代币，转为金库份额（债权）。为此需要在链上发行金库份额
- **智能合约接收的代币视为金库资产**，所以用户存入代币，金库的资产会增加，用户赎回代币，金库的资产会减少
- 支持用户赎回代币，当然因为涉及到投资所以赎回可能有门槛，赎回后用户的债权需要销毁
- 转换操作，代币转为金库份额的逻辑，以及金库份额转回代币的逻辑
- 查询，比如当前债权可转回份额的查询，当前代币可转为金库份额的查询，总资产查询等等
- **一个金库只能管理一种资产，即只能接收一种现有的ERC20代币**，既然代币是现有的，那么它在链上的资产地址也是现有的

投资的部分需要由金库团队链下完成。所以这里要实现的ERC4626智能金库功能，就不会包括投资的部分，智能合约也没办法去写死投资逻辑。

具体实现流程如下：

1. 实现一个完整的ERC20规范的代币合约，需要包括完整的ERC20的所有方法
2. 实现一个金库合约，需要实现ERC4626的规范
3. 部署ERC20代币合约，获取到它的地址
4. 部署金库合约，并把它的代币合约地址设置到之前的ERC20的地址
5. ERC20合约发行代币到测试账户
6. 测试账户拿这个代币存入金库，获取对应筹码
7. 金库提供模拟的投资收益方法（可以增加，减少资产份额，这个环节在生产流程中需要金库运营团队在链外完成）
8. 测试账户用筹码赎回代币，根据金库当时的份额换回对应的ERC20代币（严格来说金库的投资收益和ERC20代币的发行也有关系，这里就不搞那么细致了）



目前限制，合约需要在部署时设置好资产对应代币的智能合约地址，这样它才能调用对方合约来进行转账操作，目前还没找到对应的方法，后续可能会优化吧。

限制2，无法搞定其他智能合约的问题，这里要实现的是自己写一个ERC20的代币，可以不依赖ETH，因此可以不包含#[payable]注释的方法，这种场景下如何调用其他智能合约的方法？需要再用一个例子来实现，所以**在完成这个智能金库的功能之前，需要先实现一个A合约调用B合约的DEMO**。
