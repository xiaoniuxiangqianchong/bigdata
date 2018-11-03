# MapReduce
---
---
## 概述
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
## MapReduce 计算流程原理
### Map阶段:
* 将数据拆分成splits,每个文件是一个split,并将文件按行分隔成键值对<key,value>,然后对用户定
义的map方法进行处理,生成新的键值对,Mapper会将他们按照key值进行排序,并执行Combine过程,将key
值相同的value值累加,得到Mapper的最终输出结果.
### Reduce阶段:
* Reducer先对Mapper接收的数据进行排序,再交由用户自定义的reduce方法进行处理,得到行的键值对,
并作为WordCount的输出结果.
##### 输入数据 -> 分割结果 -> map方法输出 -> 排序 -> combine输出 -> 排序结果 -> Reduce输出
***
***
## MapReduce环境配置(注意:Windows系统下目录分隔符用\)
+ 用hadooponwindows-master中的bin和etc文件夹把hadoop-2.7.3中的相同文件夹替换掉.
+ 把hadoop-2.7.3中的hadoop.dll文件和winutils.exe文件copy到电脑Windows目录下System32文件中
+ 配环境变量:配置java和hadoop的环境变量.
+ 修改hadoop-2.7.3/etc/hadoop目录下的hadoop-env.cmd文件中的JAVA_HOME路径(目录中文件夹名称
中不能有空格).
+ 修改配置文件:修改hdfs-site文件中namenode和datanode的文件存储目录完成后再DOS命令中输入
hadoop namenode -format命令格式化namenode.然后输入start-all.cmd命令开启namenode,可在浏览器
中输入localhost:50070 查看.
***
***
## MapReduce 计算框架
#### 代码:
public class WordCountTest {

	public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
		private static final IntWritable one = new IntWritable(1);
		private Text word = new Text();

		public void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			StringTokenizer itr = new StringTokenizer(value.toString());
			while (itr.hasMoreTokens()) {
				this.word.set(itr.nextToken());
				context.write(this.word, one);
			}
		}
	}

	public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private IntWritable result = new IntWritable();

		public void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			int sum = 0;
			for (IntWritable val : values) {
				sum += val.get();
			}
			this.result.set(sum);
			context.write(key, this.result);
		}
	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(WordCount.class);
		job.setMapperClass(CountMapper.class);
		job.setCombinerClass(IntSumReducer.class);
		job.setReducerClass(IntSumReducer.class); 
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));  

		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
+++
+ 使用MapReduce计算框架时,我们只需要根据自己的需求修改Map类和Reduce类即可,其他过程MapReduce
框架会自动帮我们完成.
+ 要使用自定义的类型作为输出:
1. 根据我们的数据进行建模(创建实体类)
2. mapper形参怎么写,map()方法中怎么对数据进行切割删选将我们的数据放入自定义bean(实体类)中想
好将bean以key还是value输出.
3. map的结果最终要写入磁盘当中,所以一定要注意我们自定义bean一定要支持序列化,Serializable是
java中提供的重量级的序列化接口,在我们hadoop中建议使用hadoop自身提供的WriteableComparable接
口来实现序列化,这时候有write()写入文件和readFields()从文件中读取的方法需要我们实现,否则在
reduce阶段是获取不到数据的,而这一定要注意的一点就是readFields()方法中调用read方法的时候一定
要按照写入的顺序进行读取对我们的属性进行赋值,否则会错乱uploadStream = paramDataInput.readInt();
4. reduce当中要注意形参和输出位置,以及在driver区对应好我们reduce要输出的类型
	 job.setOutputKeyClass(Text.class);
	 job.setOutputValueClass(StreamBean.class);
5. 当这些都处理好后,reduce的结果就可以输出到我们指定的位置上了,但是要注意自定义bean的toString()
方法的重写,否则无法输出我们想要的结果