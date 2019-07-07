# Redis 数据持久化

> Redis 数据存储都是在内存里，对数据的更新异步的存储在磁盘里。我们都知道对于内存中的数据如果没有持久化备份，一旦断电将会是一场灾难，在 Redis 中的数据持久化实现有两种策略，分别为 RDB 快照和 AOF 日志。

## RDB

RDB 是 Redis 持久化的一种方式，把当前内存中的数据集快照写入磁盘。恢复时将快照文件直接读到内存里。

**触发机制：**
* save 同步：会阻塞，时间复杂度为 O(n)
* bgsave 异步：异步，Redis 会 fork一个子进程创建到RDB文件，成功之后进行通知，时间复杂度为 O(n)
* 自动：提供一些配置，例如60秒中改变 1000 条数据，自动触发，内部执行策略还是使用的 bgsave

**RDB 的一些问题：**
* 耗时：在进行 save 的时候数据会有多条，是一个 O(n) 的操作
* 耗性能：在 bgsave 机制下会进行 fork 操作，也是需要耗内存的，在 fork 的过程中也会造成阻塞

## AOF

以写日志的方式在执行 redis 命令之后，将数据写入到 AOF 日志文件。

三种策略：
* always：redis 在写命令过程，是先写入硬盘的缓冲区中，缓冲区根据策略写入系统中，always 指写的每条命令都会写入到 AOF 中，保证数据不会丢失。但是I／O 开销会很大。
* everysec：每秒把缓冲区中的数据写入到硬盘，如果出现故障可能会丢失 1 秒的数据。
* no：这个策略根据操作系统定义的进行写入，虽然不需要我们操作，但同时也是不可控的。

AOF重写：

> AOF重写是将那些过期的、重复的命令进行压缩减少，从而达到减少硬盘占用量，提高数据恢复速度。

AOF重写实现方式：

* bgrewriteaof：类似于RDB中的bgsave
* 重写配置：
    * auto-aof-rewrite-min-size：AOF重写需要的最小尺寸
    * auto-aof-rewrite-percentage：AOF文件增长率

```// todo:```