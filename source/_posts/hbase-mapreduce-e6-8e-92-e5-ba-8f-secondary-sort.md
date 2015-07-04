title: HBase MapReduce排序Secondary Sort
tags:
  - Hadoop
  - HBase
  - MapReduce
id: 647
categories:
  - Hadoop
  - HBase
date: 2014-03-20 18:51:08
---

MapReduce是Hadoop中处理大数据的方法，是一个处理大数据的简单算法、编程泛型。虽然思想简单，但其实真正用起来还是有很多问题，不是所有的问题都可以像WordCount那样典型和直观, 有很多需要trick的地方。MapReduce的中心思想是分而治之，数据要松耦合，可以划分为小数据集并行处理，如果数据本身在计算上存在很强的依赖关系，就不要赶鸭子上架，用MapReduce了。
MapReduce编程中，最重要的是要抓住Map和Reduce的input和output，好的input和output可以降低实现的复杂度。最近，写了很多关于MapReduce的job，有倒排索引，统计，排序等。其中，对排序花费了一番功夫，MapReduce做WordCount很好理解,
Map input:[offset, text],  output: [word, 1],

Reduce input: [word, 1], output: [word, totalcount],还可以设置Combiner进行优化。

<!--more-->
但排序不同了，大量的文件记录，分配给map，然后reduce出来的文件分布在各个机器上，怎么保证有序呢？排序是算法中常考常用的，MapReduce做排序还需要理解一下MapReduce过程中，非常magic的过程Shuffle and Sort.[more...]

### Shuffle and Sort过程解析

[![Image and video hosting by TinyPic](http://i59.tinypic.com/1416uzt.jpg)](http://tinypic.com?ref=1416uzt)
如上图，Shuffle的过程包括了Map端和Reduce端。

#### Map端

[![Image and video hosting by TinyPic](http://i59.tinypic.com/j0ftw0.jpg)](http://tinypic.com?ref=j0ftw0)

1.  Input Split分配给Map
2.  Map进行计算，输出[key, value]形式的output
3.  Map的输出结果缓存在内存里
4.  内存中进行Partition，默认是HashPartitioner(采用取模hash (key.hashCode() &amp; Integer.MAX_VALUE) % numReduceTasks)， 目的是将map的结果分给不同的reducer，有几个Partition，就有几个reducer，partition的数目可以在job启动时通过参数 "-Dmapreduc.job.reduces"设置(Hadoop 2.2.0), HashPartitioner可以让map的结果均匀的分给不同机器上的reducer，保证负载均衡。
5.  内存中在Partition结束后，会对每个Partition中的结果按照key进行排序。
6.  排序结束后，相同的key在一起了，如果设置过combiner，就合并数据，减少写入磁盘的记录数
7.  当内存中buffer(default 100M)达到阈值(default 80%)，就会把记录spill(即溢写)到磁盘中，优化Map时可以调大buffer的阈值，缓存更多的数据。
8.  当磁盘中的spill文件数目在3(min.num.spills.for.combine)个（包括）以上, map的新的output会再次运行combiner，而如果磁盘中spill file文件就1~2个，就没有必要调用combiner，因为combiner大多数情况和reducer是一样的逻辑，可以在reduer端再计算。
9.  Map结束时会把spill出来的多个文件合并成一个，merge过程最多10（默认）个文件同时merge成一个文件，多余的文件分多次merge，merge过程是merge sort的算法。
10.  Map端shuffle完毕，数据都有序的存放在磁盘里，等待reducer来拿取

#### Reducer端

shuffle and sort的过程不仅仅在map端，别忘了reducer端还没拿数据呢，reduce job当然不能开启。

1.  Copy phase: reducer的后台进程(default 5个)到被Application Master (Hadoop 2.2), 或者之前的JobTracker 指定的机器上将map的output拷贝到本地，先拷贝到内存，内存满了就拷贝的磁盘。
2.  Sort phase(Merge phase): Reduer采用merge sort，将来自各个map的data进行merge， merge成有序的更大的文件。
3.  如果设置过Combiner，merge过程可能会调用Combiner，调不调用要看在磁盘中产生的文件数目是否超过了设定的阈值。(这一点我还没有确认，但Combiner在Reducer端是可能调用。)
4.  Reduce phase: reduce job开始，输入是shuffle sort过程merge产生的文件。

### MapReduce排序(Secondary Sort)案例

理解了Shuffle的过程后，我们可以着手开始做排序了。
1\. 需求
现在我在HBase里面，存储了10万条文献的评论统计记录，每个文献的评论数目是2~20，每个文献评论统计记录在HBase的存储示例如下：
即表 "pb_stat_comments_count"的记录
[rowkey, column family, column qualifer, value] =
[literature row key, 'info', 'count', 12]
[literature row key, 'info', 'avgts', 1400815843854]
count:是每个文献的记录
avgts：是所有文献的平均评价时间戳
排序要求：将文献按照评价次数逆序排，次数大的靠前，次数相同的平均评价时间相同的靠前。

2\. 思想
MapReduce在Map端排序时，都只按key排序，而现在我们的排序指标有两个字段count, avgts,组合的key，为了保证排序结果对于第二个字段有序，MapReduce里面叫做Secondary Sort,就是个名称而已，可以扩展为保证第三个字段有序，第四，第五...

*   首先，我们需要重新自定义一个组合Key,即我们将看到的示例中的SortKeyPair类, 包括count和avgts两个字段，并overwrite里面的compare to方法，按我们的需求，count大的靠前，count相同，avgts大的靠前。
*   其次，利用SortKeyPair，自定义一个SortComparatorClass给Map端排序时用。
*   再次，自定义PartitionClass.
我们单机伪分布式下，默认只有一个Partition，一个Reducer，Partition有无均可，但是分布式环境下，我们要求分布在各个reducer上的结果有序，即评论次数2~5的在reducer0上，6~10的在reducer 1上，11~15在reducer2上，16~20在reducer3上，如果采用取模hash，会造成各个reducer的结果顺序无法控制，因此我们不能用HashPartitioner。而自己的Partitioner则需要只按count进行partition，不管平均评价时间，我们这里把partition的主要的这个key叫做NaturalKey.
*   最后，我们可以看到reduce端要进行merge，merge过程中，我们需要把相同的count分做一组，需要自定义GroupingComparatorClass。
否则如果是按整个组合key进行分组，每个组合key都是不同的，不能分到一个reduce调用中，reduce方法会被调用10万次（单机环境，一个reducer拿到所有10万条文献）， 而如果按count分组，文献评论数目2~20，reduce方法只会被调用最多19次，减少了开销。
我们可以看到一个Reducer实例中，reduce方法会被调用多次，按道理是调用一次，但是因为Reducer过程会把数据分成多个组，reduce调用多次是可能的。（具体的需要再看看源码）

### 案例代码

1\. 自定义组合key类：SortKeyPair.java

[java]
public class SortKeyPair implements WritableComparable&lt;SortKeyPair&gt; {
	private int count = 0;
	private long avgts = 0;

        //要写一个默认构造函数，否则MapReduce的反射机制，无法创建该类报错
	public SortKeyPair() {

	}

	/**
	 * 
	 * @param count
	 * @param timestamp
	 *            Average timestamp
	 */
	public SortKeyPair(int count, long avgts) {
		super();
		this.count = count;
		this.avgts = avgts;
	}

	public int getCount() {
		return count;
	}

	public void setCount(int count) {
		this.count = count;
	}

	public long getAvgts() {
		return avgts;
	}

	public void setAvgts(long avgts) {
		this.avgts = avgts;
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeInt(this.count);
		out.writeLong(this.avgts);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.count = in.readInt();
		this.avgts = in.readLong();
	}

	/**
	 * We want sort in descending count and descending avgts， Java里面排序默认小的放前面，即返回-1的放前面，这里直接把小值返回1，就会被排序到后面了。
	 */
	@Override
	public int compareTo(SortKeyPair o) {
		int res = this.count &lt; o.getCount() ? 1
				: (this.count == o.getCount() ? 0 : -1);

		if (res == 0) {
			res = this.avgts &lt; o.getAvgts() ? 1
					: (this.avgts == o.getAvgts() ? 0 : -1);
		}

		return res;
	}

        //这个方法需要Overrride
	@Override
	public int hashCode() {
		return Integer.MAX_VALUE - this.count;
	}

	@Override
	public String toString() {
		return this.count + &quot;,&quot; + this.avgts;
	}

        // 这个方法，写不写都不会影响的，至少我测的是这样
	@Override
	public boolean equals(Object obj) {
		if (obj == null) {
			return false;
		}
		if (this == obj) {
			return true;
		}
		if (obj instanceof SortKeyPair) {
			SortKeyPair s = (SortKeyPair) obj;
			return this.count == s.getCount() &amp;&amp; this.avgts == s.getAvgts();
		} else {
			return false;
		}
	}

}
[/java]

2\. 自定义SortComparatorClass, 我命名为：CompositeKeyComparator.java

[java]
public class CompositeKeyComparator extends WritableComparator{

	public CompositeKeyComparator () {
		super(SortKeyPair.class, true);
	}

	@Override
	public int compare(WritableComparable a, WritableComparable b) {
		SortKeyPair s1 = (SortKeyPair)a;
		SortKeyPair s2 = (SortKeyPair)b;

		return s1.compareTo(s2);
	}

}
[/java]

3\. 自定义PartitionerClass, 我命名为NaturalKeyPartitioner.java

[java]
public class NaturalKeyPartitioner extends Partitioner&lt;SortKeyPair, Text&gt;{

	@Override
	public int getPartition(SortKeyPair key, Text value, int numPartitions) {
		// % is hash partition, can't make sure bigger count go to reducer with small id.
		int count = key.getCount();
		if (count &lt;= 5) {
			return 0;
		} else if (count &gt; 5 &amp;&amp; count &lt;= 10) {
			return 1;
		} else if (count &gt; 10 &amp;&amp; count &lt;= 15) {
			return 2;
		} else {
			return 3;
		}
	}

}
[/java]

4\. 自定义GroupingComparatorClass,
我命名为NaturalKeyGroupComparator.java

[java]
public class NaturalKeyGroupComparator extends WritableComparator {

	public NaturalKeyGroupComparator() {
		super(SortKeyPair.class, true);
	}

	@Override
	public int compare(WritableComparable a, WritableComparable b) {
		SortKeyPair s1 = (SortKeyPair) a;
		SortKeyPair s2 = (SortKeyPair) b;
		int res = s1.getCount() &lt; s2.getCount() ? 1 : (s1.getCount() == s2
				.getCount() ? 0 : -1);

		return res;
	}

}
[/java]

5\. 定义Mapper，Reducer，并定义Job提交

[java]
public class SecondarySort {

	/**
	 * It must be declared static, in case of reflection error
	 * 
	 * @author lgrcyanny
	 * 
	 */
	public static class SortMapper extends TableMapper&lt;SortKeyPair, Text&gt; {

                // Map input [hbase row key, hbase result], output: [SortKeyPair, Text] (Text中包括了我们最后输出到文件的信息：literature row key, count, avgts)
		@Override
		protected void map(ImmutableBytesWritable key, Result rs,
				Context context) throws IOException, InterruptedException {
			String literature = Bytes.toString(rs.getRow());
			int count = Integer.valueOf(Bytes.toString(rs.getValue(
					Bytes.toBytes(&quot;info&quot;), Bytes.toBytes(&quot;count&quot;))));
			long avgts = Long.valueOf(Bytes.toString(rs.getValue(
					Bytes.toBytes(&quot;info&quot;), Bytes.toBytes(&quot;avgts&quot;))));
			context.write(new SortKeyPair(count, avgts), new Text(literature
					+ &quot;,&quot; + count + &quot;,&quot; + avgts));
		}

	}

	public static class SortReducer extends
			Reducer&lt;SortKeyPair, Text, Text, Text&gt; {

		/**
		 * Now the key is max SortKeyPair in the list, we just dump the ordered
		 * items
                 * Reduce input: [SorKeyPair, list of text], Output: [text, text] 
		 */
		@Override
		protected void reduce(SortKeyPair key, Iterable&lt;Text&gt; items,
				Context context) throws IOException, InterruptedException {
			Iterator&lt;Text&gt; iterator = items.iterator();
			while (iterator.hasNext()) {
				context.write(null, iterator.next());// 因为value中已经包括了需要输出的信息，SortKeyPair的信息不需要输出，key设置为null即可。
			}
		}
	}

	public static void main(String[] args) throws IOException,
			ClassNotFoundException, InterruptedException {
                // 因为数据源是HBase，因此需要使用HBase中的MapReduce启动设置，可以参考HBase官方网站
                // 自己做测试，可以改成文件输入，看具体需求而定
		Configuration conf = HBaseConfiguration.create();
		Job job = new Job(conf, &quot;Secondarysort&quot;);
		job.setJarByClass(SecondarySort.class);
		Scan scan = new Scan();
		scan.addFamily(Bytes.toBytes(&quot;info&quot;));
		scan.setCaching(5000); // Default is 1, set 500 improve performance
		scan.setCacheBlocks(false); // Close block cache for MR job
		TableMapReduceUtil.initTableMapperJob(&quot;pb_stat_comments_count&quot;, scan,
				SortMapper.class, SortKeyPair.class, Text.class, job);
		job.setReducerClass(SortReducer.class);

		// For secondary sort, 这里设置自定义排序的三个类
		job.setSortComparatorClass(CompositeKeyComparator.class);
		job.setPartitionerClass(NaturalKeyPartitioner.class);
		job.setGroupingComparatorClass(NaturalKeyGroupComparator.class);

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);

                // Reducer的输出，设置为文件输出
		FileOutputFormat.setOutputPath(job, new Path(&quot;secondary-sort-res&quot;));

		long start = System.currentTimeMillis();
		boolean res = job.waitForCompletion(true);
		long end = System.currentTimeMillis();
		if (res) {
			System.out.println(&quot;Job done with time &quot; + (end - start));
		} else {
			throw new IOException(&quot;Job exit with error.&quot;);
		}
	}

}
[/java]

### 总结

MapReduce做排序，麻烦的不是写Mapper和Reducer，而是自定义
CompositeKeyClass
SortComparatorClas
PartitionerClass
GroupingComparatorClass
利用好Shuffle这个很magic课程，可以实现很多奇妙的功能。
本次案例的代码在[我的GitHub上](https://github.com/lgrcyanny/PaperBook-MapReduce/tree/master/src/com/paperbook/mapreduce/stat/secondarysort)。
这些都是个人的理解，如有不对之处，还请指正。

最后，本学期的云计算课到最后了才让我们去些MapReduce程序，之前做无关的MySQL的项目，然后迁移HBase，看着很高端，但是做那个网站到底有什么意义，这是云计算，不是前端计算，HBase in Action中也提到，HBase的设计和MySQL的设计是完全不同的，而MySQL我相信到这个阶段的同学们都会用，何必再次重复劳动呢？直接上HBase不就可以了，本末倒置是我对本学期的课程的失望和吐槽。课程内容多半是概念，涉及技术的有多少？看论文，看概念每一个研究生都可以做到，而我想我们缺的是实践上的指导，比如MapReduce，除了能讲讲WordCount，还能再深入点么？Shuffle的过程可以多说说么？Zookeeper里面的PAXOS算法可以跟我们说说么？学院有Hadoop的集群机器，可以让我们去玩玩不？而不是写那个无聊的网站。
我个人觉得让我们去写一些Hadoop的架构分析，看看源码，也比写那个无聊的网站强。最后9次作业，做的头大，重复劳动，体力劳动，最后还考试，说写的多分会高，有意思么？那么多作业还考试，数据库课程就两次作业+一个论文，但是我觉得我学了也复习了很多知识，比起那些大而空的概念，高到不知哪里去了。
我对云计算，Hadoop,HBase感兴趣，本学期学了很多，但不是课堂上学的，估计我最后对云计算的概念们，要靠下周背PPT了。 我对云计算有兴趣，但对本学期的课程没兴趣。吐槽都是不好的情绪，调整好心态，好好考试吧~BTW，下周还有一次作业。