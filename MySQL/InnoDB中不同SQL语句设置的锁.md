# InnoDB中，不同SQL语句设置的锁

一个锁定读、UPDATE或者DELETE通常会在SQL语句处理扫描的每个索引记录上设置记录锁。WHERE语句中是否存在排除该行的条件并不重要。InnoDB不会记得确切的WHERE条件，只会知道扫描了哪些索引范围。锁通常是next-key锁，它也会阻止插入记录前的“间隙”里。然而，可以显式禁用间隙锁，这会导致next-key锁没有用。

如果在搜索中使用二级索引并且设置了排他索引记录锁，InnoDB也还会检索相应的聚集索引记录并对其设置锁。

如果对于你的语句没有合适的索引，并且MySQL必须扫描整个表来处理该语句，则该表的每一行都会被锁定，从而阻止其它用户对该表的所有插入。所有创建好的索引很重要，这样你的查询就没有必要扫描过多的行。

InnoDB设置特定类型的锁方式如下。

- SELECT ... FROM是一致性读，读取数据库的快照并且不设置锁，除非事务隔离级别设为SERIALIZABLE。对于SERIALIZABLE级别，搜索在它遇到的索引记录上设置共享的next-key锁。但是，对于使用唯一索引去搜索一个唯一行锁定行的语句，只需要一个索引记录锁。

- 对于锁定读取（SELECT with FOR UPDATE or LOCK IN SHARE MODE）、UPDATE和DELETE语句，采用的锁取决于语句是使用具有唯一搜索条件的唯一索引还是范围类型搜索条件。

  - 对于具有唯一搜索条件的唯一索引，InnoDB只锁定找到的索引记录，而不锁定记录前的间隙。

  - 对于其它搜索条件，和非唯一索引，InnoDB锁定扫描的索引范围，使用间隙锁（gap locks）或者next-key locks来阻止其它会话插入范围所覆盖的间隙。
  
- 对于搜索遇到的索引记录，在某些事务隔离级别，SELECT ... FOR UPDATE会阻止其它会话执行SELECT ... LOCK IN SHARE MODE或读取。一致性读取忽略对读取视图中存在的记录设置的任何锁。

- UPDATE ... WHERE ...在搜索遇到的每条记录上设置一个独占的next-key锁。但是，对于使用唯一索引去搜索唯一行的锁定行的语句，只需要索引记录锁。

- 当UPDATE修改一个聚集索引记录，隐含的锁会影响二级索引记录。当UPDATE在插入新的二级索引记录之前执行重复检查扫描时，以及插入新的二级索引记录时，该操作还会对受影响的二级索引记录使用共享锁。

- DELETE FROM ... WHERE ...在搜索遇到的每条记录上设置一个独占的next-key锁。但是对于使用唯一索引锁定行以搜索唯一行的语句，只需要索引记录锁。

- INSERT在插入的行上设置排他锁。这个锁是索引记录锁，不是next-key锁（即没有间隙锁），并且不会阻止其它会话插入行之前插入间隙。

  在插入行之前，设置了一种称为意向间隙锁（intention gap lock）的间隙锁。这个锁表示插入的意图，如果插入到同一索引间隙的多个事务没有插入间隙内的相同位置，则它们不需要相互等待。假设存在值为4和7的索引记录。两个尝试插入值为5和6的单独事务在获得插入行的排他锁之前都使用插入意向锁锁定了4和7之间的间隙，因为插入的行是不冲突的，所以不会相互阻塞。

- INSERT ... ON DUPLICATE KEY UPDATE不同于简单INSERT在于，当发生重复键错误时，更新行放置的是排他锁（exclusive lock）而不是共享锁（shared lock）。对重复的主键值使用独占索引记录（index-record）锁。对重复的唯一键值使用独占的next-key锁。

- 如果在唯一键上没有冲突，REPLACE就像INSERT一样完成。否则，会在要替换的行设置一个独占的next-key锁。

- INSERT INTO T SELECT ... FROM S WHERE ... 在插入表T的每一行上设置排他索引记录锁（没有间隙锁）。如果事务隔离级别是READ COMMITTED，或者innodb_locks_unsafe_for_binlog是已启用并且事务隔离级别不是SERIALIZABLE，InnoDB使用一致性读来搜索表S（无锁）。否则，InnoDB在表S的行上设置共享next-key锁。InnoDB在最后的一种情况下必须设置锁：在使用基于语句的二进制日志（binary log）的回滚恢复期间，每个SQL语句都必须以与最初执行的方式完全相同的方式执行。

  CREATE TABLE ... SELECT ... 执行SELECT会使用共享next-key锁或者使用一致性读取，就像INSERT ... SELECT语句一样。

  当SELECT使用在REPLACE INTO t SELECT ... FROM s WHERE ... 或者UPDATE t ... WHERE col IN (SELECT ... FROM s ...) 这种结构语句时，InnoDB会在表s的行上设置共享的next-key锁。

- 当初始化表中一个先前指定的AUTO_INCREMENT列时，InnoDB会在与AUTO_INCREMENT列关联的索引末尾设置一个排他锁。

  当innodb_autoinc_lock_mode=0时，在访问自动递增计数器时，InnoDB使用特殊AUTO-INC表锁模式去获取锁并保持到当前SQL语句的末尾（而不是整个事务的末尾）。在持有AUTO-INC表锁时，其它客户端无法插入表中。批量插入（bulk inserts）和innodb_autoinc_lock_mode=1的模式一样。表级别AUTO-INC锁在innodb_autoinc_lock_mode=2时是没有用的。

  InnoDB在不设置任何锁的情况下获取先前初始化的AUTO_INCREMENT列的值。

- 如果在表上定义了外键约束，则任何需要检测约束条件的insert、update或delete都会在它查看以检查约束的记录上设置共享记录锁。InnoDB在约束失败的情况下也会设置这些锁。

- LOCK TABLES设置表锁，但设置的这些锁位于InnoDB层上的MySQL更高层。在innodb_table_locks = 1（默认）和autocommit = 0时，InnoDB会意识到表锁，并且InnoDB上的MySQL层会知道行级锁。

  否则，InnoDB的自动死锁检测无法检测到涉及此类表锁的死锁。此外，因为在这种情况下，更高的MySQL层不知道行级锁，所以有可能在另一个会话当前具有行级锁的表上获得表锁。但是，这不会破坏事务完整性。

- 如果innodb_table_locks=1（默认），LOCK TABLES在每个表上获取两个锁。除了MySQL层的InnoDB表锁外，它还需要一个表锁。为避免获取InnoDB表锁，请设置innodb_table_locks=0。如果没有获取InnoDB表锁，LOCK TABLES即使在表的某些记录被其他事务锁定下也能完成。

  在MySQL 5.6中，innodb_table_locks=0对LOCK TABLES ... WRITE显式锁表没有影响。它会对通过LOCK TABLES ... WRITE或者LOCK TABLES ... READ隐式读写锁表有影响（例如通过触发器）。

- 当事务被提交或中止时，事务持有的所有InnoDB锁都会被释放。因此，对于在autocommit=1模式下InnoDB表上调用LOCK TABLES并没有多大意义，因为所获得的InnoDB表锁会马上被释放。

- 你不能在事务中间锁定其它表，因为LOCK TABLES会执行隐式的COMMIT和UNLOCK TABLES。

[原文](https://dev.mysql.com/doc/refman/5.6/en/innodb-locks-set.html)