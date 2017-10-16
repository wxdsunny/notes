---
title: Day_06  Combinner和倒序索引
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
-
	* [static 修饰的内部类与其他内部类](#static-修饰的内部类与其他内部类)
	* [Map或reduce的成员变量](#map或reduce的成员变量)
	* [Combinner的过程和作用](#combinner的过程和作用)
	* [倒排索引(目的:搜索和检查)](#倒排索引目的搜索和检查)
		* [倒排索引原理](#倒排索引原理)
		* [多分片问题](#多分片问题)
		* [倒序索引代码](#倒序索引代码)
	* [wordcount的topN问题](#wordcount的topn问题)
		* [topN代码](#topn代码)


## static 修饰的内部类与其他内部类

> - static class 内部类与普通内部类类区别:
> 	- 添加static的话属于静态内部类,和普通类没有多大区别,在使用上可以直接new来实例化对象
> 	- 如果不加static的话,内部类的实例化之前要先实例化外部类

## Map或reduce的成员变量

> 在成员变量位置定义变量
>  mr运行过程中每个map节点只实例化一个map对象 
>  map方法调用时每一行数据记录都会调用一次map方法

## Combinner的过程和作用

> mr运行过程中,每个map节点值实例化一个map对象,map方法调用是每一行数据记录都会调用map方法

Combinner过程:

![][1]

> - 减少shuffle的数据量:
> 	- 每一个map 的输出kv都要经过整个shuffle的过程
> 	- shuffle是最消耗时间的过程,减少shuffle的数据量,能提高mr的运行速度
> 	- 可以先在map节点进行一次聚合,聚合后的结果再发送到reducer上,
> 		- 1减少shuffle的数据量
> 		- 2把reducer上的计算的压力在每一个reducer上进行分担
>     - 通过在mr中添加Combinner的方式给mr配置map端聚合
> 		- Combinner类型是Reducer
> 		- Combinner的reducer方法定义就是map聚合的方法
> 		- Combinner这个Reduce是在map节点发送到Reducer之前来完成的,只计算map端上面的数据
> 		- 真正的Reducer是在独立的Reducer节点上计算的,从每一个map抓取数据过来一同计算
>     - 设置Combinner
> 		- job.setCombinerClass(SomeCombinerClass.class)
>     - 在mr中可以使用Combinner就最好使用
>     - 在溢写之后执行
>     - 不适合使用Combinner场景:
> 		- 1.计算平均值,期望,方差等计算过程不适合使用
> 		- 2.Combinner的输入kv类型和输出kv类型必须保持一致,在计算不支持这种方式的情况下不适用
>     - 当Reducer的输入类型和输出类型一致,并且算法相同时,一般会把Reducer直接当作Combinner,设置IntWritable为1,这样就会便于计算统计


## 倒排索引(目的:搜索和检查)

### 倒排索引原理

> 1.分词解析
> - 先把每一行的每个字段进行解析分词
> - 把分出来的每个词作为索引
> - 把相同的词合并索引
> - 倒排索引合并相同的key(词),排序,索引不合并,排序
> 
> 2.给索引设置id,相同的词会在后面追加id
> 3..根据索引和id就可以快速查找数据

### 多分片问题

> - 概念   当一个文件有多个block时,一个文件会被分割成多个split,,然后每一个split中都存在一个MapReduce任务,Combiner任务,当不同的map
> 中相同的key到达Reducer中,会发生value值重复且没有做合并的情况
> - 解决
> 	- 设置split不分片(目的:解决一个文件可能会有多个分片导致结果错误
> 	- 使用范围:文件不宜过大
> 	- 自定义InputFormat继承oTextInputFormat(mr引擎默认使用TextInputformat)
> 	- 重写isSplitable方法(return false,是关闭分片)
> 		- 代码:


``` stylus
public static class ReversedIndexFormat extends TextInputFormat {
		@Override
		protected boolean isSplitable(JobContext context, Path file) {
			return false;
		}
	}
```

>    - - 在job中设置自定义Format

``` stylus
job.setInputFormatClass(ReversedIndexFormat.class);
```

### 倒序索引代码

``` java
package com.zhiyou100.mapreduce;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class ReversedIndex {
	//解决一个文件会可能有两个分片导致结果错误的情况,自定义一个inpuFormat,来定义一个文件只能有一个分片
	//前提文件不宜过大,否则要另想方法
	public static class ReversedIndexFormat extends TextInputFormat {
		@Override
		protected boolean isSplitable(JobContext context, Path file) {
			return false;
		}
	}
	//map:加载源文件,解析成单词,单词作为key,文件名作为value输出
	public static class ReversedIndexMap extends Mapper<LongWritable, Text, Text, Text> {
		private String [] infos;
		private String filePath;
		private Text oKey = new Text();
		private Text oValue = new Text();
		private FileSplit fileSplit;
		//在调用类的时候,只在开始的时候执行一次
		@Override
		protected void setup(Mapper<LongWritable, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			//设置文件路径,文件是指该map正在执行处理的数据文件
			fileSplit = (FileSplit) context.getInputSplit();
			filePath = fileSplit.getPath().toString();
		}
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			/*infos = value.toString().split("[\\s,\\:,(,),\",\\.,;,\\*,/+,\\[,\\],#,!,\\-,{,},<,>]");*/
			//infos = value.toString().split("[\\s,\\:()\"\\.;\\*/+\\[\\]#!\\-{}<>]");
			infos = value.toString().split("[^a-zA-Z0-9%]");
			if(infos != null && infos.length > 0){
				for (String word : infos) {
					oKey.set(word);
					oValue.set(filePath);
					context.write(oKey, oValue);
				}
			}
		}
		
	}
    //combiner:接收map的输出,然后统计出每个key(单词)在该文件中出现的次数
	//输出key(单词),value(文件名称[单词在该文件中出现的次数])
	public static class ReversedIndexCombiner extends Reducer<Text, Text, Text, Text> {
		private int wordCount;
		private Text oValue = new Text();
		private String filePath;
		private boolean isGet ;
		@Override
		protected void reduce(Text key, Iterable<Text> values, Reducer<Text, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			wordCount = 0;
			isGet = false;
			for (Text path : values) {
				wordCount += 1;
				if(!isGet){
					filePath = path.toString();
					isGet = true;
				}
			}
			oValue.set(filePath+"["+wordCount+"]");
			context.write(key, oValue);
		}
	}
	//reducer:接收Combiner的输出,并把相同key(单词)下不同的values(文件名[词频])按字符串串接起来
	//输出:key(单词),value(文件名[词频]---文件名[词频]----)
	public static class ReversedIndexReducer extends Reducer<Text, Text, Text, Text> {
		private StringBuilder valueStr;
		private Text oValue = new Text();
		private boolean isInitlized = false;
		@Override
		protected void reduce(Text key, Iterable<Text> values, Reducer<Text, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			valueStr = new StringBuilder();
			for (Text value : values) {
				if(isInitlized){
					valueStr.append("-----"+value.toString());
				}else{
					//第一次往valueStr放内容
					valueStr.append(value.toString());
					isInitlized = true;
				}
			}
			oValue.set(valueStr.toString());
			context.write(key, oValue);
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(ReversedIndex.class);
		job.setJobName("倒序索引");
		job.setMapperClass(ReversedIndexMap.class);
		job.setCombinerClass(ReversedIndexCombiner.class);
		job.setReducerClass(ReversedIndexReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		Path inPath = new Path("/reversetext");
		Path outPath = new Path("/bd14/reversetext");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileInputFormat.addInputPath(job, inPath);
		FileOutputFormat.setOutputPath(job, outPath);
		job.setNumReduceTasks(2);
		job.setInputFormatClass(ReversedIndexFormat.class);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```

## wordcount的topN问题

> - 按照顺序计算出词频前三的数据
> 	- 1.第一个mr计算词频,第二个mr只取前三输出,用两个mr
> 	- 2.统计词频,根据词频取TopN,只用一个mr
> 		- map端加载数据,分析数据,再用compare进行聚合,发送给Reducer
> 		- Reducer接收map发送的数据,按照key进行聚合
> 		- 在Reducer上开辟一块内存空间,用来存储map解析放入词频,把单词作为key,单词的出现次数作为value
> 		-  要保证只有一个Reducer节点,则上述方法会造成内存溢出问题
>      - 3 解决内存问题
> 		 - map 中只放三个,可以用treemap
> 		 - 开辟一块内存空间,只保存三个数据
> 		 - 在每个map上分别取TopN(用Combiner来实现)
> 	 - 4 重写reducer的cleanup方法
> 	 - 5 遍历treemap,输出kv

### topN代码
``` java
package com.zhiyou100.mapreduce;

import java.io.IOException;
import java.util.Set;
import java.util.TreeMap;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountTopN {

	public static class WordCountTopNMap extends Mapper<LongWritable, Text, Text, IntWritable> {
		private final IntWritable ONE = new IntWritable(1);
		private Text okey = new Text();
		private String [] infos;
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			for (String word : infos) {
				okey.set(word);
				context.write(okey, ONE);
			}
		}
	}
	public static class WordCountTopNReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private int sum;
		private Text okey = new Text();
		private IntWritable ovalue = new IntWritable();
		//开辟内存空间保存TopN
		//treeMap是一个排序的map,按照key进行排序
		private TreeMap<Integer, String> topN = new TreeMap<>();
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			//把计算结果放入topN中,如果TopN中不满N个元素的话可以直接往里面放,还需要
			//先看TopN中有没有相同的key 如果有的话就把TopN中相同的key对应的value和单词串一起,如果没有就直接放进去
			if(topN.size() < 3){
				if(topN.get(sum)!=null){
					topN.put(sum, topN.get(sum)+"---"+key.toString());
				}else{
					topN.put(sum, key.toString());
				}
			}else{
				//大于等于N 的话放进去一个,然后再删除一个,始终保持TopN中有N个元素
				if(topN.get(sum)!=null){
					topN.put(sum, topN.get(sum)+"---"+key.toString());
					//因为有同key,是归并操作,因此没增也不用删
				}else{
					topN.put(sum, key.toString());
					//topN.remove(topN.lastKey());
					topN.remove(topN.firstKey());
				}
			}
			//放进去后treeMap会自动排序,这时把最后一个再给删掉,保证TopN中只有N个kv对
		}
		@Override
		protected void cleanup(Reducer<Text, IntWritable, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			if(topN != null && !topN.isEmpty()){
				//求最小的前N
				//Set<Integer> keys = topN.keySet();
				//求最大的前N
				Set<Integer> keys = topN.descendingKeySet();
				for (Integer key : keys) {
					okey.set(topN.get(key));
					ovalue.set(key);
					context.write(okey,ovalue);
				}
			}
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(WordCountTopN.class);
		job.setJobName("词频topN");
		job.setMapperClass(WordCountTopNMap.class);
		job.setCombinerClass(WordCountTopNReducer.class);
		job.setReducerClass(WordCountTopNReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		Path inPath = new Path("/reversetext/reverse1.txt");
		Path outPath = new Path("/bd14/topN");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileInputFormat.addInputPath(job, inPath);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
```







  [1]: https://www.github.com/wxdsunny/images/raw/master/1508118485887.jpg