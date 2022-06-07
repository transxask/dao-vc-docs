# 说明
* 这是dao的最高权限，最上层的投票，决定dao管理员，群重要参数的更改。类似于substrate的公投模块
* 议会能够执行的交易，公投一定能够执行，反之不成立
* 如果是dao币，那么以愿意投的token数作为投票权重，如果是nft那么则以人数作为投票权重
* 每一个时间窗口只能有一个公投
* 每个NFT投票基本权重是1个DOLLAR
# 重要存储
略
# 重要数据结构
```angular2html
pub enum Vote<TokenId, Balance> {
	NftTokenId(TokenId), // 哪个NFT token
	FungibleAmount(Balance) // 多少金额的普通代币
}
```
```angular2html
pub enum Conviction {
	X1, // 一倍权重， 锁仓一个月
	X2, // 2倍权重，锁仓2个月
	X3, // 3倍权重， 锁仓3个月
	X6, // 6倍权重， 锁仓6个月
}
```
```commandline
pub struct Tally<Balance> {
	/// The number of aye votes, expressed in terms of post-conviction lock-vote.
	pub ayes: Balance, // 赞成票多少
	/// The number of nay votes, expressed in terms of post-conviction lock-vote.
	pub nays: Balance, // 反对票多少
}
```
```angular2html
pub enum Attitude {
  AYES, // 赞成
  NAYS, // 反对
}
```

```commandline
pub struct VoteInfo<SecondId, Vote, BlockNumber, VoteWeight, Attitude, ReferendumIndex> {
	second_id: SecondId, // Dao 映射到的群体id
	vote: Vote, // 票
	attitude: Attitude, // 赞成或是反对
	vote_weight: VoteWeight, // 票换算成的投票权重
	unlock_block: BlockNumber, // 可以解锁的高度
	referendum_index: ReferendumIndex, // 公投索引
}
```
```commandline
pub struct ReferendumStatus<BlockNumber, Call, Balance> {
	/// When voting on this referendum will end.
	pub end: BlockNumber,  // 投票结束区块高度
	/// The hash of the proposal being voted on.
	pub proposal: Call, // 提案
	/// The delay (in blocks) to wait after a successful referendum before deploying.
	pub delay: BlockNumber, // 延迟执行的时间
	/// The current tally of votes in this referendum.
	pub tally: Tally<Balance>, // 投票的统计
}
```
```commandline
pub enum ReferendumInfo<BlockNumber, Call, Balance> {
	/// Referendum is happening, the arg is the block number at which it will end.
	Ongoing(ReferendumStatus<BlockNumber, Call, Balance>),
	/// Referendum finished at `end`, and has been `approved` or rejected.
	Finished { approved: bool, end: BlockNumber }, // 已经结束
}
```
# 重要方法
1. 发起一个提案
  * 代码
    ```commandline
    pub fn propose(
			origin: OriginFor<T>,
			dao_id: T::DaoId,  // dao id
			proposal: Box<<T as dao::Config>::Call>, // 提案
			#[pallet::compact] value: BalanceOf<T>, // 锁住的金额
		)
    ```
    * 逻辑
      * dao存在，并且提案允许执行
      * 锁住金额
      * dao中提案数目没有达到上限

2. 对提案进行一个second
  * 代码
    ```commandline
    pub fn second(
			origin: OriginFor<T>,
			dao_id: T::DaoId, // dao id
			#[pallet::compact] proposal: PropIndex, // 提案索引
		)
    ```
  * 逻辑
    * 提案存在
    * 锁住金额
    
  > 如果是非nft资产，那么抵押金额应该在创建dao时确定，并且值不能为0
3. 开启一个新的公投
  * 代码
    ```commandline
    pub fn start_table(origin: OriginFor<T>, dao_id: T::DaoId)
    ```
    * 逻辑
      * 任何人
      * 已经到公投开始的周期（一轮一个）
      * 提案金额最大的升级为公投

4. 对公投进行投票
  * 代码
    ```commandline
    pub fn vote(
			origin: OriginFor<T>,
			dao_id: T::DaoId, // dao id
			index: ReferendumIndex, // 公投索引
			vote: T::Vote, // 票
			conviction: T::Conviction, // 信仰（愿意锁仓多久）
			attitude: Attitude, // 赞成或是反对
		)
    ```
  * 逻辑
    * 存在此公投
    * 根据信仰锁住金额，并且换算成投票权重
5. 撤销自己的投票
  * 代码
  ```
  pub fn cancel_vote(
			origin: OriginFor<T>,
			dao_id: T::DaoId, // dao id
			index: ReferendumIndex, // 公投索引
		)

  ```
  * 逻辑
    * 公投存在并且自己已经投票


6. 执行公投
  * 代码
    ```commandline
    pub fn enact_proposal(
			origin: OriginFor<T>,
			dao_id: T::DaoId, // dao id
			index: ReferendumIndex, // 公投索引
		)
    ```
  * 逻辑
    * 过期或是投票通过（赞成票大于反对票， 并且投票阀值达到要求）可执行
6. 解锁自己投票锁住的金额
    * 代码 `pub fn unlock(origin: OriginFor<T>)`
    * 逻辑
        * 自己透过票
        * 有可以解锁的
7. 解锁提案或者附议锁住的KICO
    * 代码 `pub fn unreserve(origin: OriginFor<T>)`
    * 逻辑
        * 有被锁住的金额
8. 为每个方法设置最小的投票权重（相当于投票率）
    * 代码 `pub fn set_min_vote_weight_for_every_call(origin: OriginFor<T>, dao_id: T::DaoId, call: Box<<T as dao::Config>::Call>, min_vote_weight: BalanceOf<T>)`
    * 参数：
        * `dao_id`  dao id
        * `call` 方法
        * `min_vote_weight` 投票权重
    * 逻辑
        * dao存在
        * 该方法dao可执行
# 其他次要的治理方法
1. 设置公投时间窗口
2. 设置共众投票时长
3. 设置提案个数上限
4. 设置发起提案需要抵押的金额
> 以上这几个方法的参数范围要作合理的限制（推荐范围）；并且均需要上面的公投权限

