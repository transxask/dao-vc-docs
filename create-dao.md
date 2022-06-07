## 说明
* 这个模块用来创建dao
* 目前只能给已经存在的资产创建dao，其他情况后面再加
## 重要数据结构
```angular2html
pub enum SecondId<ClassId, TokenId> {
	NftClassId(ClassId), // 哪个NFT类型
	FungibleTokenId(TokenId) // 哪个普通代币
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
2. dao记录事情（随意给链逼逼一句）
    * 代码 `pub fn dao_remark(origin: OriginFor<T>, dao_id: T::DaoId, _remark: Vec<u8>)`
    * 逻辑
        * dao存在并且是sudo权限