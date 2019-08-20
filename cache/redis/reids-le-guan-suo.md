# Reids - 乐观锁



1. 将本次事务涉及的所有 key 注册为观察模式
2. 执行只读操作
3. 根据只读操作的结果组装写操作命令并发送到服务器端入队
4. 发送原子化的批量执行命令 EXEC 试图执行连接的请求队列中的命令
5. 如果前面注册为观察模式的 key 中有一个或多个，在 EXEC 之前被修改过，则 EXEC 将直接失败，拒绝执行；否则顺序执行请求队列中的所有请求
6. redis 没有原生的悲观锁或者快照实现，但可通过乐观锁绕过。一旦两次读到的操作不一样，watch 机制触发，拒绝了后续的 EXEC 执行
