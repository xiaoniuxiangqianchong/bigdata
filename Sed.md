# sed 行命令
> sed
## sed 选项 动作 filename
### 选项 
####	 -n 被动作处理的行才会打印出来
### 动作 (动作要用 '' 包裹起来)
####	 a 向下增加
####	 i 向上添加
####	 c 替换 1,5c 会将这一块区域整个替换,不会每行都替换
####	 d 删除
####		sed '/^$/d'		将匹配到的删除
####	 p 输出
####		sed -n '2,4' a.txt	打印出2-4行
####	 s 替换经常配合正则进行操作
####		sed 's/da/kkk/'		将da替换为kkk,且只替换每一行中第一个
####		sed 's/da/kkk/'		将da替换为kkk,且替换全部
####		结合正则使用,在()+前要添加转义字符\,1\代表取用分组1
####		sed 's/\([a-z0-9_-]\+\):x:\([0-9]\+\):\([0-9]\+\):.*/user: \1 \2 /' passwd
####	 w 写入 sed 'w 要被写入内容的文件(目标文件) ' 读取内容的文件(源文件)只要是用w项文件
####	   中进行写入就会将目标文件之前的内容全部覆盖
####	 r 读取 sed '2r 要读取的文件 ' 加入数据的文件 (目标文件)
### 高级操作
####	 | 使用管道符配合其他命令使用
####		ls | grep aa	结合grep筛选出含有aa的文件
####	 {} 可以让sed执行多个动作, 只是动作之间要用 ; 隔开
####		sed '{10,15d;/java/bigdata/}' a.txt		删除10-15行的同时把java替换为bigdata
####	 & 替换
####	 & 相当于占位符的作用 就是我们s/查询条件/查询到的内容可以在s/查询条件要替换的内容中
####	   进行使用,实现追加的效果
####	 \u 转成大写可以配合我们的&来实现将匹配内容转成大写的操作
		