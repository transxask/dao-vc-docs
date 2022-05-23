# 一、介绍
对于DAO，可以抽象理解为一群人共同做一些事。从链的角度来看，这些事输出都是固定的、清晰的。
这些有限数量的方法，我们可以归结为一个类别，也就是实现整个宏伟目标需要的一切工具。任何一个群体想实现相同的目标，做同样的事情，套用这个类别即可。那么，这些群体之间的不同点在于哪里呢，是基于这些一个个方法之上的规则，
群体分别给每个具体的方法怎样的群众权利。而在去中心化的世界里，其实权利可以归结为公投和特定权利代理机构。如果追求绝对的去中心化，理想办法肯定是一人一票，也就是公投。但是，这里有个问题，也就是这样治理的效率是很低的，群众的意愿往往也不尽人意，
不是每一个事件都值得消费群众的精力，公投也不足以应对紧急情况，由此我们想到了一个办法，让大众把部分权利交给代理机构，
这样出现紧急情况的时候，由代理机构进行治理解决。由上面，我们可以得出，链上每个公众（需要群众共同决定）方法，都需要执行权利，这个权利是公投或者机构，
也可以两者兼而有之。而daos主要做的事情是让群体给每一个类别中的方法授予规则（机构、公投），这些规则是动态的。
不管是任何开发者，都能一键创建一个DAO模板，这些DAO模板提供给不同需求的群体使用。
而完成这些的前提，是daos的设计要有足够的抽象，它不需要考虑群众具体用什么投票，不需要考虑具体机构怎么选举出来，
由开发者自由设计。daos是权利分配，是游离在一切具体群体的外层设计，只要你项目中的DAO有公投跟权利代理机构的需求，并且不同群体做相似的事情，比如社交软件的房间管理， 比如多资产的管理， 比如每个项目的ico， 那么这个项目就是你最好的选择。
# 二、项目模块解析
## 1. create-dao
1. 作用：创建dao
2. 重要数据结构
```angular2html
pub struct DaoInfo<AccountId, SecondId> {
	creator: AccountId, // 创建者
	id: SecondId, // dao映射到的具体群体
}
```
3. 重要的数据类型
    * `CallId` 可行性方法的映射id， 用来代指某个方法
        * `TryFrom<<Self as pallet::Config>::Call>`每一个可执行的方法都会转换成一个id
    * `DaoId` 每个dao的id
        * `AccountIdConversion`每一个dao会有一个独特的链上账户
    * `SecondId` 每一个dao映射的具体群体，比如一个具体的房间，一个具体的ico，一个具体资产
        * 比如 `pub enum SecondId <TokenId, NftId>{
            Nft(NftId),
            Currency(TokenId)
      }`
        * `BaseDaoCallFilter`界定每个类型的dao有哪些方法可以执行
        * `Checked`检查创建者是否可以创建这个具体的dao
4. 提供给外部使用的方法
    * 创建dao
        * 代码 `pub fn create_dao(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            second_id: T::SecondId,
        )`
        * 逻辑
            * dao_id不能已经存在
            * 检查是否可以创建（这部分逻辑在`type SecondId中实现`）
## 2. collective
1. 作用：给可执行的方法设置议会权限；议会投票执行需要议会权限的方法
2. 重要数据类型
    * `type GetCollectiveMembers` 通过外部获取议会成员
    * `type MaxMembersForSystem` 议会成员的人数上限
3. 重要方法
    * 代理机构直接执行
        * 代码 `pub fn execute(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            proposal: Box<<T as Config<I>>::Proposal>,
            #[pallet::compact] length_bound: u32,
        )`
        * 逻辑
            * 任一代理机构成员直接执行
            * 不需要投票
    * 代理机构成员发起提案
        * 代码 `pub fn propose(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            #[pallet::compact] threshold: MemberCount,
            proposal: Box<<T as Config<I>>::Proposal>,
            #[pallet::compact] length_bound: u32,
        )`
    * 代理机构成员给提案投票
        * 代码 `pub fn vote(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            proposal: T::Hash,
            #[pallet::compact] index: ProposalIndex,
            approve: bool,
        )`
    * 任何人关闭提案（投票通过可执行或者已经过期）
        * 代码 `pub fn close(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            proposal_hash: T::Hash,
            #[pallet::compact] index: ProposalIndex,
            #[pallet::compact] proposal_weight_bound: Weight,
            #[pallet::compact] length_bound: u32,
        )`
    * 给每个方法设置权限
        * 代码 `pub fn set_ensure_for_every_call(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            call: Box<<T as dao::Config>::Call>,
            ensure: DoAsEnsure<Proportion<MemberCount>, MemberCount>,
        ) `
        * 逻辑
            * 需要dao的root权限
4. 辅助方法
    * 直接否决提案
        * 代码 `pub fn disapprove_proposal(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            proposal_hash: T::Hash,
        )`
        * 逻辑
            * 需要dao的root权限
    * 设置代理机构投票时长
        * 代码 `pub fn set_motion_duration(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            duration: T::BlockNumber,
        )`
        * 逻辑
          * 需要dao的root权限
    * 设置代理机构提案数量上限
        * 代码 `pub fn set_max_proposals(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            max: ProposalIndex,
        )`
        * 逻辑
            * 需要dao的root权限
    * 设置代理机构人数上限
        * 代码 `pub fn set_max_members(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            max: MemberCount,
        )`
        * 逻辑
            * 需要dao的root权限
## 3. doas
1. 作用： 代理机构执行提案的中介，行使代理权利必须经过这个模块
2. 重要数据类型
    * `type DoAsOrigin` 具体的代理机构
3. 重要对外方法
    * 代码 `pub fn do_as_collective(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            call: Box<<T as dao::Config>::Call>,
        )`
    * 逻辑
        * 这个方法如果执行成功， 那么call就会获取dao-account-id的权限, 然后执行
        * 这个最终可以让dao的账户跟普通用户一样去执行其他普通方法
## 4. sudo
1. 作用：dao的创建者代替公投执行root权限（上线后会把这个模块取消）
2. 重要方法
    * 执行dao中需要root权限的方法
        * 代码 `pub fn sudo(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            call: Box<<T as dao::Config>::Call>,
        )`
        * 执行者是dao的创建者或者sudo账户
        * 获取dao-account-id权限去执行call
    * 给dao设置sudo账户
        * 代码 `pub fn set_sudo_account(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            sudo_account: T::AccountId,
		)`
        * 执行者是dao的创建者或者sudo账户
