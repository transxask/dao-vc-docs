## 说明
* 这个模块用来创建dao
* 目前只能给已经存在的资产创建dao，其他情况后面再加
## 重要数据结构
```angular2html
pub enum SecondId<NftId, TokenId> {
	Nft(NftId), // 哪个类型的nft
	Currency(TokenId) // 哪个普通资产
}
```
## 重要方法
1. 创建dao
    * 代码 `pub fn create_dao(
            origin: OriginFor<T>,
            dao_id: T::DaoId,
            second_id: T::SecondId,
        )`
    * 参数
        * `dao_id` dao id
        * `second_id` 要在哪个资产上创建
    * 逻辑
        * 资产已经存在
        * 自己是资产创建人