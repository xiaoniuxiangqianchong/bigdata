# MapReduce
---
---
## ����
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
## MapReduce ��������ԭ��
### Map�׶�:
* �����ݲ�ֳ�splits,ÿ���ļ���һ��split,�����ļ����зָ��ɼ�ֵ��<key,value>,Ȼ����û���
���map�������д���,�����µļ�ֵ��,Mapper�Ὣ���ǰ���keyֵ��������,��ִ��Combine����,��key
ֵ��ͬ��valueֵ�ۼ�,�õ�Mapper������������.
### Reduce�׶�:
* Reducer�ȶ�Mapper���յ����ݽ�������,�ٽ����û��Զ����reduce�������д���,�õ��еļ�ֵ��,
����ΪWordCount��������.
##### �������� -> �ָ��� -> map������� -> ���� -> combine��� -> ������ -> Reduce���
***
***
## MapReduce��������(ע��:Windowsϵͳ��Ŀ¼�ָ�����\)
+ ��hadooponwindows-master�е�bin��etc�ļ��а�hadoop-2.7.3�е���ͬ�ļ����滻��.
+ ��hadoop-2.7.3�е�hadoop.dll�ļ���winutils.exe�ļ�copy������WindowsĿ¼��System32�ļ���
+ �价������:����java��hadoop�Ļ�������.
+ �޸�hadoop-2.7.3/etc/hadoopĿ¼�µ�hadoop-env.cmd�ļ��е�JAVA_HOME·��(Ŀ¼���ļ�������
�в����пո�).
+ �޸������ļ�:�޸�hdfs-site�ļ���namenode��datanode���ļ��洢Ŀ¼��ɺ���DOS����������
hadoop namenode -format�����ʽ��namenode.Ȼ������start-all.cmd�����namenode,���������
������localhost:50070 �鿴.
***
***
## MapReduce ������
#### ����:
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
+ ʹ��MapReduce������ʱ,����ֻ��Ҫ�����Լ��������޸�Map���Reduce�༴��,��������MapReduce
��ܻ��Զ����������.
+ Ҫʹ���Զ����������Ϊ���:
1. �������ǵ����ݽ��н�ģ(����ʵ����)
2. mapper�β���ôд,map()��������ô�����ݽ����и�ɾѡ�����ǵ����ݷ����Զ���bean(ʵ����)����
�ý�bean��key����value���.
3. map�Ľ������Ҫд����̵���,����һ��Ҫע�������Զ���beanһ��Ҫ֧�����л�,Serializable��
java���ṩ�������������л��ӿ�,������hadoop�н���ʹ��hadoop�����ṩ��WriteableComparable��
����ʵ�����л�,��ʱ����write()д���ļ���readFields()���ļ��ж�ȡ�ķ�����Ҫ����ʵ��,������
reduce�׶��ǻ�ȡ�������ݵ�,����һ��Ҫע���һ�����readFields()�����е���read������ʱ��һ��
Ҫ����д���˳����ж�ȡ�����ǵ����Խ��и�ֵ,��������uploadStream = paramDataInput.readInt();
4. reduce����Ҫע���βκ����λ��,�Լ���driver����Ӧ������reduceҪ���������
	 job.setOutputKeyClass(Text.class);
	 job.setOutputValueClass(StreamBean.class);
5. ����Щ������ú�,reduce�Ľ���Ϳ������������ָ����λ������,����Ҫע���Զ���bean��toString()
��������д,�����޷����������Ҫ�Ľ��