# MapReduce
---
---
�ʺ�PB���������ݵ����ߴ���
mapReduce�������׶����:Map�׶κ�Reduce�׶�
Map�׶���һ��������Map Task���:
* �������ݸ�ʽ����: InputFormat
* �������ݳ���: Mapper
* ���ݷ���: Partitioner
Reduce�׶���һ��������Rdeuce Task���:
* ����Զ�̿���
* ���ݰ���key����
* ���ݴ���: Reducer
* ���������ʽ: OutputerFormat
***
***
# MapReduce ��������ԭ��
## Map�׶�:
�����ݲ�ֳ�splits,ÿ���ļ���һ��split,�����ļ����зָ��ɼ�ֵ��<key,value>,Ȼ����û���
���map�������д���,�����µļ�ֵ��,Mapper�Ὣ���ǰ���keyֵ��������,��ִ��Combine����,��
keyֵ��ͬ��valueֵ�ۼ�,�õ�Mapper������������.
## Reduce�׶�:
Reducer�ȶ�Mapper���յ����ݽ�������,�ٽ����û��Զ����reduce�������д���,�õ��еļ�ֵ��,
����ΪWordCount��������.
### �������� -> �ָ��� -> map������� -> ���� -> combine��� -> ������ -> Reduce���
***
***
## MapReduce��������(ע��:Windowsϵͳ��Ŀ¼�ָ�����\)
+ ��hadooponwindows-master�е�bin��etc�ļ��а�hadoop-2.7.3�е���ͬ�ļ����滻��.
+ ��hadoop-2.7.3�е�hadoop.dll�ļ���winutils.exe�ļ�copy������WindowsĿ¼��System32�ļ���
+ �价������:����java��hadoop�Ļ�������
+ �޸�hadoop-2.7.3/etc/hadoopĿ¼�µ�hadoop-env.cmd�ļ��е�JAVA_HOME·��
(Ŀ¼���ļ��������в����пո�)
+ �޸������ļ�:�޸�hdfs-site�ļ���namenode��datanode���ļ��洢Ŀ¼
��ɺ���DOS����������hadoop namenode -format�����ʽ��namenode
Ȼ������start-all.cmd�����namenode,���������������localhost:50070 �鿴