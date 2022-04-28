# 说明
* 这是群的最高权限，最上层的投票，决定群管理员，群重要参数的更改。类似于substrate的公投模块
* 如果是dao币，那么以愿意投的token数作为投票权重，如果是nft那么则以人数作为投票权重
* 每一个时间窗口只能有一个公投
* 应该对可以使用下面公投的方法进行一个范围限定。比如只用在vc相关的模块
# 重要存储
略
# 重要数据结构
```commandline
pub enum KicoAmount {
    可变的(BalanceOf<T>),
    不可变的，
}
```

pub enum Attitude {
  AYES,
  NAYS,
}
```commandline
pub struct VoteInfo {
    attitude: Attitude,
    amount: KicoAmount,
}
```
```commandline
pub struct Votes<AccountId, BlockNumber> {
    /// dao id
    dao_id: u64,
	/// The proposal's unique index.
	index: ProposalIndex,
	/// The number of approval votes that are needed to pass the motion.
	threshold: MemberCount,
	/// The current set of voters that approved it.
	ayes: Vec<(AccountId, Balance)>,
	/// The current set of voters that rejected it.
	nays: Vec<(AccountId, Balance)>,
	/// The hard end time of this vote.
	end: BlockNumber,
}
```
```commandline
pub struct KicoProposalInfo {
    /// 提案索引
    index: ProposalIndex,
    /// 提案hash
    proposal_hash: Hash,
    /// 提案
    proposal: Proposal,
    /// 提案附议需要抵押的金额
    second_amount: BalanceOf<T>,
    /// 提案原因
    reason: Vec<u8>,
}
```
# 重要方法
1. 发起一个提案
  * 代码
    ```commandline
    pub fn propose(
            origin: OriginFor<T>,
            dao_id: u64,  // dao id
            proposal: Box<<T as Config<I>>::Proposal>,  // 提案对应的交易
            reason: Vec<u8>，// 提案原因
        )
    ```
    * 逻辑
      * 如果是可变的资产，需要抵押一定的金额（并且记录这个金额)；如果是nft，则冻结这个nft
      * 这个dao存在
      * dao中提案数目没有达到上限

2. 对提案进行一个second(nft的话是冻结当前active的nft，dao币是对保留余额进行操作)
  * 代码
    ```commandline
    pub fn second(
            origin: OriginFor<T>,
            dao_id: u64  // dao id
            #[pallet::compact] proposal: PropIndex, // 提案索引
        )
    ```
  * 逻辑
    * 如果是可变资产，抵押跟提案发起人一模一样的金额；如果是nft，则冻结这个nft
  > 如果是非nft资产，那么抵押金额应该在创建dao时确定，并且值不能为0
3. 新的时间窗口一到，手动对提案进行处理，second票数最多的进入公投
  * 代码
    ```commandline
    fn close_window(
        origin: OriginFor<T>,
        dao_id: u64,  // dao id

    )
    ```
    * 逻辑
      * 任何人
      * 提案至少一个

4. 对公投进行投票
  * 代码
    ```commandline
    fn vote(
        origin: OriginFor<T>,
        dao_id: u64,  // dao id
        #[pallet::compact] proposal: PropIndex, // 提案索引
        vote_info:
        amount: KicoAmount, // 支持的金额
    )
    ```
  * 逻辑
    * 存在此公投
    * 如果是dao币，金额必须大于0，并且需要抵押
5. 追加投票
  * 代码
  ```
  fn extra_vote(
    origin: OriginFor<T>,
    dao_id: u64,  // dao id
    extra_amount: Balance,
  )

  ```
  * 逻辑
    * 公投存在并且自己已经投票

6. 取消投票
  * 代码
    ```commandline
    fn cancel_vote(
        origin: OriginFor<T>,
        dao_id: u64,  // dao id
        #[pallet::compact] proposal: PropIndex, // 提案索引
    )
    ```
  * 逻辑
    * 存在公投并且自己已经投票
6. 关闭公投（执行）
  * 代码
    ```commandline
    fn close_public_proposal(
        origin: OriginFor<T>,
        dao_id: u64,
        #[pallet::compact] proposal: PropIndex,
    )
    ```
  * 逻辑
    * 过期或是可执行
    * 提案在公投队列中
6. 提案发起人关闭提案
  * 代码
    ```commandline
    fn cancel_proposal(
        origin: OriginFor<T>,
        dao_id: u64,
        #[pallet::compact] proposal: PropIndex,
    )
    ```
  * 逻辑
    * 提案存在并且还没有进入公投
    * 自己是提案的发起人
    * 归还second用户的所有抵押金额
# 其他次要的治理方法
1. 设置公投时间窗口
2. 设置共众投票时长
3. 设置提案个数上限
4. 设置发起提案需要抵押的金额
> 以上这几个方法的参数范围要作合理的限制（推荐范围）；并且均需要上面的公投权限
# 有意思的地方
* 这个公投最终执行时候获取的是root权限
* 这个公投结束用root权限去执行，那就意味着其实可以通过原生的公投模块去执行他要执行的交易

