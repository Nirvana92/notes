内存, cpu, io

io设备: 磁盘, 网卡

VFS: 虚拟文件系统. 上层[应用层] 统一使用VFS. vfs 对app 做了接口抽象。隔离了物理磁盘。

文件: inode id

fd: 文件描述符。给每个app 提供的一个指针去读取文件内容。



硬连接: 

```shell
> ln test.txt t.txt
```

两个文件的inode号是一样的。两个路径变量名指向同一个物理文件。

```shell
# 查看文件状态
> stat test.txt
```

相当于两个引用指向同一个变量。

软连接: 

```shell
> ln -s test.txt t.txt
```

两个文件的inode 号是不一样的。删除test.txt , t.txt 的连接报红。