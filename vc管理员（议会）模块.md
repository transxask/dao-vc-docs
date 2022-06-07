
## 说明
* 这个模块执行的提案只有doas模块中的 `pub fn do_as_collective(
			origin: OriginFor<T>,
			dao_id: T::DaoId,
			call: Box<<T as dao::Config>::Call>,
		)`, 其他外部方法均不支持，换言之就是要操作其他方法，必须通过doas模块
## 重要数据结构
* 权限
```angular2html
pub enum DoAsEnsure<Pro, C> {
	Proportion(Pro), // 管理员多少比例能过
	Member, // 其中一个管理员执行能过
	Members(C), // 多少个管理员执行能过
	Root, // 只有dao的公投或者root才能过
	NoPermission, // 任何人都没有权限
}
```
## 重要方法

1. 代理机构成员直接执行
    * 代码 `pub fn execute(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        proposal: Box<<T as Config<I>>::Proposal>,
        #[pallet::compact] length_bound: u32,
    )`
        
    * 逻辑
        * 任一代理机构成员直接执行
        * 不需要投票
2. 代理机构成员发起提案
    * 代码 `pub fn propose(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        #[pallet::compact] threshold: MemberCount,
        proposal: Box<<T as Config<I>>::Proposal>,
        #[pallet::compact] length_bound: u32,
    )`
3. 代理机构成员给提案投票
    * 代码 `pub fn vote(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        proposal: T::Hash,
        #[pallet::compact] index: ProposalIndex,
        approve: bool,
    )`
4. 任何人关闭提案（投票通过可执行或者已经过期）
    * 代码 `pub fn close(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        proposal_hash: T::Hash,
        #[pallet::compact] index: ProposalIndex,
        #[pallet::compact] proposal_weight_bound: Weight,
        #[pallet::compact] length_bound: u32,
    )`
## 辅助方法
1. 给每个方法设置权限
    * 代码 `pub fn set_ensure_for_every_call(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        call: Box<<T as dao::Config>::Call>,
        ensure: DoAsEnsure<Proportion<MemberCount>, MemberCount>,
    ) `
    * 逻辑
        * 需要dao的root权限
2. 直接否决提案
    * 代码 `pub fn disapprove_proposal(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        proposal_hash: T::Hash,
    )`
    * 逻辑
        * 需要dao的root权限
3. 设置代理机构投票时长
    * 代码 `pub fn set_motion_duration(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        duration: T::BlockNumber,
    )`
    * 逻辑
      * 需要dao的root权限
4. 设置代理机构提案数量上限
    * 代码 `pub fn set_max_proposals(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        max: ProposalIndex,
    )`
    * 逻辑
        * 需要dao的root权限
5. 设置代理机构人数上限
    * 代码 `pub fn set_max_members(
        origin: OriginFor<T>,
        dao_id: T::DaoId,
        max: MemberCount,
    )`
    * 逻辑
        * 需要dao的root权限