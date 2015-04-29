---
layout: post
title: Hbase Flush
categories:
- 人
tags:
- 书
---



#Hbase Flush

概念：


stripe compaction

矛盾：

0. 吞吐量达->region多->store多->memstore多->flush多->compact多->影响读写
1. 某rowkey范围大量put，但要整个store做major compact.
2. split代价大

解决：

level:

2. 分level0，level1:临时mem flush,bulkload文件，stripe文件
7. level0-1上升合并。
4. 容错有level1回退level0重新compaction
9. 算法leve0-n:1.同一层rowkey不重2.边合并边删除：lru落leve低，分层合并不是所有的合并。
10. compaction风暴层数要少

level1-stripe:
1. region下按rowkey划分stripe
3. 划分规则:大小自动切，miniregions个数
5. get扫面更聚集到level1上
6. scan stripe排序？//snapshot不就已经排序了吗？
8. 提升split,直接定位拷贝




---------------


.Yali
2.jiekout jieou 
3.chengdu drquery git branch –r 
4. 返回数据索引 
5.分页 




-------
0.flush 刷region级的，
1. 排序：region内memstore 
2. bestflushableregion选region:store的hbase.hstore.blockingStoreFiles不到，memstore最多的region：. 不到且最多
3. bestanyregion maxMemstore
4. 3超2两倍的话，有compaction也选这个reigon,3，否则选2
5. region一个 mvcc，读写并发，需要写memstore
6. flush阻塞row更新锁，snapshot->memstore.
7. 而mvcc写的是memstore， mvcc读版本到当前写版本
8. mvcc对memstore的updatelock读锁释放，flush要等待mvcc写锁释放，等待目前的写版本号
