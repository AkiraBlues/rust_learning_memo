### SOLANA学习笔记



#### SOLANA介绍

它也是一个区块链平台，2017年，一个多年在高通工作的工程师，Anatoly Yakovenko，发表了一篇白皮书（理解为技术文档），里面提出了一个概念，PoH（Proof Of History），历史证明，用于解决不可信网络节点之间的时间同步的问题。这个问题的背景是这样的：

- 对于一个热门的区块链生态来说，时时刻刻都可能发生交易，一个账户可能会在一段时间内给不同的账户进行转账，因此需要确认交易的先后顺序（绝对时间不重要，**重要的是在一堆交易内排列出事件的先后顺序，重要的是顺序**），如果顺序确认错了，会导致交易结果和相关用户的预期完全不同，区块链也就失去了交易的功能
- 比特币通过POW工作量证明来确认交易顺序，但这个顺序实际上是由矿工来确认的，因此可以插队，即同一个账户A，先后给B和C进行转账，如果B的GAS较低，C的GAS较高，那么实际确认的交易顺序会是C先完成，之后才是B。**POW本质上允许插队**。
- 以太坊通过POS，权益证明来确认交易顺序，它本质上只是把矿工通过竞争上岗的机制改为了随机指派矿工的机制，但是保留了POW的高GAS优先原则，因此**实际上还是允许插队**，只是如果大家的GAS都差不多，那么就会按照交易到达交易池的顺序来处理，**即网速好的，广播速度快的交易，会优先到达交易池**，因此在GAS费用差不多的时候，被矿工更优先确认
- SOLANA通过POH，时间证明来标记交易顺序（**不是确认，是标记**）

这里介绍一下POH的机制：

- 核心是一个可验证延迟函数（Verifiable Delay Function, VDF）的哈希运算过程，它的算法保证了计算机不可能通过零延迟完成计算，即每次计算一定会消耗时间，产生一个哈希结果
- 提供额外的一个链条，POH时间链，来记录这个哈希运算结果，每个时间切面上，全链条内只有1个节点来负责这个运算，这个节点称为leader node，领导节点，为了保证去中心化，它不是永久指定的，经过一段时间后会换到其他节点来负责。leader node的最重要职责就是不停地执行VDF拿到哈希结果，并把结果拼接到POH时间链上，所以**这个POH时间链本质上就是和传统的交易区块链并行的区块链**，只不过传统区块链记录的是交易信息，而POH时间链记录的是VDF哈希运算结果，且保证这个结果一定是可验证的，即更后面的区块的VDF结果，一定表示它距离前一个区块经过了一段时间
- **POH链是全局唯一的**，leader node除了不停计算VDF结果，产生新的区块后，也会不停把结果广播到其他的验证节点，以保证所有验证节点都能基于最新的POH链进行交易顺序的验证
- 不同节点上发起的交易依然受网速，传播速度影响，也受GAS影响，POH机制下，交易的先后顺序按照到达leader node的时间确认，但是既然leader node会不停生成POH时间区块，因此**在下一个POH时间区块生成之前**，这批交易会按照GAS费用排序，然后打上时间戳以确认先后顺序，此时GAS高的排序会更靠前。**如果交易A在POH时间区块生成之前到达，交易B在POH时间区块生成之后到达，那么不管B的GAS费用有多高，它也不可能排在A之前**，因为它的时间戳是依赖于POH生成时间区块的间隙的，错过了上一个间隙就只能纳入到下一个间隙

所以2017年高通工程师Anatoly就是通过这个POH技术，加上他的同事Greg Fitzgerald的帮助，创建了SOLANA的原型，最后在2018年命名为SOLANA，这个就是SOLANA区块链，它相比比特币和以太坊，最大的优势就是通过POH大大提高了交易的确认速度，目前是400ms到1秒就可以完成交易确认，**SOLANA的每秒能支持的全球交易数量，已经超过了VISA**。而且因为交易速度很快，GAS费用也是相当低的，甚至比以太坊的L2方案Arbitrum的GAS费用还要更低（**L1费用低于L2，听过吗？**）。



#### 开发环境搭建

最好是原生LINUX，如果是WINDOWS还是建议直接WMWARE安装UBUNTU来搞，ubuntu内需要搞NODEJS。

另外一开始也可以使用solana playground来写智能合约。



#### Hello World

在线创建一个ANCHOR项目，ANCHOR可以理解为开发SOLANA智能合约的RUST版SDK。

删除lib.rs，从头开始写，先引入anchor的模块：

```rust
use anchor_lang::prelude::*;
```

然后创建一个程序的ID，实际部署的时候这块是不会做的，因为SOLANA链会自动生成一个ID，以确保所有程序ID不冲突：

```
declare_id!("my_hello_world_program");
```

之后的部分：

```rust
pub const ANCHOR_DISCRIMINATOR_SIZE: usize = 8; // 后面再解释

#[program] // 把纯RUST模块转为SOLANA兼容的程序或者智能合约
pub mod favorites {
    use super::*;

    pub fn setter() -> Return<()> {

    }

}

#[account] // 把结构体转为SOLANA程序应该保存到链上的数据
#[derive(InitSpace)] // 让SOLANA初始化存储空间
pub struct Favorites {
    pub number: u64,
    
    #[max_len(50)] // 限定一下最大byte长度
    pub color: String,

    #[max_len(5, 50)] // 限定一下最大成员数量和最大byte长度
    pub hobbies: Vec<String>
}

pub struct Setter<'info> { // 和程序暴露的函数最好同名，首字母大写
    pub user: Signer<'info>,
}
```

