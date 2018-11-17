# Hive
---
---
## Hive�����
1. �ϴ�hiveѹ������HDFS�ϲ���ѹ��
2. ��/etc/profiles������hive��������:
    * HIVE_HOME=hive·��
    * PATH��׷��:$HIVE_HOME/bin
3. ִ�� /etc/profile �����hive��������
4. ��װmysql,ʹ��root��¼mysql����root�û��޸�����
    * mysql״̬/����/�ر�ָ�� systemctl status/start/stop mysql
    * mysql�޸�����:(���ݲ�ͬ���ݿ�汾ѡ��ʹ��)
		* ����1: UPDATE mysql.user SET authentication_string=PASSWORD('6757DUgu') where USER='root';
		* ����2: UPDATE mysql.user SET Password=PASSWORD('������') where USER='root';
		* flush privileges;
5. ����mysqlԶ�̷���,����mysql����mysql-connector-java����hive/libĿ¼��
    * ����Զ�̷���ָ��:mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '����' WITH GRANT OPTION;
					   mysql>FLUSH PRIVILEGES;
6. ��hive/confĿ¼�´���hive-site.xml�����ļ�,��������
(��������޸�IP��mysql������)
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
7. ����hive

***

## Hiveʹ��:
* ����hiveserver2 -> �س� ����hive������ʷ���
* ����beeline -> �س� Ȼ����������ָ�� !connect jdbc:hive2://localhost:10000
* �����û���ΪҪ����HDFS���û���
* ��������,����ֱ�ӻس�

***

## DDL(���ݿⶨ������,�Ա�Ĳ���)[create/alter/...]
	* ������[��]����:
		create [external] table ����(�ֶ� ��������, ...)row format delimited fields terminated by '�ָ���';
	* ����������:
		create table ����(�ֶ� ��������, ...) partitioned by (���� ��������,�ӷ��� s��������) 
		row format delimited fields terminated by '\t';
	* ������Ͱ��:into����Ϊ��Ͱ����
		create table goodsbucket(goodsid int, name string)
		clustered by(goodsid) into 3 buckets
		row format delimited fields terminated by '\t';
		�����Ͱ:(set hive.enforce.bucketing=true;)
	* ��Ϊ�ⲿ���������������,��ϣ�������ݽ����ƻ�,����Ҫʹ��location��ָ����������Դ,
	�������� location '����Դ��ַ';
	* ����load data inpath �Ὣ�����ļ��ƶ�����Ŀ¼��
	* ����:
		* �޸ı���	alter table ԭ���� rename to �±���;
		* ����ֶ�	alter table userlck add columns (sex string);
		* �����ֶ�	alter table userlck replace columns(name string);(ֻʣ��nameһ��)
		* �޸��ֶ�	alter table userlck change sex gender string;(sex��Ϊgender)
		* ��ӷ���	alter table lcktime add partition(year=2019,month=10);
		* ɾ������	alter table lcktime drop partition(year='2018',month='10');

## DML(���ݿ��������,�����ݵĲ���)[insert/load]
	* ���������	insert into ���� values (1,'lck','m');
	* ��ѯ���Ľ�������1	insert into ��1 select * from ��2;
	(��������ʱ��intoǰ�����overwrite��Ḳ�ǵ���Ŀ¼��֮ǰ������)
	* �����ݿ⵼�뱾���ļ�:	load data local inpath '����·��' into table ����;
	* �����ݿ⵼��HDFS�ļ�:	load data inpath 'HDFS·��' into table ����;
	* �������: �����ļ�ָ�������� partition(������='������',�ӷ�����='�ӷ�����');

	* ��̬����:	(select�����һ���ֶ���Ϊ��������,��������뵽userslck����)
	insert into ���� partition(gender) select id,name,gender from persons;

	* ʹ�ó���
		��������Ҫ�����ݽ��з�����ʱ��,�����õ�������ȴδ�����Ѿ��ֺ������ļ�,��ʱ�Ϳ���
		ʹ�ö�̬�����������������
	* ����:
		* ���������ݴ���һ����
		* ����������,ִ�ж�̬����
	* ע��:
	* ʹ�ö�̬����ʱ partition(gender) ��Ҫ���������ù̶�ֵ,����ֻ�������ֶ�Ĭ��ʹ�õ�����
	��ģʽ,�ǲ�����̬������,��Ҫִ���������̬����:
	set hive.exec.dynamic.partition.mode=nonstrict

***

## ���ݲ�ѯ
### cats: ����ѯ��������±��г�֮Ϊcats
	* �ò�ѯ�����������(as������Ϊ�±���ֶ���,�����д���ò�ѯ�����ֶ���)		
	create table ��1 as select name (as username) from ��2;	
### �ۺϺ���:(avg/sum/count/...)
##### avg
	* select dep,avg(money) am from company_emp group by dep having am > 1000; 
	1. ʹ�÷���Ͳ���������ִ��ģʽ��,��������ǵķ����key�Է���ģʽִ��,�������ǵ����ݶ�
	��һ��һ���,û�����÷���key�;ۺϺ���֮��ĵ�������
	2. ʹ�þۺϺ����õ��Ľ��Ĭ�ϵ��ֶ�������ʹ,����Ҫ���е��ÿ��Ը���ȡ������,�Ϳ�����������ʹ
	����,ͬһ���havingʹ��,��Ϊ��ѯʱ���Ը�����where��
	3. whereִ���ھۺ�֮ǰ��������ɾѡ,��having���ھۺϽ����������������
### ������ѯ
##### ȫ����
	* select o.*,g.* from orders o full join goods g on o.goodsid = g.goodsid
##### �������:(�����ֵ��ʾ�������е��������)
	* select o.* from orders o left semi join goods g on o.goodsid = g.goodsid
### ��������
##### array�����ʹ��:
	* create [external] table ����(�ֶ��� array<string>)
	row format delimited fields terminated by '�ָ���'
	collection items terminated by '�ָ���'; (�ָ������)
##### map��ʹ��
	* create [external] table ����(�ֶ��� map<string,string>)
	row format delimited fields terminated by '�ָ���'
	collection items terminated by '�ָ���' (�ָ������)
	map keys terminated by '�ָ���'; (�ָ��map)
##### struct�ṹ���ʹ��
	* create table structs
	(id int,name string,info struct<age:int,gender:string,addr:string>)
	row format delimited fields terminated by ',' 
	collection items terminated by ':';
	ȡ��:info.age
### �򵥺���
	* ����ת��:		select cast (a as b)	aת��Ϊb
	* ����ȡ��:		select ceil(����)
	* ����ȡ��:		select floor(����)
	* ȡ���ֵ:		select greatest(a,b,c,...)
	* ȡ��Сֵ:		select least(...)
	* ��������:		
	* ��ȡ�ַ���:	select substring('�ַ���',��ʼ(int),����(int))
	* �ַ���ƴ��:	* concat('aa', 'bb')
				* concat_ws('ƴ�ӷ�','a','b')
	* �ַ����и�:	split('�ַ���','�и��')	ע��ת���ַ�,ͬjava
### ʱ�亯��:(ʱ���:ʱ��Ԫ�굽��ǰ����)
	* ��ȡ��ǰʱ��:		select current_timestamp();
	* ��ȡ��ǰʱ���:
		* �޲�: select unix_timestamp();
		* �����÷�:	select unix_timestamp('2018-11-16 11:44:22');
	* ���涨��ʽ����:select unix_timestamp('2018 11 16','yyyy MM dd');
	* ��ʱ���תΪʱ��:	select from_unixtime(unix_timestamp());
	* ��ʽ��:	select from_unixtime(unix_timestamp(),'yyyy/MM/dd');
### ��������:
	* array_contains(����,'sister')	�ж��������Ƿ����ַ���,����booleanֵ
	* map_keys(map����)		ȡ��map��keyֵ,����һ������
	* map_values(map����)		ȡ��map��valueֵ,����һ������
	* if(����,��������ֵ,����������ֵ)
	* ����������,���ڷ�����ͳ�Ʒ��������ļ�¼����
	select gender,count( if(status = 'jiuye',1,null) ) from cou group by gender
	* ����������,��ĳ���ֶζ�Ӧ��ֵ����һ��������
	select id,name,collect_set(sub) from subject group by id,name;
