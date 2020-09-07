本文主要总结下重做日志（redo log）、回滚日志（undo log）、二进制日志（binlog）的概念。redo log 是物理日志，undo log 和 binlog 是逻辑日志，物理日志的恢复速度远快于逻辑日志。

redo log 和 undo log 是保证本地事务的，binlog 是用于主从复制的。

![image-20200901102451599](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200901102451599.png)

### binlog

`binlog`记录了数据库表的变更，所以我们可以用`binlog`进行复制（主从复制)和恢复数据。

事务提交的时候，一次性将事务中的 sql 语句（一个事务可能对应多个 sql 语句）按照一定的格式记录到 binlog 中

### redo log

Redo log的主要作用是用于数据库的崩溃恢复

#### **redo流程**

![image-20200901151248449](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200901151248449.png)

第一步：先将原始数据从磁盘中读入内存中来，修改数据的内存拷贝

第二步：生成一条重做日志并写入redo log buffer，记录的是数据被修改后的值

第三步：当事务commit时，将redo log buffer中的内容刷新到 redo log file，对 redo log file采用追加写的方式

第四步：定期将内存中修改的数据刷新到磁盘中

`redo log`是顺序IO，所以**写入的速度很快**，并且`redo log`记载的是物理变化（xxxx页做了xxx修改），文件的体积很小，**恢复速度很快**。

#### redo刷盘时机

- log buffer空间不足时

- 事务提交时

- 后台线程不停的刷刷刷

  后台有一个线程，大约每秒都会刷新一次log buffer中的redo日志到磁盘

- 正常关闭服务器

> 如果整个数据库的数据都被删除了，那我可以用`redo log`的记录来恢复吗？**不能**
>
> 因为功能的不同，`redo log` 存储的是物理数据的变更，如果我们内存的数据已经刷到了磁盘了，那`redo log`的数据就无效了。所以`redo log`不会存储着**历史**所有数据的变更，**文件的内容会被覆盖的**。

### undo log

undo log主要记录的是数据的逻辑变化，为了在发生错误时回滚之前的操作，需要将之前的操作都记录下来，然后在发生错误时才可以回滚。有两个作用：用于事务的回滚；MVCC

#### undo log的写入时机

- DML操作修改聚簇索引前，记录undo日志
- 二级索引记录的修改，不记录undo日志

需要注意的是，undo页面的修改，同样需要记录redo日志。

在InnoDB存储引擎中，undo log分为：

- insert undo log
- update undo log

insert undo log是指在insert 操作中产生的undo log，因为insert操作的记录，只对事务本身可见，对其他事务不可见。故该undo log可以在事务提交后直接删除，不需要进行purge操作。

update undo log记录的是对delete 和update操作产生的undo log，该undo log可能需要提供MVCC机制，因此不能再事务提交时就进行删除，等待purge线程进行最后的删除。

purge线程两个主要作用是：清理undo页和清除page里面带有Delete_Bit标识的数据行。在InnoDB中，事务中的Delete操作实际上并不是真正的删除掉数据行，而是一种Delete Mark操作，在记录上标识Delete_Bit，而不删除记录。是一种"假删除",只是做了个标记，真正的删除工作需要后台purge线程去完成。