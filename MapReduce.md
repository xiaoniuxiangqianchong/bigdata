# MapReduce
---
---
适合PB级以上数据的离线处理
mapReduce由两个阶段组成:Map阶段和Reduce阶段
Map阶段由一定数量的Map Task组成:
* 输入数据格式解析: InputFormat
* 输入数据出来: Mapper
* 数据分组: Partitioner
Reduce阶段由一定数量的Rdeuce Task组成:
* 数据远程拷贝
* 数据按照key排序
* 数据处理: Reducer
* 数据输出格式: OutputerFormat
***
***
# MapReduce 计算流程原理
## Map阶段:
将数据拆分成splits,每个文件是一个split,并将文件按行分隔成键值对<key,value>,然后对用户定
义的map方法进行处理,生成新的键值对,Mapper会将他们按照key值进行排序,并执行Combine过程,将
key值相同的value值累加,得到Mapper的最终输出结果.
## Reduce阶段:
Reducer先对Mapper接收的数据进行排序,再交由用户自定义的reduce方法进行处理,得到行的键值对,
并作为WordCount的输出结果.
### 输入数据 -> 分割结果 -> map方法输出 -> 排序 -> combine输出 -> 排序结果 -> Reduce输出
***
***
## MapReduce环境配置(注意:Windows系统下目录分隔符用\)
+ 用hadooponwindows-master中的bin和etc文件夹把hadoop-2.7.3中的相同文件夹替换掉.
+ 把hadoop-2.7.3中的hadoop.dll文件和winutils.exe文件copy到电脑Windows目录下System32文件中
+ 配环境变量:配置java和hadoop的环境变量
+ 修改hadoop-2.7.3/etc/hadoop目录下的hadoop-env.cmd文件中的JAVA_HOME路径
(目录中文件夹名称中不能有空格)
+ 修改配置文件:修改hdfs-site文件中namenode和datanode的文件存储目录
完成后再DOS命令中输入hadoop namenode -format命令格式化namenode
然后输入start-all.cmd命令开启namenode,可在浏览器中输入localhost:50070 查看