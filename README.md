# LSMTree
LSM树技术研究


<pre>
LSM树(Log-Structured Merge Tree)树存储引擎和B树存储引擎一样，同样支持增，删，读，改，
顺序扫描操作。而且通过批量存储技术规避磁盘随机写入问题。当然凡事有利有弊，LSM树和B+树相比，
LSM树牺牲了部分读性能，用来大幅提高写性能。
</pre>

![](https://i.imgur.com/kGCj0dv.png)

内存和磁盘中的数据merge操作

![](https://i.imgur.com/29Uc3zO.png)

<pre>
LSM树的设计思想：
      LSM树的设计思想非常朴素：将对数据的修改增量保存在内存中，达到指定的大小限制后将这些
   修改操作批量写入磁盘，不过读取的时候稍微麻烦些，需要合并磁盘中历史数据和内存中最近修改
   操作，所以写入性能大大提升，读取时可能需要先看是否命中内存，否则需要访问较多的磁盘空间。
   极端情况下，基于LSM树实现的HBASE的写性能比MySQL高一个数量级，读性能则低了一个数量级。

   LSM树原理把一棵大树拆分成N棵小树，它首先写入内存中，随着小树越来越大，内存中的小树会
   flush到磁盘上，磁盘上的树定期可以做merge操作，合并成一棵大树，以优化读性能。

   以上这些大概就是HBase存储的设计主要思想，这里分别对应说明下：

   因为小树先写到内存中，为了防止内存数据丢失，写内存的同时需要暂时持久化到磁盘，对应
   了HBase的MemStore和HLogMemStore上的树达到一定大小之后，需要flush到HRegion磁盘中
  （一般是Hadoop DataNode），这样MemStore就变成了DataNode上的磁盘文件StoreFile，定
   期HRegionServer对DataNode的数据做merge操作，彻底删除无效空间，多棵小树在这个时机
   合并成大树，来增强读性能。
</pre>