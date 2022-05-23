# 存储
* Dao id => 类型
# 发起DAO
1. 创建一个DAO
  * 代码
    ```commandline
    fn create_dao(
        origin: OriginFor<T>,
        class_id_or_asset_id: Id,
        info: Info,
    )
    ```
  *逻辑
    * 有选择性的(选择nft或是Dao币)
    * 创建nft类或是创建可分割资产
    * Dao的id是自增的u64类型
    * 为每个Dao创建一个链上账户
    * 设置每个dao发起公投需要抵押的金额
2. 为一个已经存在的资产创建DAO
  * 代码
    ```commandline
    fn create_dao_for_asset(
        origin: OriginFor<T>,
        class_id_or_asset_id: Id,
    )
    ```
  * 必须是该资产的创建人
  > 除了不用额外创建资产，其余跟1步骤相同
# 管理DAO提案
1. 管理者申请解锁资金
  * 代码
    ```commandline
    fn request_unreserve(
        origin: OriginFor<T>,
        dao_id: u64,
        request_id: u64,
        amount: Balance,
    )
    ```
  * 逻辑
    * Dao的账户资金必须足够, 并且kico达到最低要求以上
    * 只有管理者才能够提议案
  > 每个dao交易扣除的u均进入reserve中
2. 拒绝管理者的申请
  * 代码
    ```commandline
    fn reject_unreserve(
        origin: OriginFor<T>,
        dao_id: DaoId
        request_id: u64,
    )
    ```
  * 逻辑
    * 需要dao公投获取root权限
3. 同意管理者的申请
  * 代码
    ```commandline
    fn approve_unreserve(
        origin: OriginFor<T>,
        dao_id: DaoId
        request_id: u64,
    )
    ```
  * 逻辑
    * 需要dao公投获取root权限
4. 关闭提案
   * 代码
   ```commandline
    fn close(
        origin: OriginFor<T>,
        dao_id: DaoId
        request_id: u64,
   )
    ```
   * 逻辑
     * 任何人都可以操作
5. 设置转账需要多少手续费
  * 代码
    ```commandline
    fn set_fee(
        origin: OriginFor<T>,
        dao_id: u64,
        fee: Amount, //dao币可以按照转账金额比例或者固定金额，nft是固定金额
    )
    ```
  * 逻辑
    * 需要dao公投获取root权限

# DAO管理者群体参与ico
* 操作任何ico中的方法
  * 代码
    ```commandline
    fn ico_operate(
        origin: OriginFor<T>,
        dao_id: DaoId
        proposal: Box<<T as Config<I>>::Proposal>,
    )
    ```
  * 逻辑
    * 需要dao治理模块获取dao_id生成的账号权限去执行（也就是获取过半管理员的同意）
    * 对proposal进行过滤
    * 有可能在需要kyc审核的地方通过不了
# ico后管理
1. Dao发起人回购销毁代币
  * 代码
    ```commandline
    fn burn_operate(
        origin: OriginFor<T>,
        dao_id: u32,
        proposal: Box<<T as Config<I>>::Proposal>,

    )
    ```
  * 逻辑
    * 以dao_id转换成的账户身份去执行， 也只有这个才能执行
    * 对proposal进行过滤
