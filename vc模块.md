
## 说明
* dao中资产的转账手续费有两种计算方式， 固定金额aUSD或者固定比例， nft只有固定金额aUSD
* 手续费进入dao id账户中，以保留金额的形式存在。要使用必须经过大家同意解锁，自由余额由管理员灵活但有限制的使用
* 未打开中心化交易所转账的资产，只能swap
## 对外开放的方法
1. 设置管理员
    * 代码 `pub fn set_guarders(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            mut guarders: Vec<T::AccountId>,
        )`
    * 参数
        * `dao_id` dao id
        * `guarders` 管理员们
    * 逻辑
        * dao的sudo权限
2. 添加管理员
    * 代码 `pub fn add_guarder(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            guarder: T::AccountId,
        )`
    * 参数 
        * `dao_id` dao id
        * `guarder` 添加的管理员
    * 逻辑
      * dao的sudo权限
3. 删除管理员
    * 代码 `pub fn remove_guarder(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            guarder: T::AccountId,
        )`
    * 参数 
        * `dao_id` dao id
        * `guarder` 删除的管理员


4. 给DAO解锁资金
  * 代码
    ```commandline
    pub fn unreserve(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            asset_id: AssetId,
            amount: BalanceOf<T>,
        )
    ```
  * 参数
    * `dao_id` dao id
    * `asset_id` 资产id
    * `amount` 解保留金额
  * 逻辑
    * Dao的账户资金必须足够
    * 管理员权限
5. 设置转账手续费
      * 代码 `pub fn set_fee(
              origin: OriginFor<T>,
              dao_id: T::DaoId,
              fee: Fee<BalanceOf<T>, Permill>,
          )`
      * 参数
          * `dao_id` dao id
          * `fee` 固定金额或者固定比例
      * 逻辑
        * 管理员权限
6. 打开或者关闭cex转账
      * 代码 `pub fn open_cex_transfer(origin: OriginFor<T>, dao_id: T::DaoId, is_open: bool)`
      * 参数
          * `dao_id` dao id
          * `is_open` 打开或者关闭