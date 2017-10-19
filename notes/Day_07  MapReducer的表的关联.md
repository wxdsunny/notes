---
title: Day_07  MapReducer的表的关联
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## 数据倾斜:

> 数据里面的某个key值的数据远远大于其他key的数据量



> Combiner的执行位置在map节点,把数据写在map本地,和Reducer不一样

## mr串联
### mr串联
> - 在MapReduce里面可以有多个map,一个reducer
> - 多个map执行不同的数据处理过程,能够提高代码的复用性
> - 把多个map串联,前一个map的输出作为下一个map的输入
> - mr串联 map+ reducer map\* 
> - map+ 在map端,reducer和map\* 在Reducer端

![][1]

> chainmapper对应map端 
> chainreducer对应的reduce端

### mr串联代码

``` java
package com.zhiyou100.mapreduce;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.chain.ChainMapper;
import org.apache.hadoop.mapreduce.lib.chain.ChainReducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MRChain {
	//map过滤掉销售数量大于1亿的数据
	public static class MRChainMap1 extends Mapper<LongWritable, Text, Text, IntWritable> {
		private String [] infos;
		private IntWritable oValue = new IntWritable();
		private Text oKey = new Text();
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			if(Integer.valueOf(infos[1])<100000000){
				oKey.set(infos[0]);
				oValue.set(Integer.valueOf(infos[1]));
				context.write(oKey, oValue);
			}
		}
		
	}
	//过滤掉在100-10000之间的数据
	public static class MRChainMap2 extends Mapper<Text, IntWritable, Text, IntWritable> {
		@Override
		protected void map(Text key, IntWritable value, Mapper<Text, IntWritable, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			if(value.get() <= 100||value.get() >= 10000){
				context.write(key, value);
			}
		}
	}
	//聚合商品总数量
	public static class MRChainReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private int sum;
		private IntWritable oValue = new IntWritable();
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			oValue.set(sum);
			context.write(key, oValue);
		}
	}
	//商品长度大于三的过滤
	public static class MRChainMap3 extends Mapper<Text, IntWritable, Text, IntWritable> {
		@Override
		protected void map(Text key, IntWritable value, Mapper<Text, IntWritable, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			if(key.toString().length() < 3){
				context.write(key, value);
			}
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(MRChain.class);
		job.setJobName("串型化");
		//设置mapper端
		ChainMapper.addMapper(job, MRChainMap1.class, LongWritable.class,
				Text.class, Text.class, IntWritable.class, conf);
		ChainMapper.addMapper(job, MRChainMap2.class, Text.class,
				IntWritable.class, Text.class, IntWritable.class, conf);
		//设置reduce端
		ChainReducer.setReducer(job, MRChainReducer.class, Text.class,
				IntWritable.class, Text.class, IntWritable.class, conf);
		ChainReducer.addMapper(job, MRChainMap3.class, Text.class,
				IntWritable.class, Text.class, IntWritable.class, conf);
		Path inPath = new Path("/test.txt");
		Path outPath = new Path("/bd14/userinfo/chain");
		FileInputFormat.addInputPath(job, inPath);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```
## MapReducer的表的关联
### 三种关联
把两种类型的数据关联起来:
- join的连接方式
	- Map端关联
		- 效率最高
		- 一个大表和一个小表进行关联,采用
			- 在map端加载数据时,先把小文件加载到分布式缓存文件中(要点)
			- 再把小表中的kv放在内存中
			- 扫描大表,解析出kv
			- 根据kv在内存中找到相同的key,进行抓取
		- 小表:单个节点可以处理,内存完全可以容纳

![][2]

- 
	- Reducer端关联
		- 效率最低
		- 两张表都是大表的时候采用
			- 给每个读取到的kv打上标签,可以是文件名也可以是表名
				- key:两张表的关联条件
			- 根据标签把相同的所有的kv分成两个部分,这两个部分进行一个笛卡尔乘积,得到的结果就是关联后的结果

![][3]

- 
   - SemiJoin半连接
		- 比Reducer端关联高些
		- 对Reducer关联的优化
		- SemiJoin半连接的关联
			- 抽取其中一个文件中的关联字段,放入cachefile中
			- map缓存文件中加载,在解析数据时把另一张表中的key和缓存文件中数据关联不上的抛除掉
			- 给两张表kv打标识
			- Reducer根据kv标识把数据分为两组,再进行笛卡尔乘积
			- 这样过滤了两个key值不同的数据,减少了reducer的工作量

![][4]

### 三种关联代码
#### map关联代码:

``` java
package com.zhiyou100.bd14;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;
import java.util.Map;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

//计算每个省份的用户对系统的访问次数
public class MapJoin {
	//Map读取分布式缓存文件,把它加载到一个hashmap中关联字段作为key,计算相关字段作为value
	//map方法中处理大表数据,每处理一条就取出相关字段,看hashmap中是否存在,存在代表可以关联,不存在代表不能关联
	public static class MapJoinMap extends Mapper<LongWritable, Text, Text, IntWritable> {
		private String[] infos;
		private Text outKey = new Text();
		private IntWritable outValue = new IntWritable(1);
		private Map<String, String> userInfos = new HashMap<>();
		@Override
		protected void setup(Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			//获取分布式缓存文件的路径
			//把小表的内容放到map端的
			URI[] cacheFiles = context.getCacheFiles();
			FileSystem fileSystem = FileSystem.get(context.getConfiguration());
			for (URI uri : cacheFiles) {
				if(uri.toString().contains("user_info.txt")){
					FSDataInputStream inputStream = fileSystem.open(new Path(uri));
					InputStreamReader inputStreamReader = new InputStreamReader(inputStream,"UTF-8");
					BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
					String readLine = bufferedReader.readLine();
					while(readLine != null){
						infos = readLine.split("\\s");
						userInfos.put(infos[0], infos[2]);
						readLine = bufferedReader.readLine();
					}
				}
			}
		}

		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			if(userInfos.containsKey(infos[0])){
				outKey.set(userInfos.get(infos[0]));
				context.write(outKey, outValue);
			}
		}
	}
	public static class MapJoinReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private int sum;
		private IntWritable oValue = new IntWritable();
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			oValue.set(sum);
			context.write(key, oValue);
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(MapJoin.class);
		job.setJobName("map表关联");
		job.setMapperClass(MapJoinMap.class);
		job.setReducerClass(MapJoinReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		//设置分布式缓存文件(小表)
		Path cachePath = new Path("/user_info.txt");
		job.addCacheFile(cachePath.toUri());
		//大表
		Path inPath = new Path("/user-logs-large.txt");
		Path outPath = new Path("/bd14/user/userinfo/mapJoin");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileInputFormat.addInputPath(job, inPath);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```
#### Reducer关联代码

``` java
package com.zhiyou100.bd14;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

//计算每个省份的用户对系统的访问次数
public class ReducerJoin {
	//保存value同时打标签
	public static class ValueWritable implements Writable{
		private String value;
		private String flag;
		public String getValue() {
			return value;
		}
		public void setValue(String value) {
			this.value = value;
		}
		public String getFlag() {
			return flag;
		}
		public void setFlag(String flag) {
			this.flag = flag;
		}
		@Override
		public void write(DataOutput out) throws IOException {
			out.writeUTF(value);
			out.writeUTF(flag);
		}
		@Override
		public void readFields(DataInput in) throws IOException {
			this.value = in.readUTF();
			this.flag = in.readUTF();
		}
	}
	//读取两个文件,根据来源把每一个kv对打上标签输出给reducer,key必须是关联字段
	public static class ReducerJoinMap extends Mapper<LongWritable, Text, Text, ValueWritable> {
		private String[] infos;
		private String fileName;
		private FileSplit inputSplit;
		private Text outKey = new Text();
		private ValueWritable outValue = new ValueWritable();
		@Override
		protected void setup(Mapper<LongWritable, Text, Text, ValueWritable>.Context context)
				throws IOException, InterruptedException {
			inputSplit = (FileSplit)context.getInputSplit();
			if(inputSplit.getPath().toString().contains("user-logs-large.txt")){
				fileName = "userLogsLarge";
			}else if(inputSplit.getPath().toString().contains("user_info.txt")){
				fileName = "userInfo";
			}
		}
		@Override
		protected void map(LongWritable key, Text value,
				Mapper<LongWritable, Text, Text, ValueWritable>.Context context)
				throws IOException, InterruptedException {
			outValue.setFlag(fileName);
			infos = value.toString().split("\\s");
			if(fileName.equals("userLogsLarge")){
				//解析user-logs-large.txt的过程(用户名,行为类型,ip地址)
				outKey.set(infos[0]);
				outValue.setValue(infos[1]+"\t"+infos[2]);
			}else if(fileName.equals("userInfo")){
				//解析user-info.txt的过程(用户名,性别,省份)
				outKey.set(infos[0]);
				outValue.setValue(infos[1]+"\t"+infos[2]);
			}
			context.write(outKey, outValue);
		}
	}
	//接收map发来的kv,根据value中的flag来把同一个key对应的value分成两组
	//那么两组中的数据就是分别来自两个表中的数据,对这两组数据笛卡尔乘积即完成关联
	public static class ReducerJoinReducer extends Reducer<Text, ValueWritable, Text, Text> {
		private List<String> userLogsLargeList;
		private List<String> userInfoList;
		private Text oValue = new Text();
		@Override
		protected void reduce(Text key, Iterable<ValueWritable> values,
				Reducer<Text, ValueWritable, Text, Text>.Context context) throws IOException, InterruptedException {
			userLogsLargeList = new ArrayList<>();
			userInfoList = new ArrayList<>();
			for (ValueWritable value : values) {
				if(value.getFlag().equals("userLogsLarge")){
					userLogsLargeList.add(value.getValue());
				}else if(value.getFlag().equals("userInfo")){
					userInfoList.add(value.getValue());
				}
			}
			//对两组数据中的数据进行笛卡尔乘积
			for(String userLogsLarge : userLogsLargeList){
				for(String userInfo : userInfoList){
					oValue.set(userLogsLarge+"\t"+userInfo);
					context.write(key, oValue);
				}
			}
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(ReducerJoin.class);
		job.setJobName("reducer的表关联");
		job.setMapperClass(ReducerJoinMap.class);
		job.setReducerClass(ReducerJoinReducer.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(ValueWritable.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		FileInputFormat.addInputPath(job, new Path("/user_info.txt"));
		FileInputFormat.addInputPath(job, new Path("/user-logs-large.txt"));
		Path outPath = new Path("/bd14/user/userinfo/reducerJoin");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```
#### 半连接代码

生成缓存文件

``` java
package com.zhiyou100.bd14;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class SemiJoin {
	public static class SemiJoinMap extends Mapper<LongWritable, Text, Text, NullWritable> {
		private String[] infos;
		private Text oKey = new Text();
		private NullWritable oValue = NullWritable.get();
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			oKey.set(infos[0]);
			context.write(oKey, oValue);
		}
	}
	public static class SemiJoinReducer extends Reducer<Text, NullWritable, Text, NullWritable> {
		private NullWritable oValue = NullWritable.get();
		@Override
		protected void reduce(Text key, Iterable<NullWritable> values,
				Reducer<Text, NullWritable, Text, NullWritable>.Context context) throws IOException, InterruptedException {
			context.write(key, oValue);
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(SemiJoin.class);
		job.setJobName("半关联第一个文件");
		job.setMapperClass(SemiJoinMap.class);
		job.setReducerClass(SemiJoinReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);
		FileInputFormat.addInputPath(job, new Path("/user-logs-large.txt"));
		Path outPath = new Path("/bd14/user/userinfo/semijoin1");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```

排除与缓存文件中不一致的,再得出一致的笛卡尔集

``` java
package com.zhiyou100.bd14;

import java.io.BufferedReader;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class SemiJoinTwo {
	public static class ValueSemin implements Writable {
		private String value;
		private String flag;
		public String getValue() {
			return value;
		}

		public void setValue(String value) {
			this.value = value;
		}

		public String getFlag() {
			return flag;
		}

		public void setFlag(String flag) {
			this.flag = flag;
		}

		@Override
		public void write(DataOutput out) throws IOException {
			out.writeUTF(value);
			out.writeUTF(flag);
		}

		@Override
		public void readFields(DataInput in) throws IOException {
			this.value = in.readUTF();
			this.flag = in.readUTF();
		}
	}
	public static class SemiJoinTwoMap extends Mapper<LongWritable, Text, Text, ValueSemin>{
		private String[] infos;
		private Text oKey = new Text();
		private ValueSemin oValue = new ValueSemin();
		private String fileName;
		private List<String> keyList = new ArrayList<>();
		private FileSplit inputSplit;
		@Override
		protected void setup(Mapper<LongWritable, Text, Text, ValueSemin>.Context context)
				throws IOException, InterruptedException {
			URI[] cacheFiles = context.getCacheFiles();
			FileSystem fileSystem = FileSystem.get(context.getConfiguration());
			for (URI uri : cacheFiles) {
				if(uri.getPath().toString().contains("part-r-00000")){
					FSDataInputStream fsDataInputStream = fileSystem.open(new Path(uri));
					InputStreamReader inputStreamReader = new InputStreamReader(fsDataInputStream);
					BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
					String readLine = bufferedReader.readLine();
					while(readLine != null){
						keyList.add(readLine);
						readLine = bufferedReader.readLine();
					}
				}
			}
			inputSplit = (FileSplit)context.getInputSplit();
			if(inputSplit.getPath().toString().contains("user-logs-large.txt")){
				fileName = "userLogsLarge";
			}else if(inputSplit.getPath().toString().contains("user_info.txt")){
				fileName = "userInfo";
			}
		}
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, ValueSemin>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			if(keyList.contains(infos[0])){
				if(fileName.equals("userLogsLarge")){
					oKey.set(infos[0]);
					oValue.setFlag(fileName);
					oValue.setValue(infos[1]+"\t"+infos[2]);
				}else if(fileName.equals("userInfo")){
					oKey.set(infos[0]);
					oValue.setFlag(fileName);
					oValue.setValue(infos[1]+"\t"+infos[2]);
				}
				context.write(oKey, oValue);
			}
		}
		
	}
	public static class SemiJoinTwoReducer extends Reducer<Text, ValueSemin, Text, Text>  {
		private List<String> userLogsLargeList;
		private List<String> userInfoList;
		private Text oValue = new Text();
		@Override
		protected void reduce(Text key, Iterable<ValueSemin> values, Reducer<Text, ValueSemin, Text, Text>.Context context)
				throws IOException, InterruptedException {
			userLogsLargeList = new ArrayList<>();
			userInfoList = new ArrayList<>();
			for (ValueSemin value : values) {
				if(value.getFlag().contains("userLogsLarge")){
					userLogsLargeList.add(value.getValue());
				}else if(value.getFlag().contains("userInfo")){
					userInfoList.add(value.getValue());
				}
			}
			for(String LogsLarge : userLogsLargeList){
				for(String Info : userInfoList){
					oValue.set(LogsLarge+"\t"+Info);
					context.write(key, oValue);
				}
			}
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(SemiJoinTwoMap.class);
		job.setJobName("半关联");
		job.setMapperClass(SemiJoinTwoMap.class);
		job.setReducerClass(SemiJoinTwoReducer.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(ValueSemin.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		job.addCacheFile((new Path("/bd14/user/userinfo/semijoin1/part-r-00000")).toUri());
		FileInputFormat.addInputPath(job, new Path("/user_info.txt"));
		FileInputFormat.addInputPath(job, new Path("/user-logs-large.txt"));
		Path outPath = new Path("/bd14/user/userinfo/semijoin2");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```






  [1]: https://www.github.com/wxdsunny/images/raw/master/1508210695843.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1508421075477.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1508421139027.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1508421169224.jpg