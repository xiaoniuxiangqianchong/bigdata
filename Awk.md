# awk ������
## awk ѡ�� '����' �ļ� 
	NR �к�
	NF ����
	$n �����и�����ĵ�n��
	-F ���÷ָ���,�ָ���Ҫ��''������
		awk -F ':' '{print "line:"NR,"col:"NF,"user:"$1}' a.txt	��ͨ�����ʽ
		awk -F ':' '{printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' a.txt ��ʽ�������ʽ
		awk -F ':' '{if($3>150) printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' awk.txt �������
		awk -F ':' '$1~/^j.*/{printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' a.txt �������ʹ��(!~����ȡ��)
## awk��չ��ʽ(���ʽ) awk ѡ�� 'BEGIN() ���� END()' file (BEGIN���ʼʱִֻ��һ��)
		awk -F ':' 'BEGIN{print "Line Col User"}{printf("%3s %s %2s\n",NR,NF,$1)}' �ļ�
		���ܼ�	ll����ļ���ϸ��Ϣ,$5���õ�����
		ll | awk 'BEGIN{size=0} {size+=$5} END{print "FileSize is "size}'
		�������	����ļ�����
		awk -F ':' 'BEGIN{count=0} $1!~/^$/{count++} END{print "peopleCount is" count}' a.txt
		������鼰if����	�ѷ���������name�浽������,������
		awk -F ':' 'BEGIN{count=0} {if($3>100) name[count++]}=$1 END{for (i=0;i<count;i++)print i,name[i]}' a.txt