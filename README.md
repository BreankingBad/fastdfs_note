# fastdfs_note
fastdfs 分布式文件系统部署笔记

1、按照https://zhuanlan.zhihu.com/p/35728079 ,对两台云服务器进行部署，一台当tracker和storage,另一台只当storage

2、因为我这里是两台服务器分别存放不同文件，所以将一台storage配置为group_name=group1，另一台storage配置为group_name=group2

3、注意，相关配置修改后，需重启相应服务(需要等几秒钟查看服务是否正常)，不行的话重启所有tracker/storage, 命令为/etc/init.d/fdfs_storaged start|restart
  和 /etc/init.d/fdfs_trackerd start|restart
  
4、注意tracker的配置reserved_storage_space为实际情况的大小，避免某台服务器空间不够没办法后续测试

\# 用于设置系统或其他应用程序的存储空间。<br>
\# 当剩余的空间小于reserved_storage_space设置的空间<br>
\# 该group组就无法再上传其它文件<br>
reserved_storage_space = 10%

5、因为tracker默认选择最大空闲空间的storage进行上传，而我想测试两台storage服务都可用，所以设置tracker配置store_lookup=1，store_group=想测试
的storage对应的group名

\# 上传文件时卷组的选择方式<br>
\# 0: 轮循<br>
\# 1: 指定<br>
\# 2: 负载均衡, 会选择最大空闲的卷组去上传文件<br>
store_lookup=2<br>
 
\# 如果上面的设置为 1, 就必须设置一个group组<br>
store_group=group2<br>

6、上传测试：    /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/include/stdio.h

7、如果运行/上传文件出错，务必看log信息，cat /fastdfs/storage/logs/storaged.log 或 cat /fastdfs/storage/logs/trackerd.log

8、查看storage的状态是否为active:    
fdfs_monitor /etc/fdfs/client.conf

之前遇到一个服务器一直是wait_async状态，导致不能正常上传文件，解决办法：

\# 从集群中删除storage<br>
fdfs_monitor /etc/fdfs/client.conf delete group1 10.1.8.101<br>
 
\# 删除数据文件夹<br>
rm -rf /fastdfs/storage/data<br>
 
\# 重启storage<br>
fdfs_storaged /etc/fdfs/storage.conf<br>
 
\# 重新查状态(tracker)<br>
fdfs_monitor /etc/fdfs/client.conf<br>

9、查看已经上传的文件列表：

不能直接查看，可以看/fastdfs/storage/data/sync/目录下的binlog文件，里面有文件记录，最下面的是最近上传的文件

