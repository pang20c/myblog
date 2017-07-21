title: mysql主从 binlog-do-db 和 binlog-ignore-db
date: 2015-01-04 14:27:11
tags: 
- mysql
category:
- mysql
---

和 binlog-do-db，binlog-ignore-db对应的是replicate-do-db,replicate-ignore-db 前者对应的是master后者则是在从服务器上
检查的顺序是 
1. 如果没有设置任何的binlog-do-db 和 binlog-ignore-db则记录全部操作 如果有继续后面判断
2. 是否有use任何库 如果没有则不记录日志 如果有继续后续判断
3. 如果设置了 binlog-do-db 那么只记录binlog-do-db的数据库 其他的都不记录 如果没有设置任何binlog-do-db 才会去判断binlog-ignore-db
4. 判断是否在binlog-ignore-db 如果有则不写入日志 如果没有则写入日志


