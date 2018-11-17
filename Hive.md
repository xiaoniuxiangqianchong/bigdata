# Hive
---
---
## Hive环境搭建
1. 上传hive压缩包到HDFS上并解压缩
2. 在/etc/profiles中配置hive环境变量:
    * HIVE_HOME=hive路径
    * PATH后追加:$HIVE_HOME/bin
3. 执行 /etc/profile 后进行hive启动测试
4. 安装mysql,使用root登录mysql并给root用户修改密码
    * mysql状态/开启/关闭指令 systemctl status/start/stop mysql
    * mysql修改密码:(根据不同数据库版本选择使用)
		* 命令1: UPDATE mysql.user SET authentication_string=PASSWORD('6757DUgu') where USER='root';
		* 命令2: UPDATE mysql.user SET Password=PASSWORD('新密码') where USER='root';
		* flush privileges;
5. 开启mysql远程访问,并将mysql驱动mysql-connector-java放入hive/lib目录下
    * 开启远程访问指令:mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '密码' WITH GRANT OPTION;
					   mysql>FLUSH PRIVILEGES;
6. 在hive/conf目录下创建hive-site.xml配置文件,数据如下
(根据情况修改IP和mysql的密码)
	<configuration>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://192.168.87.131:3306/hivedb?createDatabaseIfNotExist=true</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>password</value>
        </property>
	</configuration>
7. 启动hive

***

## Hive使用:
* 输入hiveserver2 -> 回车 开启hive允许访问服务
* 输入beeline -> 回车 然后输入连接指令 !connect jdbc:hive2://localhost:10000
* 输入用户名为要链接HDFS的用户名
* 输入密码,这里直接回车

***

## DDL(数据库定义语言,对表的操作)[create/alter/...]
	* 创建内[外]部表:
		create [external] table 表名(字段 数据类型, ...)row format delimited fields terminated by '分隔符';
	* 创建分区表:
		create table 表名(字段 数据类型, ...) partitioned by (分区 数据类型,子分区 s数据类型) 
		row format delimited fields terminated by '\t';
	* 创建分桶表:into后面为分桶个数
		create table goodsbucket(goodsid int, name string)
		clustered by(goodsid) into 3 buckets
		row format delimited fields terminated by '\t';
		允许分桶:(set hive.enforce.bucketing=true;)
	* 因为外部表多用于数据引用,不希望对数据进行破坏,所以要使用location来指定引用数据源,
	在最后添加 location '数据源地址';
	* 命令load data inpath 会将数据文件移动至表目录下
	* 命令:
		* 修改表名	alter table 原表名 rename to 新表名;
		* 添加字段	alter table userlck add columns (sex string);
		* 重置字段	alter table userlck replace columns(name string);(只剩下name一列)
		* 修改字段	alter table userlck change sex gender string;(sex换为gender)
		* 添加分区	alter table lcktime add partition(year=2019,month=10);
		* 删除分区	alter table lcktime drop partition(year='2018',month='10');

## DML(数据库操作语言,对数据的操作)[insert/load]
	* 插入表数据	insert into 表名 values (1,'lck','m');
	* 查询到的结果插入表1	insert into 表1 select * from 表2;
	(导入数据时在into前面加上overwrite则会覆盖导入目录下之前的内容)
	* 向数据库导入本地文件:	load data local inpath '本地路径' into table 表名;
	* 向数据库导入HDFS文件:	load data inpath 'HDFS路径' into table 表名;
	* 导入分区: 导入文件指令后面加上 partition(分区名='分区号',子分区名='子分区号');

	* 动态分区:	(select后最后一个字段作为分区条件,将结果插入到userslck表中)
	insert into 表名 partition(gender) select id,name,gender from persons;

	* 使用场景
		当我们想要对数据进行分区的时候,你能拿到的数据却未必是已经分好区的文件,这时就可以
		使用动态分区来解决这种问题
	* 步骤:
		* 将混乱数据传入一个表
		* 创建分区表,执行动态分区
	* 注意:
	* 使用动态分区时 partition(gender) 不要给分区设置固定值,括中只给分区字段默认使用的是严
	格模式,是不允许动态分区的,需要执行命令开启动态分区:
	set hive.exec.dynamic.partition.mode=nonstrict

***

## 数据查询
### cats: 将查询结果存在新表中称之为cats
	* 用查询到结果创建表(as别名作为新表的字段名,如果不写就用查询到的字段名)		
	create table 表1 as select name (as username) from 表2;	
### 聚合函数:(avg/sum/count/...)
##### avg
	* select dep,avg(money) am from company_emp group by dep having am > 1000; 
	1. 使用分组就不再是逐行执行模式了,会根据我们的分组的key以分组模式执行,所以我们的数据都
	是一组一组的,没法调用分组key和聚合函数之外的单独属性
	2. 使用聚合函数得到的结果默认的字段名不好使,我们要进行调用可以给他取个别名,就可以在条件中使
	用了,同一层给having使用,作为查询时可以给外层的where用
	3. where执行在聚合之前进行数据删选,而having是在聚合结束后继续条件处理
### 关联查询
##### 全连接
	* select o.*,g.* from orders o full join goods g on o.goodsid = g.goodsid
##### 左半连接:(结果中值显示左连接中的左表内容)
	* select o.* from orders o left semi join goods g on o.goodsid = g.goodsid
### 数据类型
##### array数组的使用:
	* create [external] table 表名(字段名 array<string>)
	row format delimited fields terminated by '分隔符'
	collection items terminated by '分隔符'; (分割出数组)
##### map的使用
	* create [external] table 表名(字段名 map<string,string>)
	row format delimited fields terminated by '分隔符'
	collection items terminated by '分隔符' (分割出数组)
	map keys terminated by '分隔符'; (分割出map)
##### struct结构体的使用
	* create table structs
	(id int,name string,info struct<age:int,gender:string,addr:string>)
	row format delimited fields terminated by ',' 
	collection items terminated by ':';
	取用:info.age
### 简单函数
	* 类型转换:		select cast (a as b)	a转换为b
	* 向上取整:		select ceil(参数)
	* 向下取整:		select floor(参数)
	* 取最大值:		select greatest(a,b,c,...)
	* 取最小值:		select least(...)
	* 四舍五入:		
	* 截取字符串:	select substring('字符串',起始(int),结束(int))
	* 字符串拼接:	* concat('aa', 'bb')
				* concat_ws('拼接符','a','b')
	* 字符串切割:	split('字符串','切割符')	注意转义字符,同java
### 时间函数:(时间戳:时间元年到当前秒数)
	* 获取当前时间:		select current_timestamp();
	* 获取当前时间戳:
		* 无参: select unix_timestamp();
		* 传参用法:	select unix_timestamp('2018-11-16 11:44:22');
	* 按规定格式处理:select unix_timestamp('2018 11 16','yyyy MM dd');
	* 将时间戳转为时间:	select from_unixtime(unix_timestamp());
	* 格式化:	select from_unixtime(unix_timestamp(),'yyyy/MM/dd');
### 条件函数:
	* array_contains(数组,'sister')	判断数组中是否含有字符串,返回boolean值
	* map_keys(map集合)		取出map中key值,返回一个数组
	* map_values(map集合)		取出map中value值,返回一个数组
	* if(条件,成立返回值,不成立返回值)
	* 按条件分组,并在分组内统计符合条件的记录条数
	select gender,count( if(status = 'jiuye',1,null) ) from cou group by gender
	* 按条件分组,将某个字段对应的值放在一个数组里
	select id,name,collect_set(sub) from subject group by id,name;
