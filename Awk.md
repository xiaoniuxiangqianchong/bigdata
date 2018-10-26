# awk 行命令
## awk 选项 '命令' 文件 
	NR 行号
	NF 列数
	$n 调用切割出来的第n段
	-F 设置分隔符,分隔符要用''包起来
		awk -F ':' '{print "line:"NR,"col:"NF,"user:"$1}' a.txt	普通输出方式
		awk -F ':' '{printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' a.txt 格式化输出方式
		awk -F ':' '{if($3>150) printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' awk.txt 条件输出
		awk -F ':' '$1~/^j.*/{printf("Line:%3s Col:%s User:%s\n",NR,NF,$1)}' a.txt 配合正则使用(!~正则取反)
## awk拓展格式(表格式) awk 选项 'BEGIN() 命令 END()' file (BEGIN在最开始时只执行一次)
		awk -F ':' 'BEGIN{print "Line Col User"}{printf("%3s %s %2s\n",NR,NF,$1)}' 文件
		做总计	ll输出文件详细信息,$5调用第五行
		ll | awk 'BEGIN{size=0} {size+=$5} END{print "FileSize is "size}'
		结合正则	输出文件行数
		awk -F ':' 'BEGIN{count=0} $1!~/^$/{count++} END{print "peopleCount is" count}' a.txt
		结合数组及if条件	把符合条件的name存到数组中,最后输出
		awk -F ':' 'BEGIN{count=0} {if($3>100) name[count++]}=$1 END{for (i=0;i<count;i++)print i,name[i]}' a.txt