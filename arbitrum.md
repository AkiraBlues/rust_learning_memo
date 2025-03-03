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

然后是在linux环境下操作了，先安装rustup，然后是VS CODE，之后是GIT，其他相关所需的，安装CARGO-STYLUS的时候提示缺了很多东西，然后慢慢补充，最后终于构建成功，有了CARGO-STYLUS工具链，之后再引入一个DEMO应用，然后编译构建，终于成功了。

之后流程参考Arbitrum官方的教程，流程是这样的：

- RUST开发环境，即RUST相关工具，环境，和IDE，这部分之前已经提到
- Docker，这部分后面会介绍
- Foundry CLI，用于和EVM合约进行交互
- Nitro开发节点，需要从GITHUB上下载一个开发节点项目并启动它，它的脚本会触发docker相关行为

Docker介绍，它是一个公开的平台，可以用来开发应用。Docker可以在一个沙箱中构建和运行应用，甚至在多个沙箱中做同样的事情。沙箱非常轻量，它包含了运行应用所需的所有环境，因此不需要直接在宿主环境（一般指个人用开发电脑）上安装对应的开发环境。一个最简单的过程就是通过Docker搭建一个CI / CD环境，这样每次代码提交后都可以直接在Docker上进行自动代码审核，打包构建，测试，发布等一系列流程，而不用去在自己的电脑上直接搭建这样的环境。

Docker安装过程记录：

- 确认OS，linux下需要开启KVM虚拟化，由于我是通过虚拟机跑的Ubuntu，因此实际上是通过虚拟机的虚拟化去支持KVM虚拟化，宿主Windows11环境本身需要调整一下，然后是虚拟机开启对应虚拟化配置，然后才是Ubuntu自身安装一些软件去开启KVM虚拟化，并授权给当前用户
- 之后参考Docker官方教程去安装，还是通过下载deb包的形式安装
- 然后在设置里面的docker engine修改一下国内镜像源，写法是`"registry-mirrors": ["https://some-domain.com",
  "https://some-domain2.com"]`，这里要改成实际的镜像源
- resource里面改一下memory limit，最少要4G

然后就开始涉及DOCKER最重要的2个概念，container（容器）和image（镜像），首先要下载镜像，就是操作OS，它是静态的，不可修改的，然后基于这个静态的镜像可以创建多个运行实例，就是容器，每个容器都是动态的，可以修改的，而且各自独立。

DOCKER安装好之后可以在gitbash或者CMD使用`docker --help`来确认它安装好了，并且可以查询各种命令。

之后在DOCKER的主界面可以打开终端，然后执行以下命令，会开始下载镜像并创建容器：`docker run -d -p 8080:80 docker/welcome-to-docker`，会先尝试加载本地镜像，失败后从镜像源下载对应镜像，然后创建容器，如果一切顺利，运行后，打开浏览器，输入`localhost:8080`就可以看到一个页面，表示容器运行成功。

安装Foundry CLI，先启动gitbash，确保可以使用curl命令，然后执行`curl -L https://foundry.paradigm.xyz | bash`，这个是用来下载foundryup（类似rustup），如果出现握手错误或者其他网络连接错误，设置一下VPN，确保`foundry.paradigm.xyz`要走代理，然后安装，成功后重启一下gitbash，然后执行`foundryup`，如果成功会看到FOUNDRY标题和下载安装进度条，安装完成后执行一下`foundryup -v`来查一下版本号，如果有就是真的装好了。

安装Nitro开发节点，这个需要在一个文件夹里面打开gitbash，因为需要把对应代码下载到这个位置，执行`git clone https://github.com/OffchainLabs/nitro-devnode.git .`，注意最后这个点，它表示把代码下载到当前所在文件夹，而不是创建新文件夹，之后执行`./run-dev-node.sh`，这个脚本会尝试通过docker相关命令去调用DOCKER去下载镜像创建容器，然后部署一个样例智能合约。如果一切顺利，可以从DOCKER那里看到有一个容器正在运行，说明脚本执行成功。

还有几个工具需要安装，先安装cargo-stylus，它是一个CLI，用于辅助开发stylus智能合约：

```
cargo install --force cargo-stylus
```

如果安装失败，提示rustc版本过低，则执行`rustup self update`和`rustup update`更新一下所有的工具。

安装完成后执行`cargo stylus --version`，确认是否能出结果。

然后可以创建一个hello world项目了，在项目空间里面执行`cargo stylus new <PROJECT_NAME>`。创建项目后会下载模板代码，然后不着急运行，先在项目内执行`rustup target add wasm32-unknown-unknown`，来往当前项目的RUST的构建目标内增加对WASM的支持，这里不指定rustup工具链为具体数字版本，采用默认的stable版本，如果后续出现兼容性问题，再修改。

之后启动Nitro开发节点，即先打开DOCKER，然后执行开发节点项目的`./run-dev-node.sh`，确保容器在运行。

之后再执行`cargo stylus check`，检查项目的合约情况，Ubuntu环境下应该能编译通过并进行检查。

倒退，中间遇到sda3分区撑满了的情况，用了各种办法删除硬盘空间，但是估计删除了一些必要的东西，导致后面启动的时候一直无法联网，试了很多办法搞到很晚都没用，因此只能挥泪删除虚拟机分区然后重来了，哎又浪费了1天。





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

