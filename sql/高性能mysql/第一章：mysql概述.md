> tip：本篇是基于《高性能mysql第三版》总结出的笔记类博文，只作为个人学习的记录，若有机会，还是希望大家能实际去阅读该书。  
  
Mysql服务器逻辑架构图如下所示：  
  
![image](https://github.com/Duuuhs/blog/blob/master/res/png/EffectiveMysql_1-1.png)  
    
    
1. 排他锁(X锁),也叫写锁  
2. 共享锁(S锁),也叫读锁  
3. 锁粒度: 在给定的资源上,锁定的数据越少,则系统的并发性越高.  
   **表级锁**:开销最小的策略,锁粒度较大,服务器会为诸如ALTER  TABLE之类的语句使用表级锁,而忽略存储引擎的锁机制.  
**行级锁**:最大程度支持并发处理,同时也带来了最大的锁开销(InnodeDB).  
4. 事务：  
   **ACID**，即原子性,一致性,持久性,隔离性.  
**隔离级别**:    
4.1. **Read Uncommitted**(未提交读);  
4.2. **Read Committed**(提交读,有些版本也叫”不可重复读”),绝大多数数据库使用的隔离级别,但Mysql不是;  
4.3. **Repeatable Read**(可重复读,mysql的默认隔离级别);    
4.4. **Serializable**(可串行化,最高的隔离级别).
![image](https://github.com/Duuuhs/blog/blob/master/res/png/%E9%AB%98%E6%80%A7%E8%83%BDmysql_1-2.png)  
5. 死锁  
![image](https://github.com/Duuuhs/blog/blob/master/res/png/%E9%AB%98%E6%80%A7%E8%83%BDmysql_1-3.png)  
InnodeDB处理死锁的方式:**将持有最少行级排他锁的事务进行回滚**(这是比较简单的死锁回滚算法),对于死锁,大多数情况下只需要重新执行因死锁回滚的事务即可.  
6. **多版本并发控制(MVVC)**:   MVVC是行级锁的一个变种,他在很多情况下避免了加锁操作,实现了非阻塞式读操作,写操作也只是锁定必要的行,因此开销更低.    
**MVVC的实现,是通过保存数据在某个时间点的快照来实现**.也就是说,不管执行多长时间,每个事务看到的数据是一致的.根据事务的开始时间不同,每个事务对于同一张表,  
同一时刻看到的数据可能是不一样的.    
**InnoDb的MVCC:是通过在每行记录后面保存两个隐藏的列来实现的.这两个列,一个保存了行的创建时间,一个保存了行的过期时间(或删除时间).当然存储的并不是实际的时  
间值,而是系统的版本号.每开启一个新的事务,系统版本号都会自动递增**.MVVC目前只在Read Committed与Repeatable Read两个隔离级别下工作.其他两个级别都与MVVC  
不兼容, Read Uncommitted总是读取最新的数据行,而不符合当前事务版本的数据行, Serializable则会对所有读取的行都加锁.  
7. InnoDB存储引擎: **InnoDB采用MVCC来支持高并发**,并且实现了四个级别的隔离标准,默认级别是Repeatable Read(可重复读),**并且通过间隙锁(next-key locking)  
策略防止幻读的出现**.间隙锁使得InnoDB不仅仅锁定查询涉及的行,还会对索引中的间隙进行锁定,防止幻影行的插入.  
