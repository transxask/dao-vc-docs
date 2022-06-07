
## 作用
* 执行需要议会权限去执行的方法
## 重要对外方法
* 执行需要议会权限去执行的方法
   * 代码 `pub fn do_as_collective(
           origin: OriginFor<T>,
           dao_id: T::DaoId,
           call: Box<<T as dao::Config>::Call>,
       )`
     * 逻辑
         * 这个方法如果执行成功， 那么call就会获取dao-account-id的权限, 然后执行
         * 这个最终可以让dao的账户跟普通用户一样去执行其他普通方法