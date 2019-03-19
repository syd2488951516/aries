## 目标
+ 理解ARIES恢复算法
+ 用C++实现ARIES恢复算法

## 开始
&emsp;阅读书上恢复的章节
+ 16.7-介绍恢复管理
+ 第18章 介绍了ARIES恢复算法 超级重要去阅读和理解这一章节，因为这是项目想让你实现的内容
+ 下载Canvas上项目描述中包含的zip文件
+ 了解恢复模拟器的不同组件：LogRecond 、Storage Engine、Main、LogMgr 。这是您将为项目实现的内容

## 分级
+ 你可以在下面的网站访问autograder： https://grader484.eecs.umich.edu/
+ Autograder 测试版现在已经开放
+ 每天限制提交4次
+ Autograder使用您的LogMgr.cpp运行隐藏的测试套件并检查输出
+ 最高分将是最终成绩 如果您迟交，则罚款仅适用于延迟提交

## 磁盘  
+ 磁盘表示为简单的文本文件
+ 每行对应一页
+ 每个页面都包含一个LSN，后跟一个字符串，该字符串是存储在页面上的数据
+ 磁盘的示例位于StorageEngine / sampleDBFile.txt中

## 测试案例  
+ 测试案例中包含6个基本测试用例，以帮助检查您的项目是否正常工作
+ 通过所有6个测试案例并不意味着你将通过Autograder，创建自己的测试用例。
+ 测试用例的结果可以在output / log和output / dbs /中找到
+ 如果再次运行案例，请记住删除文件

## 测试案例的形式
+ 该文件的第一行告诉模拟器在哪里找到表示磁盘初始状态的文本文件
+ 指定要执行的操作后的所有行  
&emsp; Write - txid, action, pageid, offset, data     
&emsp; Abort – txid, action, # of writes allowed during abort  
&emsp; Checkpoint – action    
&emsp; Crash – action {# of writes allowed during crash}–end     
&emsp; End标记事务的结束      

## 运行工程
+ 在父目录上运行make 你或许想要去修改Makefile 去做更多的操作例如运行所有的测试案例
+ 在父目录上，使用命令运行模拟器：./main.o testcases/test00 这将运行test00  

## LogRecord.h
+ 包含 txTableEntry 结构 ->通过调用txTableEntry（lsn，status）构造函数创建新条目
+ 包含4个日志记录类
&emsp;  LogRecord  ->基本的日志记录类型       
&emsp; UpdateLogRecord ->对于更新的日志记录    
&emsp; CompensationLogRecord  -> 对CLR的日志记录        
&emsp; ChkptLogRecord  -> 针对检查点的日志记录  

## LogRecord.cpp
&emsp;我们在这关心两种功能:  
+ toString() 这将把任何类型的日志记录转换为字符串。
+ stringToRecordPtr 这将把与单个日志记录相对应的字符串转换为LogRecord 指针   

## Storage Engine  
+ 这可以跟踪页面并管理磁盘访问  如果执行读/写操作，则将页面添加到内存缓冲区,如果缓冲区已满，则将页面刷新到内存
+ 页面由id，LSN，脏位和数据字符串组成。
+ 可能有用的功能：
&emsp; nextLSN() 这将返回以单调递增顺序分配的唯一ID   
&emsp; pageWrite(…) 这将会写一个页面到内存：1.如果不再允许写页面，则可以返回false。这由page_write_permitted决定。 2.正常情况下，存储引擎将执行此操作，但中止或恢复时需要此操作     
&emsp; getLSN(…) 这将返回给定页ID的最后一个LSN   
&emsp; store_master(…) 这会将int写入磁盘中的某个位置   
&emsp; get_master() 这将检索通过store_master写入的int  
&emsp; getLog() 从磁盘获取日志  

## Main
+ 这会读测试案例并且初始化日志管理器和存储引擎
+ 它还解析testcase文件并调用相应的函数   

## LogMgr
+ 这是您为项目提交的文件
+ 实现LogMgr.h中指定的所有函数
+ 您必须维护事务表和脏页表。 它们作为映射存储在LogMgr.h中

## 调试代码
+ 在CAEN机器上使用GDB可能会在显示某些数据结构（如地图）时遇到问题
+ 一种可能的解决方案
&emsp; mkdir <directory_name>  
&emsp; cd <directory_name>  
&emsp; svn co svn://gcc.gnu.org/svn/gcc/branches/gcc-4_6-branch/libstdc++-v3/python  
&emsp; vim ~/.gdbinit
&emsp; 将下一张幻灯片上的代码添加到gdbinit

## 代码添加到GDBinit
+  Replace /path/to/<directory_name>/python with the actual filepath
&emsp; python  
&emsp; import sys  
&emsp; sys.path.insert(0, '/path/to/gdb_printers/python')
&emsp; from libstdcxx.v6.printers import  
&emsp; register_libstdcxx_printers
&emsp; register_libstdcxx_printers (None)
&emsp; end

## 几件需要注意的事情
+ 不要修改logmgr.cpp以外的任何文件
+ 不要直接读/写磁盘。 您必须使用StoreEngine接口来执行此操作
+ 不要在LogMrg.cpp中创建静态变量或函数
+ 尽早开始，在autograder打开时准备好一些东西  
+ 阅读教科书章节。 第18章包含了完成项目所需的大部分信息
+ 在 stringToLRVector 功能中，添加：    
&emsp;vector<LogRecord*> result;   
&emsp;istringstream stream(logstring);   
&emsp;string line;  
&emsp;while(getline(stream, line)) {  
&emsp;&emsp;&emsp; LogRecord* lr =   
&emsp;LogRecord::stringToRecordPtr(line);   
&emsp;&emsp;&emsp; Results.push_back(lr);   
&emsp;}   
&emsp;return result;   

## 例子
+ 1 write 2 0 one  
+ 2 write 3 0 two
+ 2 commit
+ 3 write 1 0 three
+ 3 write 2 0 four
+ crash {7}

## 稳定存储
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r1.png)
## T1 writes
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r2.png)
## T2 writes
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r3.png)
## T2 commits
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r4.png)
## Stable Storage – After commit
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r5.png)
## After Commit
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r6.png)
## T3 Writes
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r7.png)
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r8.png)
## Crash
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r9.png)

## 分析阶段
+ 查找最近的开始检查点。 如果没有，则从日志的开头开始
+ 查看每条记录:   
&emsp; End log:从事务表中删除T  
&emsp; Others:1.如果不在事务表中，则将T添加到事务表中; 2.改变 lastLSN 3.如果是提交，则将状态设置为C。如果它是P上的可重做记录，则将P添加到脏页表   

## Analysis
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r10.png)    
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r11.png)    
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r12.png)     
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r13.png)

## 重做阶段  
+ 在日志（最小的recLSN）中查找最旧的更新，并从日志中的该点开始
+ 对于每个可重做的记录：
&emsp;脏页表中的页面是？  
&emsp;该页面位于脏页表中，但该页面的recLSN是否小于或等于记录的LSN？  
&emsp;pageLSN是否小于日志记录的LSN？    
&emsp;如果对所有三个都是肯定的，重做记录  
+ 从表中删除已提交的事务

## Redo
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r14.png)    
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r15.png)   
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r16.png)   
## Stable Storage – After redo  
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r17.png)  

## 撤销阶段  
+ 从事务表中LSN值最大的事务开始 ，撤消事务表中的所有操作
+ 对于每个记录：   
&emsp;如果是CLR: 1.如果undoNextLSN为null，则将结束记录添加到日志中 2.否则，将undoNextLSN添加到要撤消的集合中.如果是更新：1.在日志中创建CLR记录。 如果undoNext为null，则添加结束记录。2.添加prevLSN以设置为撤消

## Undo
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r18.png)   
## Stable Storage – After undo  
&emsp;&emsp;&emsp;&emsp;&emsp;![](/images/r19.png)   
