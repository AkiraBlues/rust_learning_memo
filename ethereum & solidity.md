## Ethereum & Solidity learning memo

专门介绍以太坊 / Solidity相关知识点。



#### 入门

以太坊（官网点[这里](https://ethereum.org/en/)），是一个分布式虚拟机网络，开发者可以在上面部署DApps。2015年以太坊正式启动。

以太坊和比特币最大的区别是，比特币是去中心化数据库，而以太坊是去中心化虚拟机，而且它的虚拟机是支持运行Solidity编程语言的，这个语言和现代编程语言类似，专为智能合约设计，所以EVM通过支持智能合约从而支持DApps。

智能合约是一段预先设定好规则的程序，触发条件后会自动执行，且程序本身必须完全开源，以方便所有人审查是否有黑箱操作

另外以太坊通过POS而非POW的共识机制来创建区块，使得交易可以更快得到确认，因此效率比比特币更高一些。

分布式网络实际上还是单例模式的，即以太坊只有一个状态对象，一个单例，当某个节点广播了状态变化时，它最终会传播到整个网络，所以**从长期来看任何节点运行的虚拟机都是和单例状态保持一致的**。

虽然以太坊的代码执行效率不高，但是它的全球单例模式还是比较有新引力的。

实际上以太坊在2015年初期创建时还是基于POW机制的，改为POS要到**2022年，进入以太坊2.0时代**。

开发者或用户接入以太坊，都需要创建账户，这里有2种类型：

- EOA，Externally Owned Accounts，外部账户，即一般的私人账户，由真实的人类进行控制，只能进行转账
- CA，Contract Accounts，合约账户，通过智能合约创建的账户，由智能合约进行控制，由于智能合约的程序逻辑是写死的，因此不能基于人的行为来干涉，但是**它也不具备自启动特性，需要由外部账户来手动启动**

以太坊账户包含以下字段：

- nonce，如果是外部账户，它表示交易次数，新账户默认是0，如果是合约账户，它表示执行合约操作的次数，默认是1，表示合约账户被创建了
- balance， 余额，合约账户也可以有余额，比如如果需要空投，那么这个过程可以是，创建一个合约账户，给它设定好空投规则，并充值，此时合约账户也有余额
- codeHash，只针对合约账户存在，即它包含的以太坊虚拟机（EVM）的代码的哈希值，对于外部账户，此字段是空字串，哈希这个工具是防止合约账户的代码被篡改，一般来说合约账户一旦部署，其代码原则上不能更改，但是以太坊也允许在特定条件下修改代码
- storageRoot，以太坊也用到了Merkle Patricia Tree结构来存储数据，因此每个账户的数据都会对应一个这个树结构，这个字段用于记录树的根节点的256位哈希值，这样基于哈希来找到存储根节点，从而查询账户数据，就会比较快

此外，外部账户还拥有私钥和公钥，用于确认这个账户是由一个真实的人类来控制的，而合约账户没有私钥和公钥，因此它的地址也不能基于公钥来运算，而是基于部署它所对应的智能合约的负责人的钱包地址和一个nounce值来确定的。



#### 共识机制

这个还是在bootcamp的课程里面看到的比较好的说明。

在一个去中心化的网络中，各个节点之间的协作是一个痛点，比如怎样确保各个节点和全局状态保持一致（比如某个交易导致了总供应量的变化，各个节点如何接收到这个信息），如何新增区块，如何确保交易不是伪造的，这些都需要共识机制的参与。

共识机制是一组原则，最核心的原则是：

- 一个代币不能用于支付2笔交易，即消费2次
- 最长链原则，即如果出现分叉了（通常是由于POW的矿工竞争机制导致同时创建了2个区块），**区块链会暂时进入分叉状态，由后续区块来选择哪个分支更长**，因为还是分布式网络，部分节点会选择分叉A（因为它先收到了分叉A的广播），部分节点会选择分叉B（同理，它先收到分叉B的广播），但如果后续又有新的区块出现了，它如果加入到分叉A，那么意味着分叉B是更短的链，分叉A是更长的链，此时分叉B的所有交易会被视为无效，而回滚，分叉A会被视为主链

- 单独验证，即如果全局状态产生了变化，通常是因为有了交易，可以基于同一套验证方式，让各个节点单独验证这些交易是否有效（有点类似科学发现，A小组发表了试验过程和结果，其他小组按照A小组的步骤去验证，只不过这里的步骤是先于区块链成立的，所有节点都要遵守）

- 51%认同，即随着网络传播，这个状态变化被一个个的节点进行验证，当总体验证数量达到了51%，那么这个变化就是全局认可的，当然未确认的节点也是需要继续做确认的，一般来说不会有问题，如果出现冲突，就需要基于最长链原则来解决，这个也是共识机制的一部分

POW也是一种共识机制，但它不是最核心的，因为还有POS，POH等等。这些共识机制是用于决定如何创建新区块的，比如POW就是纯粹的矿工竞争，谁运气好算力高，谁就能解开数学题从而确认交易，形成新区块，POS则是看谁有门票，谁运气好，POH则是在过程中加入了时间戳，以标记交易的先后顺序。

另外POW中挖矿难度也是基于自动化算法控制的，即根据过去一段时间的新区块产生速度，来调整当前的难度，简单来说如果挖矿的人少，难度就会下降，如果人多，难度就会上升，这一切都是通过一个算法自动控制的，这个算法也是比特币创建者中本聪在一开始的白皮书里面提到的，后面他也给出了算法细节。



#### 挖矿和区块补充

挖矿是确认交易，形成新区块的过程，通过消耗算力来增加DDOS或者伪造区块的成本。

挖矿过程也是算法自动执行的，人的作用是准备好硬件软件环境，设定好参数和挖矿策略，之后启动程序，然后剩余过程就是自动化的了。程序一般由区块链核心团队负责，比如最早的挖矿程序就是比特币的，由中本聪设计，当然随着行业发展，第三方也可以参与编写。

挖矿核心的操作是基于一组选定的交易，去进行哈希计算，虽然不带入SALT，但是会不断修改NONCE（这里矿工可以选择NONCE的变动策略，最简单的就是从0开始递增，也可以选择随机数策略）来生成不同的哈希，最后以使得哈希值低于难度目标，一旦低于，那么此矿工就可以构建区块并广播出去了。

实际影响哈希计算的几个入参：

- 上一个区块头的哈希值，一般会放在区块头里面
- NONCE，这个会不断修改
- 当前待确认交易的默克尔根，即当前交易构成的哈希树的根节点的值，所以受到交易池选择的影响，矿工也可以修改这部分

注意哈希计算是把当前区块的信息作为一个整体的，不是单独修改NONCE值后单独只计算NONCE的哈希值。

一般来说都会对区块的哈希值有要求，不合法的哈希创建出来的区块不会被其他节点认可。

区块链的所有区块组成链表结构，头节点，一般也被称为创世区块（genesis block），只有它的头部不会包含上一个区块的哈希值。

另外比特币没有BURN机制，也没有基础GAS费用，所以目前支撑全球的完整节点的主要是矿工，因为搭建和维护一个完整节点需要消耗电力，而为此需要进行挖矿以填补搭建和维护成本，比特币的矿工奖励除了新区块产生的基础奖励外，确认的交易也会包含小费，或者交易费，这些费用也会全部奖励给矿工。当所有比特币都被挖出后，新的区块依然可以产生，只是不会有挖矿基础奖励了，但是矿工依然可以通过确认交易获得手续费，因此这也会激励矿工去继续维持全球各地的完整节点。

比特币的区块头包含一个默克尔树的root值，**它对应的是当前区块的所有交易的哈希树根，不包含所有区块的所有交易**。

默克尔树就是二叉树的一种，它的父节点是子节点的值拼接在一起的哈希值，即`parentNode = Hash(left + right)`，然后重复这个过程就会得到根节点，用于数据完整性校验，因为对任何一个节点的篡改都不够，还需要篡改到它的所有祖先节点。



#### POS

2022年以太坊从POW切换到POS，这个是一个硬分叉（后面会提到什么是分叉，这里简单理解为软件升级，需要各个节点去选择升级还是不升级，因此这个选择的过程和参与就是一次分叉）。

POW依赖矿工通过算力竞争来构建新区块，而且POS不依赖这个，而是依赖参与者的质押，通过质押32个ETH，并设置好一个不断运行的程序就可以了，每12秒，EVM的智能合约会选出一个当前的参与者，参与者运行的程序收到消息后，**就会自动开始收集交易并打包签名的过程**，在此过程中参与者不需要手动处理，参与者完成后进行广播，当其他的参与者确认后交易就成立，之后参与者获得奖励。

所以从这个过程看POS最大的作用是大大缩短了交易确认的时间，使得以太坊的生态可以支持更多的交易（比特币公链需要10分钟来确认交易，因为矿工挖矿就是要挖这么久）。

POS内的验证需要其他参与者的参与，以太坊设计了一套机制以规范验证者和其他参与者的行为，简单来说如果验证者和其他参与者都规范行为，那么不会有问题，如果验证者试图作弊，其他参与者通常可以通过验证签名或者其他信息来证明他作弊，从而验证者受到惩罚。如果其他参与者中存在故意捣乱的人，比如故意说验证者是作弊，那么在经过多轮投票或者轮流举证等程序后，一般可以确认出这个人是故意捣乱的，因此他也会受到惩罚。

由于POS的这种机制，它允许一定程度上的质疑和回滚，因此一个区块产生后，可以用一个安全程度来衡量它，即抵抗回滚的程度，比如创世区块，100%不可能回滚，注意定义顺序，是从最安全到最不安全的：

- earlist，表明这个区块是创世区块。
- finalized，这个涉及到几个概念，以太坊每隔12秒创建一个区块，因此这个12秒就是一个slot，间隙，而32个slot等于1个epoch，即一个epoch里面包含了32个区块，其中以太坊会选择一个区块作为检查点，当2/3的POS参与者确认这个检查点后，它的状态就变为了justified，那么finalized的就表示这个区块所在的epoch之后还有至少一个新的epoch，而且那个新的epoch内的checkpoint已经是justified的状态了，简单来说就是finalized表示当前区块后面还有至少一个epoch的区块，且那些区块和当前区块都已经是justified的状态，**有至少一个justified的垫背epoch，那么当前epoch内所有区块都是finalized**。
- safe，还是涉及到这个epoch的概念，以太坊的每12秒创建区块是强制性的，因此每个epoch等于32个区块也是强制性的，即使某段时间没有交易，区块还是会产生（只是不包含交易），因此当一个epoch填满32个区块后，POS参与者需要进行一次投票以确认这个区块是justified或者不是，如果投票成立（通常需要2/3赞成），那么这个epoch内所有区块都是justified状态，此时，如果后续的epoch还没有填满因此无法投票，没有垫背状态时，当前这个最后一个justified的epoch内区块就都是safe标记的。它标记为safe不代表绝对安全，也可能会被回滚。
- latest，最新的区块，有更大可能会被回滚。
- pending，创建中的区块，还没有被广播和确认。

当请求交易数据的时候，需要指定查询的区块范围，此时这个区块安全标记就很有用了。



#### 区块的数据存储

以太坊的区块使用Patricia Merkle Tree（以下简称PMT）来存储数据。

Patricia Tire是前缀树，**它和默克尔树不同，它不是二叉树**，把所有字串抽离公共部分作为根部，然后不断分叉，就形成了前缀树，换言之从根到叶子的所有路径上节点拼接到一起，就会构成一个完整的字串。

前缀树+默克尔树，产生PMT，PMT用于保存key--value结构（也就是Map结构）的数据，为了高效存储和保持数据完整性校验，每个节点保存2类信息：

- 部分key，用于保存key，这里用前缀树处理，因此从根到叶子节点的所有key组合起来才是一个完整的key
- value / hash，如果是叶子节点，则它保存完整的value，如果是非叶子节点，则它用于完整性校验，它保存的是它的子节点的值合并后的哈希值，PMT是非二叉树结构，因此一个节点的哈希值可以是由它的多个子节点的值合并计算的

以太坊之所以采用PMT来保存数据，是因为以太坊内的账户结构数据经常会发生变动，而PMT结构很适合这种场景，考虑2个行为：

- 更新一个key对应的value，只需要更新从根节点到对应的key保存的value的叶子节点上所有的节点就可以，由于key没有变化，因此先基于key前缀查找到value，然后更新它，然后找到它所有的父节点，重新计算对应哈希即可
- 新增一个key和对应的value，只需要从根部开始往下，找到key的分叉点，新增一个分叉，并增加值就可以，因为PMT不受限于二叉树，分叉可以有多个

在以太坊的区块头中，通过3个关键字段来保存核心信息，它们分别是：

- state root，世界状态树根
- transaction root，交易记录根
- receipts root，回执根

state root，世界状态树根，世界状态树是PMT结构，它的key是账户，value就是账户状态。由于区块链的状态是经常变化的，因此每个区块的state root，实际上体现的是它作为最新的区块的时候的世界状态，一旦新的区块产生，最新的世界状态就会保存到那个新区块上，而之前区块的区块头记录的状态就只是历史状态了。这个过程有点类似REACT的time capsule功能，就是把各个时间切面的状态保存下来，而不是基于一个状态不停地mutate。

世界状态树的完整信息另外存储于以太坊虚拟机的数据库内，一般是LevelDB或者RocksDB，这样做可以减少区块本身的存储压力。

transaction root，当前区块所有交易的交易记录构成的默克尔树的根节点哈希值，由于一个区块在形成后，其交易记录不再变动，因此这个字段也不会发生变化，所以直接用默克尔树记录就可以了。

receipts root，当前区块的所有交易日志构成的默克尔树的根节点哈希值，回执就类似日志，和交易记录根一样，一旦区块形成也不再变动，所以也是用默克尔树记录就可以。



#### 不对称加密补充

传统加密通信都是对称的，也就是说**A和B进行通信之前，A必须要通过其他渠道告知B，A使用的密钥**。这里存在信任问题，因为A和B共享一套密钥，**如果A不相信B，是不可能把密钥共享给B的**，所以A要和B建立信任关系。

不对称加密就是用来解决这个问题的，它本质上解决了互信问题，A和B通信之前，A可以不需要通过其他渠道和B接触，A不需要信任B，**A只需要假设B持有B的私钥即可**，因为不对称加密有2种场景：

- 在通信场景下，A用B的公钥加密消息，B收到消息后用B的私钥解密，假设A的消息是广播出去的，那么也只有B可以解密，因为只有B有B的私钥
- 在证明场景下，A用自己的私钥加密，然后广播出去，所有人都可以用A的公钥来解密，从而证明这个信息是A创作的

主流的不对称加密方式是RSA和ECDSA（椭圆曲线），目前ECDSA更主流因为使用开销更小。



#### 以太币

以太币，也叫ETH，是以太坊生态内的货币，1ETH等于10的18次方的WEI。

最初的以太币是在2014年通过众筹的方式，参与者通过比特币进行交换的，即它最初的价值是比特币背书。

之后以太坊参考比特币的POW机制，每创造一个新的区块，都会发行新的以太币以便给矿工提供奖励。

2022年升级到POS之后，同样还是每创建一个新的区块，创建者会获得新发行的ETH作为奖励。

由于交易需要GAS，而每次交易都会有基础GAS，而**这部分在交易完成后，不会作为任何人的奖励，而是会被销毁，即交易发起者，每次交易都会因为基础GAS导致轻微的资产损失，因此GAS的设计会轻微降低ETH的流通量**。用于控制以太坊生态的通货膨胀。



#### EVM

EVM就是运行智能合约代码的环境，和JVM有点类似，但是JVM不同的是，运行EVM的大部分代码都会产生GAS费，即EVM设计出来时，就对基础的逻辑运算进行了各自的GAS费定义，所以**运行一个无限循环的程序会导致GAS费不断增加**，最终使得它无法在任何一个节点上运行。

虽然加减乘除之类的运算（还有一些其他的数学运算）会导致GAS开销，但是**实际的GAS费用是GAS量和GAS单价的乘积**，而以太坊在运行中可以不断调整GAS单价，因此同一段代码在不同时期运行，也会产生不同的GAS费用。

另外涉及到存储空间的CRUD操作都会产生GAS，即使是查询，如果查询的是存储空间的数据，也需要GAS。

当然以太坊的设计者们也知道，预先设定好静态的GAS量可能会和实际运行的预期不符合，而且区块链上线后也要面临类似DDOS攻击的问题，因此**以太坊应该要具有自我升级的能力**，所以他们让以太坊支持了这个能力，通过分叉的方式。

**这里的分叉指的是形而上的分道扬镳**，以手机软件升级为例，当所有用户都收到升级通知时，大部分用户会选择升级，而少部分用户不会，此时**这个所有用户参与的决策行为，就是一个分叉**。不升级的用户，保持原有功能的使用，但是将来可能会面临功能越来越少甚至下线的问题，而升级的用户则紧跟趋势。EVM升级也是同样的道理。

EVM本质上是一个规范，类似DOM或者BOM，具体实现可以灵活，因此出现了很多对EVM的实现的软件，这些软件统称为EVM客户端。以太坊公链用的实现也有好几个选择，就是说当一个人选择搭建完整节点的时候，他也可以从几个实现中选择用哪个EVM。L2链为了和EVM兼容也会选择一些实现。**不同节点选择不同的EVM客户端，可以防止对某类特定EVM客户端的漏洞攻击传播到所有节点**。

当需要升级时，这些EVM客户端的开发团队会给出对应的更新版本，然后各个节点可以选择更新或者不更新（这里就是分叉点），一般来说大部分节点都会更新，但是也有不更新的，比如当以太坊从POW转向POS的时候，之前具有强大算力的节点可能会觉得自己的利益受损了，因为POS的参与者变得更随机，而不是之前的纯依靠算力之间的竞争，因此部分POW爱好者会选择不升级以留在POW，这些节点后面创建了ETHW，当然从结果来看不跟紧官方步伐就导致了ETHW的价格一直都很低，而且用户基数也在不断减少，所以可以说本质上分叉点虽然给各个节点提供了选择自由，但**其实是出于自由经济规则下的强制升级行为**。

对于不升级的节点，当收到一个升级后发出的交易广播时，由于它不具有对应的处理能力，通常会直接拒绝。

如果一个升级不涉及特性，只涉及安全，一般这种升级（或者叫分叉）就是软分叉，而改变特性的升级就是硬分叉。最有名的一次硬分叉发生在2016年，当时一个对某个DAO的攻击导致了以太坊的最大分叉，ETH选择回滚以保护DAO和相关用户的利益，而ETC选择了不回滚以遵循“区块链不能被篡改”这一原则。到目前ETH的市场和市值已经远远超过了ETC，所以估计ETC后面会被淘汰。



#### 智能合约和EVM

智能合约就是运行在以太坊链上的程序。

合约账户用于创建和管理智能合约。每个合约账户可以创建多个智能合约，它自身的nounce值会随着合约操作而不断增加，但是只能部署一个智能合约，因此**每个新创建的智能合约，如果需要部署，则必须部署到新的合约账户上**。

所以合约账户和部署的智能合约是一对一的关系，即**每个合约账户只能部署一个智能合约，因为它内部只能存储一份合约代码**。

合约账户虽然没有公钥，地址由部署智能合约的人和nounce值确定，但**它和外部账户一样，一旦创建，地址就不可变了**，部署的智能合约也就和这个地址绑定了，如果其他人要使用它发起操作，则所有人也都必须把消息发送到它部署在的这个合约账户的地址。

常见的智能合约，一般会有代币发行和管理的能力，其他的合约甚至还可以实现P2P借贷，或者支持多方签名。在去中心化组织内，通过智能合约还可以发起投票，比如发起者通过智能合约发起投票议题和投票选项，其他参与者进行投票和确认，最后智能合约给出投票结果等等。

智能合约本质上就是部署在EVM上的开源代码，任何人都可以监视它的客观公正性，以及查看它所支持的能力。它就像一个API接口，用户发起使用请求，它就会按照这些编写好的程序给出响应。开发智能合约可以用多种语言，经过编译之后成为字节码，最后部署到EVM。

EVM就是以太坊虚拟机，正常小白用户使用以太坊是不需要涉及的，因为直接用DApps就可以，而如果需要开发智能合约，则需要本地部署EVM，以降低开发成本，提高开发效率，相当于本地搭建开发和测试环境。

另外除了交易会需要GAS外，**执行智能合约也需要支付GAS**，因为它本质上也是运行在各个去中心化节点上，也需要消耗算力和电力。

**智能合约只被允许访问链上数据，包括它自身和其他智能合约占用的Storage，不能访问链下数据，比如第三方服务器等等**。



#### GAS

执行代码所需的GAS量是确定，但是以太坊可以通过升级来新增操作代码，以及调整不同运算所需的GAS量。

此外，每个区块执行交易时的GAS价格也是不同的，GAS的价格单位是GWEI，和ETH之间的计算如下

$1 WEI = 10^{-18} ETH; 1 GWEI = 10^9 WEI; 1GWEI = 10^{-9} ETH$

注意GAS是计量单位，WEI和GWEI是价格单位，计量 × 价格才会得到金额。

由于每个区块本质都是执行交易并确认交易，而执行交易需要消耗GAS，因此以太坊给每个区块设定了能使用的GAS上限，一般是上限30M，目标是15M。这个目标是以确认交易时只使用每个区块1半的存储空间为基准值算出来的。

换言之，理想情况下，如果确认的交易占用了1/2区块存储空间，那么基础GAS费用算出来会等于15M的目标值。但是实际场景下，如果确认的交易超出了1/2区块存储空间，那么GAS费用就会增加以迫使用户放弃交易，反过来如果当前交易占用区块存储空间不足1/2，那么GAS费用就会下降以鼓励用户发起交易。

由于金融市场是一个阻尼系统（区块链本质上也是金融市场），而且上一个区块的GAS费用会影响到下一个区块的交易费用，因此如果上一个区块GAS费较低，下一个区块很可能就会出现超出基准的情况，反之亦然，以太坊就是通过这种机制来调节用户交易的热情和频率。

当用户发起交易时，设定的GAS费实际是用户愿意支付的最大费用，而实际的成交会基于上一个区块的GAS消耗，因此实际的GAS费用有可能会低于意愿费用，或者高于，导致交易成功或失败。



#### 账户

以太坊的账户分2类，EOA和CA。

EOA（Externally Owned Accounts），就是一般人类开设的账户。以太坊不像比特币那样需要消耗UTXO才能交易，以太坊使用传统的账户余额模型，因此交易就是从A账户减去金额，然后B账户增加金额。此外为了防止重放攻击（就是如果A广播了他向B发起的转账，如果有攻击者拿到了转账请求，并重复广播，在没有预防措施的前提下，就会导致创建重复的交易，这样A就会一直向B转账直到A没钱了），每个账户还有一个nonce，它会加入到A的签名中，并在每次执行交易后自动增加，以保证即使其他参数相同时，A的签名也会不一样，这样就无法完成重放攻击了。**虽然攻击者知道nonce是自动递增的，但是它拿不到A的私钥，所以它无法制作出下一个nonce参与的A的签名**。

此外，nonce的意思是a **n**umber we are using only **once**。它的理论极限值是$2^{64} - 1$，没有重置手段，因此每个EOA有理论上的交易次数限制（虽然这个数字很大，在人一生中都不太可能做得到），当实际达到这个数字后，用户需要开设一个新的EOA，并把上一个EOA的资产转移过来，每个EOA对应的公钥和私钥都必须是不一样的。

CA（Contract Accounts），智能合约账户，智能合约是由人参与编写的程序，而且通常需要由EOA账户发起部署（智能合约理论上可以部署新的智能合约，但是最终还需要由EOA账户来发起这个行为）。一旦部署完成，这个智能合约就会被EVM分配一个账户（**通常是部署者的地址 + 部署者的nonce**），它也可以持有ETH（比如金库智能合约，或者其他ERC20或NFT之类的智能合约），它也可以使用一部分链上的存储空间。由于区块链的不可篡改特性，一旦一个智能合约被部署了，它的代码也是不可篡改的，所以如果需要升级智能合约，一般会采取代理模式，就是A合约只是一个代理，保存了实际合约的地址，然后实际合约可以通过不断部署地方式进行升级，而且在每次升级后，只需要修改A合约保存的实际合约的地址就可以了。

以太坊可能的账户数量是$2^{160}$，换言之是$16^{40}$，即40个16进制数字的组合，**可以发现所有的地址都是40个HEX字符的长度**。这个数字实际上非常非常大，因此不太可能会被耗尽。

注意**实际上在EVM看来EOA和CA没有本质区别，都是一个KEY--VALUE结构**，都可以存储金额并进行转账，只不过CA账户包含了字节码，因此可以通过ABI进行交互，而EOA账户只保留了EVM赋予的最基本权限，因此在行为上更受限制，但是可以接收转账和发起转账。所以任何交易都必须由EOA发起，更多是基于以太坊的游戏规则而在代码层面进行的简单限制，并没有在架构上做大改动。



#### 节点通信

由于区块链中没有中心节点，因此不存在统一的调配方来确保各个节点的数据保持一致，虽然有共识机制可以保证，**但那个是属于制度层面的设计，在实际执行上，由于网络问题，可能导致某些节点跳过某些广播，或者某些节点接收广播的顺序不对，从而导致潜在的数据不同步问题**。

区块链的设计者们为此思考了解决方案，我总结为以下几个核心原则：

- 广播机制，即口耳相传，即通过不断广播出去确保信息可以传递到整个网络
- 广播终结传播机制，即每个节点需要记录自己是否广播过某个消息，如果它已经广播过，那么当它再次从别人那里获得这条广播时，它不应该再进行广播，以保证每个节点只对每个消息广播一次
- 消息内容的连贯性，由于区块链是链式结构，每个区块都会指向上一个区块，因此每个涉及区块的广播实际上都会包含它的上一个区块的信息，假设有A和B区块分别创建并广播了，但是某个节点只收到了B区块的信息，它因此会知道它缺失了A区块的信息
- 节点的本地验证机制，还是上述例子，当某个节点在经过广播后知道它缺失了部分区块的信息，它可以主动向其他节点发起查询请求，以获得对应的区块信息并补充，当然它对于每个接收到的广播都要进行本地验证，上述例子就是一个说明，它通过验证区块B的信息，得知它自己缺失了区块A，这个过程可以无限递归下去，直到节点向其他节点请求并补充完整了区块

所以区块链设计者在设计节点时，应该考虑到了这个场景，即从零开始搭建节点，并确保它会通过向其他节点请求数据，以构建出完整的区块链的过程。

此外，由于区块链的特殊性，**要么自己本地搭建一个完整节点，要么使用别人搭建好的节点作为中转，否则无法直接通过简单发送网络请求就可以直接连接到区块链**。区块链行业的infra一般指的就是用自身资源搭建一个可以访问区块链的平台，以便他人在不搭建节点时也可以进行区块链相关开发。

如果需要自己搭建本地以太坊，那么环境准备好，启动客户端之后，**它会运行一个程序来访问以太坊社区维护的节点（通过IP访问到），这部分节点的IP是直接硬编码到软件内的，因此相当于以太坊社区会持续维护这些节点，也可以理解为遍历起点，之后的过程叫做peer discovery，就是类似一个图的遍历过程了，从遍历起点开始，找到和这些节点连接的节点，然后再访问它们，再遍历**，如果网络通畅的话，理论上可以访问到世界各地的所有完整节点。记住节点之间是平等的，因此本地构建就是一个图的遍历过程。一些provider本质上就是通过搭建自己的节点，然后给出一个中心化的API，让其他人可以通过它的节点来访问整个以太坊EVM单例。所以这个过程中其实就是它自己的前端服务器接受API请求，然后传递给它自己的EVM客户端，然后再传递到整个以太坊网络的过程：

```
用户 => 前端服务器 => 后端以太坊节点 => 以太坊区块链网络
```



#### 交易

这里的交易不是传统意义上的商品交易，而是指**必须由外部账户发起和私钥签名的，会导致EVM状态变化的行为**，一般的不消耗GAS的查询不会改变EVM的状态，如果某些特殊查询消耗了GAS，那么会导致ETH总供应量的下降，因而会导致状态改变，这种查询就是交易。

交易包含几个要素：

- from，发起者的钱包地址，**合约账户无法自行发起交易，必须是外部账户才可以**
- recipient，交易接收方，如果是部署合约，这里必须是空的
- signature，发起者的私钥参与的签名
- nonce，发起者的账户nonce，为了避免重放攻击必须添加这个
- value，发起者转移的以太币数量，以WEI进行表示，$1 ETH = 10^{18} WEI$，可以理解为ETH是大额钞票，WEI是零钱
- data，可支持发起方输入任意信息，一般在部署合约时这部分要填写编译后的合约代码（通常是编译为16进制的字节码），如果是调用合约，这里就是入参，注意即使是合约调用，入参也要转16进制，否则就用foundry cli之类的工具去自动帮我们转换
- gasLimit，交易所需的GAS最大值，如果矿工需要比它更大的GAS才能确认交易，则交易不会被确认
- maxPriorityFeePerGas，给矿工的每GAS对应的小费，类似打车软件的加价调度，所以如果这块给得多，可以增加交易被确认的速度
- maxFeePerGas，愿意支付的对应每GAS的最大费用，包括每GAS基础费用baseFeePerGas（由行情和网络繁忙程度确定）和maxPriorityFeePerGas，即每GAS给矿工的加价
- chainId，公链的ID，也是为了防止重放攻击，比如ETH主网就是1，Arbitrum One就是42161

交易类型有以下几种：

- 一般交易，即P2P的外部账户间的交易
- 合约部署交易，没有接收方地址（也可以理解为接收方就是整个区块链），此时需要在input data内提交合约代码
- 执行合约交易，即由外部账户发起的执行合约的行为，此时接收方是合约账户

交易的具体步骤：

1. 只有外部账户可以发起交易，因此先由外部账户发起并私钥签名
2. 广播，即交易最终会被广播到全区块链，从用户所在的节点开始，逐步扩散到其他节点，当然也会到达矿工节点，进入这些矿工所收集的待处理交易池（不同矿工看到的待处理交易池可以不同，这个要看矿工自己的配置，以及它更新的频率）
3. 进入池子之后，矿工会从池子中选择它觉得比较靠谱的交易，用来存入到下一个区块内，一般矿工会选择小费较高的交易，矿工验证交易有效（就是比对交易数据的哈希值和发起方的公钥解密的签名是否一致）后，就会纳入到下一个区块内
4. 交易被矿工挑出后，打包到了一个新的区块，新的区块形成，交易也被确认，如果后续不断地交易确认，产生新的区块，则之前的交易就不会被逆转，最终成为区块链不可篡改的一部分

GAS和WEI不同，它没有一个和以太币固定的兑换比例，可以理解为**GAS有一个市场波动价格，随着网络繁忙程度而变化**。到2025年初，1以太币约等于693GAS。

另外以太坊以安全和严格著称，因此虽然支持很多用户在任何时间点发起交易，但对矿工来说，**确认交易，存入区块的过程，永远都是单线程的**，而**以太坊也只允许单线程地创造新的区块**，所以本质上，**在任何时刻，全区块链上只允许执行最多一个交易**，以确保当知道了待执行的交易序列后，所有人都可以预测到这一系列交易的最后结果，因为它是线性执行的。



#### 交易签名过程

简单来说是这样的，交易的时候，发起方的代码如下：

```js
const transactionData = {
    from: address,
    to: recipient,
    value: parseInt(sendAmount),
  };
  const txStr = JSON.stringify(transactionData);
  const txHash = keccak256(utf8ToBytes(txStr));

  // create signature
  const privateKeyBytes = hexToBytes(fromPrivateKey);
  const signature = secp.sign(txHash, privateKeyBytes);

  // include signature in the payload
  const {
    data: { balance },
  } = await server.post(`send`, {
    data: transactionData,
    signature: {
      r: "0x" + signature.r.toString(16),
      s: "0x" + signature.s.toString(16),
      recovery: signature.recovery
    },
  });
  setBalance(balance);
} catch (ex) {
  console.error(ex);
  alert(ex.response.data.message);
}
```

发起方封装一个未加密的交易信息，然后用自己的私钥签名，最后把签名信息和未加密交易信息一起发送到区块链的智能合约上。

之后智能合约的验证逻辑（假设也是用JS写的话）：

```js
const { data, signature } = req.body;

// verify the signature first
const txHash = keccak256(utf8ToBytes(JSON.stringify(data)));
const rBytes = hexToBytes(signature.r);
const sBytes = hexToBytes(signature.s);
let sig = secp.Signature.fromCompact(new Uint8Array([...rBytes, ...sBytes]));
sig = sig.addRecoveryBit(parseInt(signature.recovery));

// keep public key uncompressed
const publicKeyBytes = sig.recoverPublicKey(txHash, signature.recovery).toRawBytes(false);

// now we have public key, transform it into wallet address and compare
const wallet = generateWalletAddr(publicKeyBytes);
const walletIsValid = wallet === data.from;
if (walletIsValid) {
  setInitialBalance(data.from);
  setInitialBalance(data.to);

  if (balances[data.from] < data.value) {
    res.status(400).send({ message: "Not enough funds!" });
  } else {
    balances[data.from] -= data.value;
    balances[data.to] += data.value;
    res.send({ balance: balances[data.from] });
  }
} else {
  res.status(400).send({ message: 'Signature is invalid!' });
}
```

从请求中拿到签名和交易数据，然后结合签名和交易信息，反向推导出转账者的公钥，以及钱包地址，和签名信息进行校验，如果校验通过说明这个签名是使用转账者的私钥制作的，间接证明这个转账是转账者本人发起的（持有私钥者视为本人）。

所以也可以看出一般的交易签名不太需要转账者的公钥。

为了防止重入攻击（也就是每次转账理论上都是类似NFT的，唯一的），一般会在转账请求中加一个随机数nonce。



#### 比特币的节点选择

比特币的节点分为完整节点和轻量节点，前者包含了所有的交易信息和世界状态树，大概是600多G空间，后者只包含所有区块的区块头信息，当有需要时才会下载对应区块的交易信息，因此可以大幅节省存储空间。一般用手机钱包的都是部署的轻量节点。



#### 比特币的UTXO模型

比特币和以太坊都需要某种数据结构来存储信息，比特币采用的是UTXO模型，即Unspent Transaction Output，这个后面再解释。

区块链上任何一笔转账，都需要以下要素：

- 转账地址
- 目标地址
- 金额
- 转账人的签名（公钥私钥体系）

区块链上任何一笔转账都会导致全局的状态变化。

一般的电子系统，如果要记录转账信息，会通过账户的形式，一个账户通常包含以下字段：

- 账户名，ID
- 余额

一般还会有一张表用于记录所有交易信息，即转账账户，接收账户，时间，金额等等。

比特币的UTXO模型，从字面上理解就是**未花费的转账结果**，即它表示的是用户的账户余额是一系列交易的结果，**用户拥有的不是一个数字代表的余额，而是交易记录和记录中附带的金额**，用户的代币是由不同的交易记录中附带的金额累积的，这里具体解释一下：

1. 新入场用户可以通过交易所，从他人手里购买比特币，假设B转账给A，3个比特币，此时A就拥有了一份UTXO，它的金额是3个比特币
2. 假设后续C转账给A，2个比特币，那么A就拥有了两份UTXO，合计5个比特币
3. A转账给D，2个比特币，那么A需要花费他的UTXO，这里可以选择第一个UTXO，**它持有3个比特币，会被标记为已消费，然后一个新的持有1个比特币的UTXO会被创建，且状态是未消费（unspent）**，他也可以选择消费第二个UTXO，这样这份UTXO就被标记为已消费，并被销毁（**用户无法继续持有已消费的UTXO**），第一份UTXO状态保持不变，同时D会拥有一个新的UTXO，金额是2个比特币，状态是未消费

所以UTXO模型本质上记录的是交易记录，用户持有的UTXO里面带有金额，UTXO的状态可以是未消费或已消费，已消费的UTXO的金额不能再次使用，以避免double spent问题。

以太坊的账户模型通过不断修改转账请求中的nonce来防止重入攻击，而UTXO模型天然就可以防止重入攻击，因为每一个UTXO就可以理解为一个NFT，只是在金额上它们允许互换，具有等价性。

UTXO还有一个特性，叫做**witnessScript**，意思就是一个上锁机制，来限制它的花费条件，当上锁机制被满足时，才允许解锁从而消费，一般来说锁机制是这样的：

- 对一段文本，可以是自然语义也可以是随便什么字符串集合，进行哈希，把这个哈希保留到链上，但是文本本身不保留
- 一些签名要求，即类似多签机制，锁可以设置一个或者多个签名要求，如果尝试解锁方没有提供对应的签名（一般要求解锁方持有这些签名对应的密钥），那么也不可以解锁，类似现实中的多个锁被多个人持有，共同开启某个门的机制

如果需要解锁，解锁方需要提供文本的明文，以进行哈希，确认它和存储到链上的哈希一致，此外还需要提供对应的签名。



#### 比特币和以太坊的供应量模式

比特币是有总固定数量的，大概每2016个新区块被创建后，挖矿收益就会减半，因此按照它发布时的模型，整体预测下来大概100多年后，挖矿不再产生奖励，矿工只能从确认交易中收取手续费作为报酬了。这个固定数量就使得比特币未来随着经济发展会越来越值钱，类似黄金的价值。

以太坊则采取了POS的创建区块规则，创建区块会产生奖励，日常交易和维护智能合约会导致基础货币的消耗，而且以太坊不像比特币并没有一个预先定义好的严格的挖矿收益计算公式（实际有一个公式但是参数可以被委员会调整）。所以实际上委员会可以调整挖矿收益和基础货币燃烧效率。如果总体上是供应量增长则会进入通胀状态，反之就是通缩，目前情况是略有通缩。

另外以太坊允许的账户数量是$2^{160}$，这个数字远远大于地球上曾经到现在存在过的人类数量，所以在可预期的将来，账户都是够用的。



#### L1和L2

这部分是网上查资料看到的。

有一个理论，叫区块链不可能三角，即一个区块链不可能同时满足安全，去中心化，和拓展性，以太坊专注于安全和去中心化，因此它失去了拓展性，为此有很多人在研究怎么解决这个问题。

所谓拓展性就是保持现有能力和效率的前提下，扩大参与者数量和使用量。对区块链来说，拓展性就是增加交易频次，但不会导致功能缺失或者降级。

主流意见就是on-chain和off-chain，即链上（L1）和链下（L2），简单来说L1侧重于优化现有的链条，而**L2则侧重于绕开现有链条的限制去解决问题，然后再解决怎么把解决方案和主链交互的问题**。

基本上从2018年开始以太坊的拓展性问题得到了重视，之后各种技术和生态出现，主流的思路在向L2看齐，不少新的生态也是基于某个基础加密货币的L2来开发的，比如后面要学习的Arbitrum就是基于以太坊的L2解决方案，它本质是一个运行于L1之外的去中心化程序，因为部署和开发都不直接依赖L1，因此可以绕开L1的各种限制，进行更高效的操作，当然操作完成后还是要和L1进行交互以确认交易结果。



#### JSON-RPC

区块链开发，如果作为一个完整的产品看，本质上是这样的，前端是DAPP，通过JSON-RPC和后端，也就是区块链交互，区块链作为后端，就是创建和部署智能合约。虽然每次交互都是和某个节点去交互，但交互后最终会被广播到全节点，所以**整个开发过程可以看作DAPP和一个单例的EVM之间的交互**。实际开发中后端会对接钱包，然后由钱包作为中转服务器来对接区块链。

而JSON-RPC（JSON Remote Procedure Call）本质上就是使用JSON格式和区块链进行交互的方式，它和RESTFUL API类似，也擅长CRUD操作。

比如一个JSON-RPC的请求如下：

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_getBlockByNumber",
  "params": [
    "finalized",
    false
  ]
}
```

注意到上述数据只是JSON，但实际上如果通过钱包去和区块链交互，发送的都是HTTP请求，因此**这部分数据实际上是作为请求体，被包含在一个POST请求内的**。

上述请求的响应部分可以自己测试，比如直接去ALCHEMY上创建一个应用，然后拿到应用的API-KEY，就可以获取到对应开发代码，然后把代码下载到本地运行测试就可以。

测试用代码：

```javascript
// run npm install alchemy-sdk first
import { Network, Alchemy } from "alchemy-sdk";

// Optional config object, but defaults to demo api-key and eth-mainnet.
const settings = {
  // Replace with your Alchemy API Key.
  apiKey: "Vzt_437Deygk7-Zi3kKFCO2D_f_tMI1W", 

  // choose eth mainnet here
  network: Network.ETH_MAINNET,
};
const alchemy = new Alchemy(settings);

alchemy.core.getBlock(15221026).then(console.log).catch(err => console.error(err));
```

执行上述代码就会发送一个POST请求，请求体包含JSON-RPC格式，从以太坊主链获取到某个区块的信息。

用AXIOS也可以实现相同的效果，只不过需要自己去把JSON-RPC数据封装到请求体内。

上述例子只是展示了一个只读的JSON-RPC，在某些请求中，比如一个查询钱包余额的请求：

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "params": [
    "0xe5cB067E90D5Cd1F8052B83562Ae670bA4A211a8",
    "latest"
  ],
  "method": "eth_getBalance"
}
```

可以看到它甚至都不需要发送者的钱包地址或者私钥，换言之任何人都可以通过各种发送请求的工具去向以太坊请求查询某个钱包的余额并且不需要支付GAS。而其他的需要支付GAS的行为，即交易，就需要发送者把某个私钥拿出来签名才能发送了。



#### JSON-RPC发起交易

注意一般的VIEW或者链下查询操作，通过JSON-RPC构造一些`eth_getXXXX`的方法，传入参数就可以了，但是对于交易来说，情况更复杂，简单是这样的：

- EOA，也就是你，通过各种方式构造一个不带签名的JSON-RPC数据
- EOA把这个数据发送给PROVIDER，一般就是钱包
- 钱包拿到这个数据，结合你的授权，进行签名，生成一个很长的HEX字符串
- 钱包把这个签名加入到另一个JSON-RPC数据内，最后把这个数据发送给以太坊

所以我们看到的JSON-RPC实际上有两个，一个是不签名的，一个是签名的，不签名的一般长这样：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendTransaction",
  "params": [
    {
      "from": "0xYourEOAAddress",
      "to": "0xYourContractAddress",
      "gas": "0x76c0",
      "gasPrice": "0x9184e72a000",
      "value": "0x0",
      "data": "0xYourEncodedFunctionCallData",
      "nonce": "0x1",
      "chainId": "0x1",
    }
  ]
}
```

注意，method的值是`eth_sendTransaction`，表示它是一个需要进一步加工的JSON-RPC。

data里面是需要调用的智能合约的函数签名和入参的哈希转换。

签名后的JSON-RPC长这样：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendRawTransaction",
  "params": [
    "0xAVEYLONGANDLONGHEXSTRINGTHISISJUSTANEXAMPLE"
  ]
}
```

注意此时method变成了`eth_sendRawTransaction`，说明它是一个直接可以被EVM解析的JSON-RPC，并且params里面只有一个很长的十六进制字符串，它就是原始交易数据加私钥签名后的结果。

如果用代码去写，以下是一个样例，基于ARBITRUM SEPOLIA网络，使用了alchemy-sdk：

```javascript
import { Network, Alchemy, Wallet, Utils } from "alchemy-sdk";
import 'dotenv/config';

const privateKey = process.env.TEST_PRIVATE_KEY;
const apiKey = process.env.ALCHEMY_API_KEY;
const targetAddr = process.env.TARGET_ADDR;

const settings = {
  apiKey: apiKey,
  network: Network.ARB_SEPOLIA 
};

const alchemy = new Alchemy(settings);
const wallet = new Wallet(privateKey);

async function run() {
  const nonce = await alchemy.core.getTransactionCount(wallet.address, 'latest');
  const tx = {
    to: targetAddr,
    value: Utils.parseEther('0.001'),
    gasLimit: '21000',
    maxPriorityFeePerGas: Utils.parseUnits('5', 'gwei'),
    maxFeePerGas: Utils.parseUnits('20', 'gwei'),
    nonce: nonce,
    type: 2,
    chainId: 421614
  };
  const rawTx = await wallet.signTransaction(tx);
  const txResult = await alchemy.core.sendTransaction(rawTx);
  console.log(txResult);
}

run();
```

发起交易需要签名，需要拿到钱包的私钥，基本上用METAMASK注册的钱包可以满足大部分场景。ALCHEMY需要在DASHBORAD里面构建一个能连接ARBITRUM SEPOLIA的网络（注意不同于ETHEREUM SEPOLIA），拿到对应API_KEY，就可以开发了。



#### 前端开发库ether.js

前端开发主流语言是JS，WEB3的主流库是web3.js或者ether.js。之前用到的alchemy-sdk是一个单独的库，但也可以结合这2个库一起使用，不过这里就抛开alchemy-sdk。

问了一下AI关于这2个库的比较，看上去web3.js更老一些，模块化程度更低，性能也更差一些，所以目前首选是ether.js。

钱包初始化，直接用私钥是可以构造钱包的，但前端一般不会这样做，不安全，而是直接和浏览器钱包插件交互更安全，比如metamask如果安装了，会提供一个window.ethereum对象，就可以直接拿到对应的签名授权（估计还是要问用户授权的）。即**前端开发时不构造钱包，直接构造对应的provider，然后拿到对应的签名**。

但是ether.js也可以用于nodejs后端开发，此时就可以把私钥存储在服务器上并在需要时直接使用ether.js构造钱包实例了。

编码过程中注意，由于以太坊的线性特性，一个交易发送后不一定会成功，需要等待它出结果后再发送下一个交易，不然如果同时发送多个交易，会导致nonce的变化不对，从而交易失败，代码是这样写的：

```javascript
const tx = await wallet.sendTransaction(rawTx);
const recepit = await tx.wait(); // wait for the transaction to be confirmed
if (receipt.status === 1) {
  console.log("Transaction successful! Block:", receipt.blockNumber);
  // then start another transaction
} else {
  // handle error
}
```

另一个例子，比如如何查询某个账户的所有发起的交易，答案是从每个区块开始扫描交易记录，然后对每个交易进行查看，找到对应的发起者和接收方，然后和该账户匹配，最后就可以得到此账户发起的所有交易：

```javascript
async function findTransactionReceivers(address) {
  const blockNum = await provider.getBlockNumber();
  const receiverArr = [];
  for (let i = 0; i <= blockNum; i++) {
    const block = await provider.getBlock(i);
    if (block.transactions.length > 0) {
      const txList = block.transactions;
      for (let j = 0; j < txList.length; j++) {
        const txHash = txList[j];
        const txDetail = await provider.getTransaction(txHash);
        const sender = txDetail.from;
        const receiver = txDetail.to;
        if (sender === address) {
          receiverArr.push(receiver);
        }
      }
    }
  }
  return receiverArr;
}
```

当然上述方法相当于遍历整个区块链的链表结构，对每个区块的所有交易都需要发起请求查询，相当费时费力，所以像etherscan这样的网站就会负责在链下（也就是他们的中心化服务器上）以更高效的传统数据库存储结构把交易信息直接存储好，这样当需要查询某个账户相关的所有交易时，可以不花费GAS，以更快的速度，更方便的操作得到结果。



### Solidity智能合约语言学习

SOLIDITY语言用于编写以太坊的智能合约，它是高级汇编语言，从语法上看和JavaScript类似，也有编译过程，产出字节码。

SOLIDITY专门用于编写EVM兼容的智能合约。由于EVM本质上是单例的，因此任何部署在其内部的智能合约，都可以看作是对EVM的API增强。



#### 入门

一个最简单的SOLIDITY程序如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract MyContract {
	constructor() {
    } 
}
```

第一行注释表示当前智能合约的开源协议，一般都是MIT，因为它限制最小。

第二行`pragma solidity ^0.8.4;`表示当前合约的源码版本，换言之如果编译器版本低于源码版本，编译就会失败，必须声明，不然编译器不知道用哪个版本。

之后声明了一个智能合约，名称是`MyContract`，然后后续所有内容都放到它的作用域里面。

合约可以声明构造器，和JS一样，它会在合约初始化后执行一次，**智能合约只能部署一次，因此构造器内的代码也只会执行一次，如果后续有新的节点被设立好，那些节点也只会把此智能合约的代码下载过来，并不会再次执行**，因为这个合约对它们来说已经是部署好的状态了。 一般来说构造器就是用于合约初始化的。

既然有了构造器那么就要声明状态了，比如：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

address owner;
bool isHappy;

contract MyContract {
    constructor(address _owner, bool _isHappy) {
        owner = _owner;
        isHappy = _isHappy;
    }
}
```

语法是`<type> <variableName>`，类型声明放在前面。address，地址类型，默认是0x0，bool布尔，和JS一样默认是false。另外注意到构造器里面没有`this.owner = owner`这种写法，关键是没有`this`，实际上SOLIDITY语法里面是有`this`关键字的，但**它表示的是当前合约的地址**，而非当前合约的面向对象实例，因此这个场景下可以不用。当然原则上可以在构造器内使用`this`，这也说明此时合约已经部署完成并且拿到它自身的地址了。

还是上面构造器的例子，因为不用THIS，所以入参和状态变量不能重名，所以规范就是入参加一个下划线区分，此外它也表示如果要部署合约，**需要在部署时添加入参**，不然部署会失败，以JS代码编写的部署过程会是这样的：

```javascript
const myContractInstance = await contract.deploy('0x38cE03CF394C349508fBcECf8e2c04c7c66D58CB', true);
```

状态变量还可以加访问修饰符：

```solidity
address public owner;
bool public isHappy;
```

加了修饰符的状态变量可以通过对应的`get`方法访问到，关于访问修饰后续再补充。

还可以加数字类型的状态变量：

```solidity
uint public x = 10;
int public y = -50;
```

unit默认是256位，所以uint = unit256，int = int256，范围应该都知道怎么算。

一些常用的数据类型如下：

- `bool`，布尔类型
- `string`，字符串类型，**必须用双引号包裹**，单引号在SOLIDITY里面表示CHAR类型
- 数字类型，用intxx或者unitxx表示，比如int8，unit16等等，不加XX就默认的256
- `bytes`，单byte数组，SOLIDITY内表示这个类型和JS不一样，**注意它每个元素是单BYTE长度的**，因此最大值都是FF，255，比如一个字符串表示的数字`"AB2367CD"`，会被SOLIDITY解读为这个数组`[0xAB, 0x23, 0x67, 0xCD]`，另外`"234"`这种写法会报错，因为bytes要求对应的数字字符串**必须是偶数长度**，所以要么是`2340`，或者`2304`才可以。**还可以用`bytesXX`表示固定长度的单byte数组**，比如`bytes32`表示固定长度是32的单byte数组，即这个数组有32个位置，每个位置允许存一个byte大小的数字，即0~255的范围。如果只是声明为`bytes`，那么会和`string`一样是可变长度的类型
- enums，枚举
- arrays，数组，支持动态扩容
- mapping，表结构，即键值对
- tuples，有限复合集，和RUST里面的一样，使用`(8, true)`表示，也可以解构
- structs，类似对象的存在，RUST里面叫结构体
- address，地址类型，SOLIDITY语法专有的，因为它是编写智能合约的语言，它是基于字符串的再次封装，加了一些特殊的方法，比如`balance`，`transfer`等等，构造一个地址类型的写法是`const addr = address("0x12345678");`之后就可以调用其`balance`或`transfer`方法等

还有一些环境变量（context），用于获取一些额外信息，比如：

- msg，拿到当前调用合约的账户的信息
- tx，当前调用合约的交易行为
- block，当前合约部署所在的区块的信息

进一步介绍枚举，枚举在SOLIDITY内本身是作为一种类型定义的，因此定义枚举，不管是全局定义（在contract作用域外部定义）还是在contract内定义，都是可以，而且不会触发GAS消耗，声明枚举写法：

```solidity
enum MessageType { Greeting, Criticism }
```

注意枚举类型本身不能是public，只有状态变量才能是public，另外声明枚举类型不需要以分号结尾。

使用枚举也很简单，举例：

```solidity
function setMessage(MessageType newType) public {
    if (newType == MessageType.Greeting) {
        message = "hello I like you";
    } else if (newType == MessageType.Criticism) {
        message = "hey I dont like you";
    }
}
```

测试部分是这样的，在chaijs里面无法获取到具体的枚举定义，**枚举值只能用数字表示，第一个值是0，第二个值是1**，因此测试这样写：

```typescript
it("test set message", async () => {
    await contract.setMessage(1);
    const message = await contract.getMessage();
    expect(message).to.equal("hey I dont like you");
});
```

有限复合集，它可以作为函数的返回值，也可以作为一个变量的表达式，也可以解构：

```solidity
function getTuple() public returns pure (uint8, bool) {
    return (12, false);
}
(uint8 x, bool y) = getTuple();
```

上述代码中`pure`表示不和状态变量交互，也不和区块链交互，还有其他修饰符，本质上都是给函数进行标记，错误的标记会导致编译或者执行失败，或者更低的性能，SOLIDITY的编译器SOLC不会像RUST那样进行完整的变量跟踪，所以需要开发者自己去标记。



#### 环境搭建

操作系统是LINUX。

先安装NODEJS和NPM，在UBUNTU下官方建议是先安装NVM，而NVM需要通过CURL命令去下载一个脚本并执行脚本安装。

安装完成NVM后可以通过`nvm -v`测试一下。

然后通过NVM命令安装NODEJS，这个参考官方网站就可以了，确保NODEJS和NPM都可以通过命令访问到。

**注意`package.json`内千万不要配置`"type": "module"`，会导致后续无法通过`npx hardhat run foo.ts`直接编译TS文件并执行，切记切记**，只能配置为`"type": "commonjs"`或者不配置（还是默认的commonjs）。

然后安装HARDHAT，它是使用JS开发的，一个兼容EVM虚拟机程序的开发环境，因此需要NODEJS运行环境，它包含了SOLIDITY编译器solc和运行环境EVM，以及测试相关组件，可以说是本地开发SOLIDITY智能合约的最佳工具，具体这样操作：

- VS CODE IDE先安装solidity拓展插件
- 创建一个空文件夹，然后执行npm init初始化
- 在这个项目内安装HARDHAT，执行`npm install -D hardhat`，之所以放在项目内去安装是为了确保其他项目可以安装不同版本的HARDHAT，这样如果出现问题，本地更好基于对应版本去复现，这里安装了2.22.19版本
- 然后执行`npx hardhat init`，注意到它提供了JS，TS等选择，**注意不要选TYPESCRIPT版本，选JAVASCRIPT版本就可以了，这样兼容性最好，得出此结论花费了数个小时**，所以选择JS版本，之所以有JS代码是因为**部署，编译，测试等等环节需要使用JS代码处理，但智能合约本体还是用solidity语言开发**。选项里面有一个是否安装hardhat-toolbox，它是一个开发套件，就是集成了编译，测试，部署等功能，这里还是建议安装，少走一点弯路
- 初始化后可以看到有JS代码，也有SOL文件，里面写的就是SOLIDITY代码，定义智能合约，执行`npx hardhat compile`可以发现多出了一个文件夹`artifacts`，里面就是编译后的字节码，可以用于部署智能合约，也说明它基于SOLIDITY版本下载了对应的SOLC包并进行了编译
- 后续所有JS代码都使用COMMONJS风格，因为这是HARDHAT目前版本推荐的风格，选择ESM会有一些问题

然后把`hardhat.config.js`的代码也调整一下：

```javascript
require("@nomicfoundation/hardhat-toolbox");

const config = {
  solidity: "0.8.28",
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545"
    }
  }
};

module.exports = config;
```

之后编写一个简单的智能合约，放在contracts路径内：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.28;
import "hardhat/console.sol"; // 这里引入console，支持在合约中通过console.log输出信息，方便调试

contract HelloWorld {
    string public message;

    constructor(string memory initMessage) {
        message = initMessage;
        console.log("hello world contract inited");
    }

    function setMessage(string memory newVal) public {
        console.log("set message");
        message = newVal;
    }

    function getMessage() public view returns (string memory) {
        console.log("get message");
        return message;
    }
}
```

编写好后执行`npx hardhat compile`，看是否能编译成功，如果成功，在`artifacts`内可以看到对应的包含编译后字节码的JSON文件。

之后在`test`路径内写一个简单的单元测试，引入CHAIJS库去写：

```javascript
const { expect } = require("chai");
const hre = require("hardhat");

let contract;

describe("test begins", function() {
    this.beforeAll(async () => {
        const contractFactory = await hre.ethers.getContractFactory("HelloWorld");
        contract = await contractFactory.deploy("init message");
        console.warn("contract deployed");
    });
    it("test contract addr", async () => {
        const addr = await contract.getAddress();
        expect(addr).to.not.be.empty;
    });
    it("test get message", async () => {
        const message = await contract.getMessage();
        expect(message).to.equal("init message");
    });
    it("test set message", async () => {
        await contract.setMessage(1);
        const message = await contract.getMessage();
        expect(message).to.equal("hey I dont like you");
        await contract.setMessage(0);
        const message2 = await contract.message();
        expect(message2).to.equal("hello I like you");
    });
    it("test add and add2", async () => {
        const a = await contract.testAdd(11,22);
        const b = await contract.testAdd2(33,44);
        expect(a).to.equal(33);
        expect(b).to.equal(77);
    });
});
```

之后执行`npx hardhat test test/HelloWorld.ts`就可以单独针对这个用例进行测试了，通过这种方法来调试合约代码的问题。

注意每次执行上述命令后都会重新编译合约，因此如果修改了合约代码也不用手动去执行编译命令。



#### HARDHAT CLI常用命令

因为HARDHAT是安装在项目中的，所以使用`npx hardhat`作为前缀，下面不再提及。

- --help，帮助
- init，初始化项目
- clean，清空缓存和构建结果
- compile，编译智能合约
- test，使用测试套件去编译和测试智能合约
- node，启动本地区块链模拟节点，会有测试账号和测试币和测试私钥
- ignition，使用ignition相关命令，一般是配合node命令本地部署智能合约

后面慢慢补充……

之后学习SOLIDITY就可以用上述简单合约来进行各种测试了。



#### 状态变量

定义在合约全局的变量就是状态变量，比如：

```solidity
contract Contract {
	bool myVariable; // 这个就是状态变量
}
```

**状态变量会永久存储于区块链的Storage区域**，因此修改会消耗GAS，如果通过`view`标记的函数访问，可以不消耗GAS。



#### 函数

函数写法：

```
function foo(uint8 arg1, string arg2) public view returns (bool) {}
```

使用`function`关键字开头，接函数名称和入参，之后接函数的访问控制以及函数类型，最后声明返回类型，如果函数没有返回值可以不写，最后是代码块和函数体。

注意到上述例子的函数类型有多个组合，实际上也是，函数类型一般分成2类，访问性以及可变性，**一个函数支持最多配置1个访问性类型和最多1个可变性类型**，不支持配置多个访问性类型或者多个可变性类型，**类型声明的先后顺序不影响效果**。

访问性类型：

- public，函数可以被自身，子类合约，外部调用
- external，函数只能被外部调用，自身无法调用，子类合约需要通过特殊方式才可以调用
- internal，函数只能被自身和子类合约调用
- private，函数只能被自身调用，子类合约和外部都无法调用

可变性类型：

- pure，函数是一个标准意义上的纯函数，它不读取状态变量，也不读取区块链的数据
- view，函数是一个只进行查询的函数，它可以读取状态变量，也可以读取区块链数据，但是不能做任何会导致修改区块链状态的行为
- 常规，就是不加任何可变性类型，因此可以做任何事情，包括修改区块链状态，一般来说如果一个函数一定要修改区块链状态，则不要加可变性类型，否则建议添加，会减少GAS计算

此外，如果一个函数是pure类型，但是用view进行了修饰，**虽然不会影响函数的执行，但是会增加GAS计算**，所以建议对每个函数进行精确标记以优化整体GAS消耗。

返回值的写法，以下2种都是可以的：

```solidity
function testAdd(uint8 a, uint8 b) pure public returns (uint16 c) { // 注意返回类型后面声明了一个占位符变量
    c = a + b;
}
function testAdd2(uint8 a, uint8 b) pure public returns (uint16) {
    uint16 c = a + b;
    return c;
}
```

注意第一种写法，它在函数的前面里面的返回类型加了一个变量声明，这样只要在函数内对返回值类型进行了初始化或者赋值运算，就视为它有了返回结果，**这个操作不需要放到最后**，比如：

```solidity
function testAdd(uint8 a, uint8 b) pure public returns (uint16 c) {
    c = a + b;
    c = c + 1; // 最后的返回值会以这行代码执行后的c为准
    unit8 x = 3; // 这里继续写代码不会影响函数的执行，因为它前面里面的变量c已经有了结果了
}
```

当然推荐第二种传统写法，因为它符合传统思路，在编写函数体时也不需要考虑返回值已经被占据了一个名称。

SOLIDITY函数支持重载，和JAVA一样，**只要入参不同就可以重载**。



#### 变量修饰符

注意到变量声明只有类型，而不像JS那样有可变性声明（`let`或`const`）吗？实际上SOLIDITY里面也有可变性声明，只是限制的范围更少，它通过`const`和`immutable`来控制，比如：

```solidity
uint256 public constant MAX_SUPPLY = 1000; // 禁止任何形式的修改

contract Example {
    uint256 public immutable startTime;

    constructor() {
        startTime = block.timestamp; // 必须在合约构造时就初始化，且只能初始化一次，后续无法修改
    }
}
```

简单来说，`const`就和RUST里面的同名修饰变量一样，真正意义上的常量，而`immutable`则是只能在构造器内初始化一次的变量，一个是编译期就不可变，一个是运行时期的不可变。

没有上述修饰符的变量，默认都是可以变化的。

变量的访问修饰符，比函数的少了external，因为变量默认都是需要在函数内调用的，具体修饰符有：

- public，自身，子类合约，外部都可以访问到，比如一个状态变量是`public`修饰，名称是message，则外部可以通过`message()`同名函数的调用访问到它
- internal，自身和子类合约可以访问到
- private，只有自身可以访问

注意，和函数的类型修饰符可以忽略顺序不同，**变量的访问修饰符必须放在类型之后**，即`uint public x`是可以的，`public uint x`是不可以的。



#### 函数的装饰器modifier

SOLIDITY允许用装饰，或者切面编程的方式在函数执行前后执行通用逻辑，用法如下：

- 声明一个modifier，起到装饰的作用，语法是`modifier fooBar() {}`，注意没有使用function关键字
- 在装饰器内部，编写代码逻辑，并用`_;`表示需要插入被装饰的函数的代码，比如：

```solidity
modifier logModifier {
    console.log("before");
    _; // 这里表示被装饰的函数的代码执行
    console.log("after");
}
```

- 最后在需要装饰的函数上，添加此装饰器，比如：

```solidity
function logMessage() public view logModifier { // 注意这里把装饰器和其他的修饰符并列
    console.log("log message");
}
```

装饰器的好处有很多，比如在智能合约内，一些外部方法可能需要校验调用者的账户是否具有资格，比如只有部署者，或者初始化时的某几个账户才能调用，此时就可以通过装饰器声明一段校验代码，如果校验失败则不会执行实际的业务代码。



#### 获取ABI信息

ABI是一个智能合约对外的接口说明，通常是JSON格式，表示这个合约有哪些方法可以调用，各自的签名是什么。

如果是自己开发的智能合约，使用HARDHAT编译后会自动生成对应JSON文件，里面就包含了ABI。

如果要获取一个链上的智能合约，通常需要使用CURL等命令，结合一个API-KEY去请求，请求结果往往就是JSON格式的ABI。

所以，**ABI主流以JSON格式展示**。开发时如果需要引入ABI，也建议以原封不动的JSON格式引入。



#### 本地部署智能合约和简单交互

这里和之前的单元测试不同在于，需要在本地开一个端口部署智能合约，这样后续可以模仿在链上使用ether.js直接和智能合约交互。

之前的部署是基于单元测试的，它部署完成后立刻就进行测试了，而且并没有用到ether.js。这里会用到HARDHAT内置的IGNITION模块进行合约部署，部署到本地模拟节点上，并进行测试。

首先改一下HARDHAT配置，即hardhat.config.ts：

```typescript
const config: HardhatUserConfig = {
  solidity: "0.8.28",
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545"
    }
  }
};
```

然后启动hardhat的服务器，`npx hardhat node --port xxxx`，如果不加port就是默认8545，可以自己改，启动这个服务器就意味着启动了一个模拟的EVM节点，可以用于部署本地合约并后续用etherjs进行测试了，注意这个终端不要关闭。

启动完成后服务器会返回一组测试用的账号和对应私钥。

如果后续要部署到测试链，比如SEPOLIA，则可以增加一个新的配置，但是**注意必须要配置私钥和测试链的ID，同时保证对应账户有足够的测试币**。也可以通过添加`--estimate`来预估一下部署GAS。

之后去`ignition/modules`下面新建一个TS文件，然后编写部署智能合约的部署代码：

```javascript
const { buildModule }  = require("@nomicfoundation/hardhat-ignition/modules");

// pass module id and callback function
const myModule = buildModule("HelloWorldModule", (m) => { 
    // name the contract and pass constructor args, the name must be the actual contract name
    const module = m.contract("HelloWorld", ["build by ignition"]);
    return { helloWorld: module };
});

module.exports = myModule;
```

里面注意几个点：

- buildModule方法后面的第一个入参传入的是模块ID，这个ID可以随意，不要重复就行
- 后面跟随的m看源码可以看到是一个ModuleBuilder，它用于模拟合约部署和激活（初始化）的过程
- 调用`m.contract`方法模拟一个合约的部署和初始化过程，其中入参1是合约名称，必须要和编译出来的合约JSON文件里面的contractName字段匹配，第二个参数是合约构造器的入参，如果合约构造器没有入参可以不写
- 之后可以进一步写一些代码，以模拟需要在部署后立刻执行的操作，比如设置其他参数，修改Storage的值等等
- 匿名函数要返回一个对象，value一定要是通过部署生成的Future对象，可以把Ignition的Future对象理解为JS的PROMISE
- 最后把这个函数调用过程作为模块默认导出

然后就可以部署智能合约了，保证合约已经编译好，有JSON文件后，开启一个新的终端，执行：

```
npx hardhat ignition deploy ignition/modules/HelloWorld.ts --network localhost
```

这里注意一定要加`--network localhost`，因为这里是部署到本地节点。

如果部署成功，可以看到节点终端那边输出了新的消息，比如显示合约部署成功，地址是什么，消耗了多少GAS。这个合约地址需要记录下来，后续用ETHERJS连接的时候需要用到。

然后编写测试文件，即实际用ETHERJS连接部署到的合约并进行测试：

```javascript
const ether = require("ethers");
const contract_json = require("../artifacts/contracts/HelloWorld.sol/HelloWorld.json");

const contractAddr = "0x5fbdb2315678afecb367f032d93f642f64180aa3";
const abi = contract_json.abi;
const privateKey = '0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80';

async function main() {
    const provider = new ether.JsonRpcProvider("http://127.0.0.1:8545");
    const signer = new ether.Wallet(privateKey, provider); // 实际开发时这里都是和各种钱包进行交互，由钱包来提供signer，避免泄漏私钥
    const contract = new ether.Contract(contractAddr, abi, signer);

    const message = await contract.getMessage();
    console.log(message);
    await contract.setMessage(0);
    const message2 = await contract.message();
    console.log(message2);
}

main();
```

编写完成后，执行代码就可以看到结果了，比如调用合约查询到了某个值，或者修改了某个值。



#### 转账到合约内

智能合约可以通过`payable`来获取外部转账，**金额单位是WEI**，但是一旦涉及到钱，事情就应该变得谨慎，一个最简单的例子如下：

```solidity
function getBalance() public view returns (uint) {
    return address(this).balance; // 这里this表示当前合约的地址，通过address()封装后可以获得一个对象，从而查询其余额
}

function deposit() payable external {}
```

注意上述代码，每个智能合约都有一个余额，部署后默认是0，通过`address(合约地址).balance`可以拿到其余额。

之后只要通过`payable external`声明一个函数，就开通了从外部转账到合约的渠道，**注意这个函数默认是没有实现的，意味着任何外部转入，只会有一个作用，就是增加当前合约的余额，但是合约不会记录除此之外的其他信息，也不会有任何操作，相当于纯捐款**。还注意到这个函数没有任何形参，但实际上外部调用时可以传入参数。

测试此功能的代码：

```javascript
it("test transfer and balance", async () => {
    const b = await contract.getBalance();
    expect(b).to.equal(0x0);
    await contract.deposit({value: hre.ethers.parseEther("1.0")}); // 注意这里传入了一个对象，即使原方法没有形参
    const b2 = await contract.getBalance();
    console.warn(`after deposit, contract balance is ${b2}`);
    const [owner, otherAccount] = await hre.ethers.getSigners(); // 在chai里面测试是这样写，在合约里面直接用msg.sender就可以拿到当前调用合约的地址
    const ownerBalance = await hre.ethers.provider.getBalance(owner.address); // chai里面是这样写，在合约里面直接用address(addr).balance就可以拿到账户对应的余额
    console.warn(`after deposit, owner balance is ${ownerBalance}`);
});
```

注意外部调用payable修饰的函数时，只要传入一个`{ value: XXX }`的对象，这里的XXX就是金额，就可以从当前账户内划走余额。再次提示，如果直接这样调用，相当于捐款给智能合约，因为智能合约的实际方法体内根本不记录捐款者信息。

此外，还可以在合约内声明一个类似`constructor`的特殊函数`receive`，以使得外界不需要查看合约的ABI就可以进行转账：

```solidity
contract SharedAccount {
    // ...之前代码省略
    receive() external payable {
        deposit(); //这个实现看之前代码
    }
}
```

注意这个`receive`是一个特殊函数，它没有关键字`function`修饰，也禁止声明入参，类型必须是external payable，禁止声明返回类型，这样外部调用时就可以不需要参考ABI写函数签名。

单元测试代码：

```javascript
it("test default payable function", async() => {
    const contractAddr = await contract.getAddress();
    const tx = await owner.sendTransaction({ // 通过sendTransaction来发起向合约的转账，但是不需要依赖合约的ABI，只要能拿到合约地址就可以
        to: contractAddr,
        value: hre.ethers.parseEther("1.0")
    });
    await tx.wait();
    const result = await contract.getBalance();
    expect(result[0]).to.above(balance1);
    expect(result[1]).to.above(balance2);
    balance1 = result[0];
    balance2 = result[1];
});
```



#### 合约向EOA转账

之前的操作都是外部通过ether.js操作合约，修改状态或者存入资金，这里介绍如何在合约内向EOA转账。

这里给出一个简单的智能合约的例子，它接收捐款，每次的捐款分2等分（如果是3等分，直接做除法会导致捐款丢失，因为SOLIDITY也有浮点运算不够精确的问题），之后把这些金额转入它初始化时预定的2个EOA账户内。

智能合约写法：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.28;
import "hardhat/console.sol";

contract SharedAccount {
    address owner1;
    address owner2;

    constructor(address addr1, address addr2) {
        owner1 = addr1;
        owner2 = addr2;
        console.log("owner all set", owner1, owner2);
    }

    function getBalance() external view returns (uint256, uint256) {
        return (owner1.balance, owner2.balance);
    }

    function deposit() external payable {
        uint money = msg.value; // get the value of transaction
        require(money > 0, "must send some ETH");

        (bool success1, ) = owner1.call{value: money / 2}("");
        require(success1);

        (bool success2, ) = owner2.call{value: money / 2}("");
        require(success2);
    }
}
```

注意上述`deposit`函数的内部实现：

- 可以通过`msg.value`在一个`payable`修饰的函数内获取合约调用者传递过来的金额。
- `require(condition, error_message)`表示一个校验方法，入参1就是校验的表达式，也可以执行某个函数，如果结果是true，则继续执行后续代码，如果结果是false，则返回入参2的错误消息，并回滚交易
- `(bool success1, ) = owner1.call{value: money / 2}("");`这个重点解释一下。owner是一个地址类型，任何地址类型都包含通用的方法，比如balance，transfer，以及call，上述代码整体就是基于call方法的一个调用。

`(bool result, bytes memory returnData) = address(some_addr).call{value: amount}("encodedFunctionCall");`这个是完整的call方法的使用场景。call在SOLIDITY内是一个底层方法，用于调用其他智能合约或者往其他EOA账户发送ETH，要调用call首先必须拿到一个地址类型的变量，其次传递对应参数，上述代码的例子就是发送ETH而非合约调用，所以`{value: amount}`的部分是必须的，**如果是合约调用且不涉及ETH转账，则`{value: amount}`的部分可以省略，但是一般会需要通过`{gas: gas_amount}`来设置能使用的GAS**。后面的括号跟随的就是合约调用相关的函数签名和入参，这里是转账到EOA账户所以不涉及，最后返回值是一个有限复合集`(bool result, bytes memory returnData)`，元素1表示执行结果，true当然就是成功，元素2表示在合约调用场景下返回目标合约的执行结果，这里是EOA账户转账所以不涉及，因此也不需要声明一个形参来接收。

上述代码因为是需要向2个账户转账，因此每转账一次都要使用`require`来确认交易结果，只要有一个交易结果是失败的，那么整个函数的执行都要回滚。

之后写单元测试，用例如下：

```javascript
const { expect } = require("chai");
const hre = require("hardhat");

let contract;
let balance1;
let balance2;

describe("test begins", function() {
    this.beforeAll(async () => {
        const [_owner, other1, other2] = await hre.ethers.getSigners(); // 可以在单元测试时构造大量EOA账户
        const contractFactory = await hre.ethers.getContractFactory("SharedAccount");
        contract = await contractFactory.deploy(other1.address, other2.address);
        console.warn("contract deployed");
    });
    it("test initial balance", async () => {
        const result = await contract.getBalance();
        balance1 = result[0];
        balance2 = result[1];
        console.warn(result[0], result[1]);
    });
    it("test deposit", async() => {
        await contract.deposit({value: hre.ethers.parseEther("2.0")});
        const result = await contract.getBalance();
        expect(result[0]).to.above(balance1); // 存入金额后，各个账户的当前金额应该比之前存入要高
        expect(result[1]).to.above(balance2);
    });
});
```

注意`hre.ethers.getSigners()`，它可以在单元测试时构造大量包含余额的EOA账户，只要用一个数组去接收就可以，第一个元素是合约部署者，第二个及之后的都是随机的EOA账户。



#### 终止交易

智能合约也需要处理异常情况，一般来说当出现异常时应该立刻终止交易并回滚状态。

上文提到的`require`就是终止交易的方式之一，`require(bool_condition, error_message)`表示校验，如果校验失败则返回错误信息并终止交易，如果校验成功则执行后续代码。

还可以使用`revert`配合自定义错误来优化GAS消耗，因为`require`的错误消息是字符串类型，会消耗更多GAS来表达错误信息，而自定义错误可以通过类型名称来传递错误信息，此外还可以包含入参以提供更多相关错误信息，因此可以优化GAS消耗。

最后，如果需要确保代码执行到某个阶段之前不能有代码内的运算错误，则可以用`assert`，用法是`assert(bool_condition)`。

assert用例，顺便介绍一下SOLIDITY的金额单位用法：

```solidity
assert(1 wei == 1);
assert(1 gwei == 1e9);
assert(1 ether == 1e18);
```

声明自定义错误类型，可以声明在合约外部，这样一个SOL文件内的多个合约都可以共享此错误类型，或者声明在合约内部，确保只有当前合约可以使用此错误类型：

```solidity
error FundsNotEnough(uint minimal, uint actualAmount);
```

配合`revert`的使用：

```solidity
revert FundsNotEnough(1 ether, 0.1 ether);
```

这样如果交易回滚，则调用者可以直接从日志中看到触发了`FundsNotEnough`错误，以及对应的参数。



#### 合约的this和自毁（即将废弃）

`this`在合约内也是有用的，指代的是当前合约的地址的字符串，因此`address(this)`表示的是当前合约的地址。

合约如果需要升级，那么部署了新合约后，最好把老合约销毁，因此EVM提供了销毁合约的操作。这里有一个问题，既然合约销毁也会导致EVM状态变动，因此也会消耗GAS，那么为什么要花费GAS去做一个没什么用的事情呢？所以以太坊社区规定，**销毁合约会返还ETH**，因此鼓励开发者去销毁不会再用到的合约。

当然合约销毁也会带来一个问题，即**销毁的是代码和Storage，而不是地址**，因此如果还有其他消息不灵通的人往这个地址发送ETH，这些ETH就相当于永远丢失，因此**还有一种策略是不销毁合约，而是设置一个运行状态标记**，当需要让合约下线时，用特定账户发送一个切换运行标记的方法，这样后续所有往这个地址发送ETH的行为都会被回滚，以避免财产损失。

注意，以下代码会在将来的EVM硬分叉中失效，即**从某个时点开始，合约将无法自行销毁（即源码内编写selfdestruct不会有任何作用），只能采用下线策略**。

```solidity
function destroy() external {
    selfdestruct(payable(msg.sender));
}
```



#### 合约调用其他合约

合约调用其他合约的情况会比较复杂，一般来说涉及一个根本问题，是否发起了交易（即状态变化），如果不涉及变化，则应该使用`staticcall`而非`call`，如果涉及变化就用call。

此外合约调用从结果上如果会导致状态改变，则当使用ETHERJS进行调用时，返回的会是交易类型，此时需要再等待一次以拿到函数的执行结果，或者交易回执。

以下词语约定，**代理合约，表示直接和EOA交互的合约，目标合约，表示代理合约调用的合约**。

先从最简单的开始，代理合约A调用目标合约B，都不修改状态：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.28;
import "hardhat/console.sol";

contract Message { // 目标合约
    string internal val;
    constructor(string memory _val) {
        val = _val;
        console.log("message created");
    }
    function getMessage() external view returns (string memory) {
        return val;
    }
    function setMessage(string memory newVal) external {
        val = newVal;
    }
}

contract MessageProxy { // 代理合约
    address internal target;
    constructor(address messageAddr) {
        target = messageAddr;
        console.log("proxy created");
    }
    function getMessage() external view returns (bytes memory) {
        (bool result, bytes memory returnData) = target.staticcall(abi.encodeWithSignature("getMessage()"));
        require(result);
        return returnData;
    }
}
```

注意当调用目标合约时，需要拿到目标合约的地址，以及确认调用方法是否会修改状态，如果**不会修改状态，应该使用`staticcall`来调用**，另外`abi.encodeWithSignature`内，入参1是函数签名的字符串，之后都是调用此函数的入参，逗号分隔。

另外注意函数签名的写法，如果目标函数入参是`string memory someVal`，则在`encodeWithSignature`内只需要写`functionName(string)`即可，因为`memory`不是一个变量类型。

之后写单元测试，完整的例子如下：

```javascript
const { expect } = require("chai");
const hre = require("hardhat");
const c_message_abi = require("../artifacts/contracts/Connect.sol/Message.json").abi; // 注意这里要拿到目标合约的ABI，用于解析目标合约的返回值

let c_message;
let c_proxy;

describe("test begins", function() {
    this.beforeAll(async () => {
        c_message = await hre.ethers.getContractFactory("Message").then(f => {
            return f.deploy("first message");
        });
        const addr = await c_message.getAddress();
        c_proxy = await hre.ethers.getContractFactory("MessageProxy").then(f => {
            return f.deploy(addr);
        });
        console.warn("contract deployed");
    });
    it("test proxy get method", async() => {
        const returnData = await c_proxy.getMessage();
        const c_message_interface = new hre.ethers.Interface(c_message_abi); // 这里通过目标合约的ABI创建解析接口
        const message = c_message_interface.decodeFunctionResult("getMessage", returnData); // 用解析接口去解析返回值，返回一个数组，第一个元素就是返回值
        expect(message[0]).to.equal("first message");
    });
});
```

到此还比较顺利，之后是修改状态的例子，首先在代理合约内新增一个方法：

```solidity
function setMessage(string memory newVal) external {
    (bool result, ) = target.call(abi.encodeWithSignature("setMessage(string)", newVal));
    require(result);
}
```

之后单元测试：

```javascript
it("test proxy set method", async() => {
    const tx = await c_proxy.setMessage("a new message"); // 注意这里是一个交易，因此返回的是交易对象
    await tx.wait(); // 需要等待交易被确认，即新区块产生
    const returnData = await c_proxy.getMessage(); // 再用get方法查询变更后数据
    const c_message_interface = new hre.ethers.Interface(c_message_abi);
    const message = c_message_interface.decodeFunctionResult("getMessage", returnData);
    expect(message[0]).to.equal("a new message");
});
```

这里有一个问题，**如果一个合约是修改状态的，它可以有返回值，但是使用ETHER.JS目前还无法直接拿到这个原始的返回值**。所以只能再调用一次合约的查询功能，这个限制我目前没有找到可以绕开的方法。

另外一种在合约内调用其他合约的方法，就是在合约内声明目标合约的interface，当拿到目标合约地址后，通过interface强行转换，进行调用：

```solidity
interface B {
    function storeValue(uint256) external; // 注意接口定义，函数签名不需要包含入参名称，只需要包含入参的数据类型和修饰符
}

contract A {
    function setValueOnB(address b) external {
        B(b).storeValue(22); // 这里直接强转
    }
}
```

虽然这样写更加方便，但是必须确认接口不能写错，而且底层是调用call还是staticcall也不能100%确定，因此还是建议直接用call或者staticcall来处理合约间调用。

从外部调用合约，本质上是通过PROVIDER发送POST请求，请求体包含JSON-RPC格式，最核心的是params里面的data，它本质上是这样的：`getHead4Bytes(keccak256("function_signature"))+padding(function_params)`，最后得到一个HEX字符串。注意它里面没有包含完整的函数签名，只是哈希后的前4个byte再加上入参。**这个data实际上也是EVM内函数互相调用时传递的byte流**。

