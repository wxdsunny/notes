---
title: Day_09 关系型数据库和mapreducer
tags: avrofile,数据库连接
grammar_cjkRuby: true
---


## sequenceFile avrofile

> 当mr读写的数据内容比较复杂,字段比较多的时候,用avrofile
> 数据内容比较简单,只有两个字段,用sequenceFile

## 关系型数据库连接
### 大数据平台:hadoop

> - 存储:hdfs
> - 处理:
> 	- 数据进入平台:从业务系统数据库:
> 		- rdbms
> 		- oracle,mysql,postgresql
> 		- 维度数据:
> 			- 对事务,类型的描述的数据(行政区划分,时间,组织机构)
> 			- 相对稳定:不经常变化,数据量不大
> 		- 事实数据:
> 			- 业务数据:描述具体发生的事情(订单,存取款,通话清单)
> 			- 不稳定:每分每秒都可能会有新增数据,数据量大

![][1]

### 导入和导出数据库数据:

![][2]

#### 把数据从数据库写出

> - 把数据导出文本到hadoop
> 	-  包含维度(维度数据)和数值(事实数据中聚合统计的汇总数据)
> 		-  维度数据的导入策略:
> 			-  1 . 每日覆盖
> 			-  2 . 定期导
> 			-  3 . 每日增量导
> 			-  4 . 直接读取业务维度数据 
> 	- mapreduce
> 		- 读取关系型数据库 :
> 			- 数据库连接
> 				- 配置job连接数据库
> 				- 	`DBConfiguration.configureDB(Configuration conf, String driverClass, String dbUrl, String userName, String passwd)`
> 			- 要读取的数据库表
> 				- 设置job从数据库中取具体的数据
> 				- 	`DBInputFormat.setInput(Job job, Class<? extends DBWritable> inputClass, String tableName, String conditions, String orderBy,
> String... fieldNames)`
> 			- Inputformat
> 				- DBInputformat
> 				- key : LongWritable  value:DBwritable
> 				- 定义一个MyTable实现或继承DBwritable
> 			- 首先定义一个类实现DBwritable接口来从rbdms中获取的数据进行对接


#### 把数据从数据库中写出代码

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
import org.apache.hadoop.mapreduce.lib.db.DBConfiguration;
import org.apache.hadoop.mapreduce.lib.db.DBInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import com.zhiyou100.bd14.WritteToDb.WordDBwritable;
//把数据库中的某张表抽取到hdfs上
public class ReadFromDB {
	public static class ReadFromDBMap extends Mapper<LongWritable, WordDBwritable, Text, NullWritable> {
		private Text oKey = new Text();
		private NullWritable oValue = NullWritable.get();
		@Override
		protected void map(LongWritable key, WordDBwritable value,
				Mapper<LongWritable, WordDBwritable, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			oKey.set(value.toString());
			context.write(oKey, oValue);
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		DBConfiguration.configureDB(conf, "com.mysql.jdbc.Driver", "jdbc:mysql://192.168.150.1:3306/test", "root", "root");
		Job job = Job.getInstance(conf);
		job.setJarByClass(ReadFromDB.class);
		job.setJobName("输出到关联型数据库");
		job.addFileToClassPath(new Path("/classpath/mysql-connector-java-5.1.39.jar"));
		job.setNumReduceTasks(0);
		job.setMapperClass(ReadFromDBMap.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);
		//设置要从数据库中提取的数据
		DBInputFormat.setInput(job, WordDBwritable.class, "word_count", "", "wc_count","wc_word,wc_count");
		Path outPath = new Path("/bd14/user/userinfo/reader");
		outPath.getFileSystem(conf).delete(outPath, true);
		FileOutputFormat.setOutputPath(job, outPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
		
	}
}

```


#### 把数据写入到数据库

- 写入关系型数据库:
	- DBOutputFormat设置为job的输出格式
	- reduce的key:DBwritable   value:(随便,不会被写入数据库,一般设为Nullwritable)
	- `DBOutputFormat.setOutput(Job job, String tableName, String... fieldNames)`,这只把数据写入到数据库哪张表中
	- `DBConfiguration.configureDB(Configuration conf, String driverClass, String dbUrl, String userName, String passwd)`设置数据库连接

#### 写入数据库的代码:

``` java
package com.zhiyou100.bd14;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.db.DBConfiguration;
import org.apache.hadoop.mapreduce.lib.db.DBOutputFormat;
import org.apache.hadoop.mapreduce.lib.db.DBWritable;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

//把WordCount的结果写入mysql中
public class WritteToDb {
	//对应表:word_count create table word_count(b varchar(255),wc_count integer)
	public static class WordDBwritable implements DBWritable,Writable {
		private String word;
		private int count;
		public String getWord() {
			return word;
		}
		public void setWord(String word) {
			this.word = word;
		}
		public int getCount() {
			return count;
		}
		public void setCount(int count) {
			this.count = count;
		}
		@Override
		public String toString() {
			return "WordDBwritable [word=" + word + ", count=" + count + "]";
		}
		//把数据写入到数据库中
		//insert into word_count(wc_word,wc_count) values(?,?)
		@Override
		public void write(PreparedStatement statement) throws SQLException {
			statement.setString(1, this.word);
			statement.setInt(2, this.count);
		}
		//从数据库中读取数据
		@Override
		public void readFields(ResultSet resultSet) throws SQLException {
			this.word = resultSet.getString("wc_word");
			this.count = resultSet.getInt("wc_count");
		}
		@Override
		public void write(DataOutput out) throws IOException {
			out.writeUTF(this.word);
			out.writeInt(this.count);
		}
		@Override
		public void readFields(DataInput in) throws IOException {
			this.word = in.readUTF();
			this.count = in.readInt();
		}
	}
	public static class WritteToDbMap extends Mapper<LongWritable, Text, Text, IntWritable> {
		private String[] infos;
		private final IntWritable ONE = new IntWritable(1);
		private Text oKey = new Text();
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			for (String word : infos) {
				oKey.set(word);
				context.write(oKey, ONE);
			}
		}
	}
	public static class WritteToDbReducer extends Reducer<Text, IntWritable, WordDBwritable, NullWritable> {
		private int sum;
		private NullWritable oValue = NullWritable.get();
		private WordDBwritable oKey = new WordDBwritable();
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, WordDBwritable, NullWritable>.Context context)
				throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			//设置输出到数据库中的数据
			oKey.setWord(key.toString());
			oKey.setCount(sum);
			context.write(oKey, oValue);
		}
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		DBConfiguration.configureDB(conf, "com.mysql.jdbc.Driver", "jdbc:mysql://192.168.150.1:3306/test", "root", "root");
		Job job = Job.getInstance(conf);
		job.setJarByClass(WritteToDb.class);
		job.setJobName("输出到关联型数据库");
		job.addFileToClassPath(new Path("/classpath/mysql-connector-java-5.1.39.jar"));
		job.setMapperClass(WritteToDbMap.class);
		job.setReducerClass(WritteToDbReducer.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(WordDBwritable.class);
		job.setOutputValueClass(NullWritable.class);
		//设置输入
		FileInputFormat.addInputPath(job, new Path("/README.txt"));
		//设置输出
		DBOutputFormat.setOutput(job, "word_count", 2);
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}

```
### 不使用hadoop

> 不使用Reducer,调用job.setNumberReducer(0)


  [1]: https://www.github.com/wxdsunny/images/raw/master/1508411578496.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1508414097347.jpg