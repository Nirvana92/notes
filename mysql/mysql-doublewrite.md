#### 双写缓冲区

> doublewrite 缓冲区是一个存储区[InnoDB writes pages flushed from buffer pool 和在writing the pages之前]。在该存储区中，操作系统crash可以从doublewrite 中恢复数据。

虽然数据被写入两次，但是不会增加io 操作。只需调用一次操作系统 fsync() 。除非 `innodb_flush_method`设置为 `O_DIRECT_NO_FSYNC`



默认情况下会启用双写缓冲区。要禁用通过`innodb_doublewrite=0`。

如果系统表空间文件[ibdata文件]位于支持原子写的Fusion-io设备上。则自动禁用双写缓冲。建议`innodb_flush_method=O_DIRECT`



写入到 doublewrite buffer -> 写**pages** 信息到 **data files**; doublewrite 需要两次写入。但是通过fsync() 只需要一次io操作。



doublewrite 解决了 partial write[页断裂] 问题.