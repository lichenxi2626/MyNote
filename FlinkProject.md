# FlinkProject

## 1. 环境准备

1. 创建Maven工程UserBehaviorAnalysis，导入依赖

   ```xml
   <properties>
       <flink.version>1.10.1</flink.version>
       <scala.binary.version>2.12</scala.binary.version>
       <kafka.version>2.2.0</kafka.version>
   </properties>
   
   <dependencies>
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-scala_${scala.binary.version}</artifactId>
           <version>${flink.version}</version>
       </dependency>
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
           <version>${flink.version}</version>
       </dependency>
       <dependency>
           <groupId>org.apache.kafka</groupId>
           <artifactId>kafka_${scala.binary.version}</artifactId>
           <version>${kafka.version}</version>
       </dependency>
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
           <version>${flink.version}</version>
       </dependency>
   </dependencies>
   
   <build>
       <plugins>
           <!-- 该插件用于将Scala代码编译成class文件 -->
           <plugin>
               <groupId>net.alchim31.maven</groupId>
               <artifactId>scala-maven-plugin</artifactId>
               <version>4.4.0</version>
               <executions>
                   <execution>
                       <!-- 声明绑定到maven的compile阶段 -->
                       <goals>
                           <goal>compile</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
   
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-assembly-plugin</artifactId>
               <version>3.3.0</version>
               <configuration>
                   <descriptorRefs>
                       <descriptorRef>
                           jar-with-dependencies
                       </descriptorRef>
                   </descriptorRefs>
               </configuration>
               <executions>
                   <execution>
                       <id>make-assembly</id>
                       <phase>package</phase>
                       <goals>
                           <goal>single</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
       </plugins>
   </build>
   ```



## 2. 实时热门商品TopN统计

1. 需求分析

   ```scala
   //1.数据源类型如下 每行数据字段含义为 用户id,商品id,商品品类id,用户行为类型,事件时间戳
   543462,1715,1464116,pv,1511658000
   662867,2244074,1575622,pv,1511658000
   561558,3611281,965809,pv,1511658000
   
   //2.具体需求如下
   每5分钟统计1次近1小时内,页面浏览次数最多的热门商品topN
   
   //3.实现思路
   1.读取数据流,将每行数据封装为样例类对象,提取时间戳
   2.过滤用户行为,仅保留用户行为为"pv"的数据
   3.按照商品id进行分组开窗,窗口长度1hour,步长5min
   4.以窗口和商品id作为分组条件对访问量分组聚合,封装为带有窗口信息的新样例类对象
   5.按照窗口和分组,将同一时间段内的商品访问量整体排序,取topN
   ```

2. 代码实现

   ```scala
   object HotItems {
   
     //定义输入数据的样例类
     case class UserBehavior(userId: Long, itemId: Long, categoryId: Int, behavior: String, timestamp: Long)
   
     //定义封装聚合结果的样例类
     case class ItemViewCount(itemId: Long, windowEnd: Long, count: Long)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       //从文件读取数据
       //    val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\HotItemsAnalysis\\src\\main\\resources\\UserBehavior.csv")
   
       //从kafka读取数据
       val properties = new Properties()
       properties.setProperty("bootstrap.servers", "localhost:9092")
       properties.setProperty("group.id", "consumer-group")
       properties.setProperty("key.deserializer",
         "org.apache.kafka.common.serialization.StringDeserializer")
       properties.setProperty("value.deserializer",
         "org.apache.kafka.common.serialization.StringDeserializer")
       properties.setProperty("auto.offset.reset", "latest")
       val inputStream: DataStream[String] = env.addSource(new FlinkKafkaConsumer[String]("USER_BEHAVIOR", new SimpleStringSchema(), properties))
   
       //转换数据为样例类对象,提取时间戳和watermark
       val dataStream: DataStream[UserBehavior] = inputStream.map { data =>
         val arr: Array[String] = data.split(",")
         UserBehavior(arr(0).toLong, arr(1).toLong, arr(2).toInt, arr(3), arr(4).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L) //升序实现戳本质上是配置了1毫秒延迟的watermark
   
       //对数据流进行处理,先以商品id分组,再进行开窗,最后在窗口内分组聚合
       val aggStream: DataStream[ItemViewCount] = dataStream
         .filter(_.behavior == "pv") //过滤行为
         .keyBy("itemId")
         .timeWindow(Time.hours(1), Time.minutes(5)) //滑动窗口 长度1h 步长5min
         .aggregate(new AggCount(), new ItemViewWindowResult())
       //此处使用自定义聚合函数运算时,单独实现AggregateFunction接口确实可以达到窗口内聚合计算的目的
       //但是AggregateFunction接口属于增量窗口函数,无法获取窗口信息,导致聚合后的输出数据无法得知结果所属的时间区间
       //因此需要额外自定义WindowFunction(全窗口函数)实现类,为增量聚合结果添加时间标记(输出ItemViewCount对象)
   
       //按窗口分组,按count对商品热度降序排序
       val resultStream: DataStream[String] = aggStream
         .keyBy("windowEnd")
         .process(new TopNHotItems(3))
       resultStream.print()
   
       env.execute()
   
     }
   
     //自定义增量聚合函数,泛型参数为[输入数据类型,累加器数据类型, 聚合返回值类型]
     class AggCount extends AggregateFunction[UserBehavior, Long, Long] {
   
       //聚合前调用1次,为accumulator初始化值
       override def createAccumulator(): Long = 0L
   
       //每条数据调用1次,返回中间状态数据
       override def add(value: UserBehavior, accumulator: Long): Long = accumulator + 1
   
       //数据遍历结束且分区merge结束调用1次,返回聚合结果
       override def getResult(accumulator: Long): Long = accumulator
   
       //session window中,窗口合并的逻辑,此处为滑动窗口可无视
       override def merge(a: Long, b: Long): Long = a + b
     }
   
   
     //自定义全窗口函数,泛型参数依次为[聚合函数返回值类型, 窗口函数的返回值类型, key的类型(javaTuple), 窗口类型]
     class ItemViewWindowResult extends WindowFunction[Long, ItemViewCount, Tuple, TimeWindow] {
       //窗口关闭时执行1次
       override def apply(key: Tuple, window: TimeWindow, input: Iterable[Long], out: Collector[ItemViewCount]): Unit = {
         val itemId: Long = key.asInstanceOf[tuple.Tuple1[Long]].f0
         val windowEnd: Long = window.getEnd
         val count: Long = input.toIterator.next()
         out.collect(ItemViewCount(itemId, windowEnd, count))
       }
     }
   
     //自定义KeyedProcessFunction,对同一分组数据进行排序
     class TopNHotItems(topSize: Int) extends KeyedProcessFunction[Tuple, ItemViewCount, String] {
   
       //声明状态,用于收集同一窗口下的所有数据
       lazy val itemViewCountList: ListState[ItemViewCount] = getRuntimeContext.getListState[ItemViewCount](new ListStateDescriptor[ItemViewCount]("itemViewCountList", classOf[ItemViewCount]))
   
       //每条数据调用1次
       override def processElement(i: ItemViewCount, context: KeyedProcessFunction[Tuple, ItemViewCount, String]#Context, collector: Collector[String]): Unit = {
         //将数据收入状态集合
         itemViewCountList.add(i)
         //为了确保一个窗口下的所有数据都保存进状态中,可以设置一个定时器
         //当下一窗口的数据进入时会推进watermark,进而触发定时器调用onTime方法,对此时状态中保存的同一窗口的所有数据进行处理
         context.timerService().registerEventTimeTimer(i.windowEnd + 1) //注册定时器,在窗口关闭1毫秒后触发
       }
   
       //定时器触发时调用 第一个参数为定时器的触发时间戳
       override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Tuple, ItemViewCount, String]#OnTimerContext, out: Collector[String]): Unit = {
   
         //将状态中保存的ItemViewCount数据逐条装入ListBuffer中,用于后续排序处理
         val iter: util.Iterator[ItemViewCount] = itemViewCountList.get().iterator()
         val windowItemViewCountList: ListBuffer[ItemViewCount] = ListBuffer()
         while (iter.hasNext) {
           windowItemViewCountList.append(iter.next())
         }
   
         //清空状态
         itemViewCountList.clear()
   
         //降序排序取topN
         val sortedList: ListBuffer[ItemViewCount] = windowItemViewCountList.sortBy(_.count)(Ordering.Long.reverse).take(topSize)
   
         //将排名信息格式化成String，便于打印输出可视化展示
         val result: StringBuilder = new StringBuilder
         result.append("窗口结束时间：").append(new Timestamp(timestamp - 1)).append("\n")
   
         // 遍历结果列表中的每个ItemViewCount，输出到一行
         for (i <- sortedList.indices) {
           val currentItemViewCount = sortedList(i)
           result.append("NO").append(i + 1).append(": \t")
             .append("商品ID = ").append(currentItemViewCount.itemId).append("\t")
             .append("热门度 = ").append(currentItemViewCount.count).append("\n")
         }
         result.append("==================================\n\n")
   
         Thread.sleep(1000)
         out.collect(result.toString())
       }
     }
   
   }
   ```



## 3. 实时热门页面TopN统计

1. 需求分析

   ```scala
   //1.数据源类型如下 主要字段为ip地址,时间戳,动作类型,url
   83.149.9.216 - - 17/05/2015:10:05:03 +0000 GET /presentations/logstash-monitorama-2013/images/kibana-search.png
   83.149.9.216 - - 17/05/2015:10:05:43 +0000 GET /presentations/logstash-monitorama-2013/images/kibana-dashboard3.png
   83.149.9.216 - - 17/05/2015:10:05:47 +0000 GET /presentations/logstash-monitorama-2013/plugin/highlight/highlight.js
   
   //2.具体需求说明
   每5分钟统计1次,近1小时内访问次数最高的url的TopN
   
   //3.实现思路
   1.读取数据源,将每行数据封装为样例类对象,并提取时间戳,由于数据乱序,还需要设置watermark延迟时间
   2.筛选过滤,保留动作类型为"GET"的日志数据
   3.根据url分组开窗,窗口长度1h,步长5m
   4.以url和窗口为分组条件聚合计算访问次数,并封装为带有窗口信息的新样例类对象
   5.以窗口分组,分别排序每个窗口中访问量最高的url的top3,输出结果
   ```

2. 代码实现

   ```scala
   object HotPagesNetworkFlow {
   
     //封装日志信息的样例类
     case class ApacheLogEvent(ip: String, userId: String, timestamp: Long, method: String, url: String)
   
     //封装按url和窗口聚合访问量后的样例类
     case class PageViewCount(url: String, windowEnd: Long, count: Long)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\NetworkFlowAnalysis\\src\\main\\resources\\apache.log")
   
       val dataStream: DataStream[ApacheLogEvent] = inputStream.map { line =>
         val arr: Array[String] = line.split(" ")
         val simpleDataFormat = new SimpleDateFormat("yyyy/MM/dd:hh:mm:ss")
         val time: Long = simpleDataFormat.parse(arr(3)).getTime
         ApacheLogEvent(arr(0), arr(1), time, arr(5), arr(6))
       }
         .assignTimestampsAndWatermarks(
           new BoundedOutOfOrdernessTimestampExtractor[ApacheLogEvent](Time.minutes(1)) {
             override def extractTimestamp(t: ApacheLogEvent): Long = {
               t.timestamp
             }
           }
         )
   
       //按照url和时间窗口分组,对访问次数进行聚合
       val aggStream: DataStream[PageViewCount] = dataStream
         .filter(_.method == "GET")
         .keyBy(_.url)
         .timeWindow(Time.minutes(10), Time.seconds(5))
         .allowedLateness(Time.minutes(1)) //延迟1分钟关闭窗口
         .sideOutputLateData(new OutputTag[ApacheLogEvent]("late")) //迟到数据归入侧输出流
         .aggregate(new PageCountAgg, new PageCountWindowResult)
   
       //按照时间窗口分组,对url的访问量降序排序后取前三名
       val resultStream: DataStream[String] = aggStream
         .keyBy(_.windowEnd)
         .process(new TopNHotPages(3))
   
       //获取迟到数据并输出
       aggStream.getSideOutput(new OutputTag[ApacheLogEvent]("late")).print("late")
       //获取流处理结果并输出
       resultStream.print("result")
   
       env.execute()
     }
   
     class PageCountAgg extends AggregateFunction[ApacheLogEvent, Long, Long] {
   
       override def createAccumulator(): Long = 0L
   
       override def add(value: ApacheLogEvent, accumulator: Long): Long = accumulator + 1
   
       override def getResult(accumulator: Long): Long = accumulator
   
       override def merge(a: Long, b: Long): Long = a + b
   
     }
   
   
     class PageCountWindowResult extends WindowFunction[Long, PageViewCount, String, TimeWindow] {
       override def apply(key: String, window: TimeWindow, input: Iterable[Long], out: Collector[PageViewCount]): Unit = {
         val count: Long = input.toIterator.next()
         PageViewCount(key, window.getEnd, count)
       }
     }
   
     class TopNHotPages(n: Int) extends KeyedProcessFunction[Long, PageViewCount, String] {
   
       //定义Map状态用于存储(url, count)数据
       lazy val pageCountMap: MapState[String, Long] = getRuntimeContext.getMapState(new MapStateDescriptor[String, Long]("pageCountList", classOf[String], classOf[Long]))
   
       override def processElement(i: PageViewCount, context: KeyedProcessFunction[Long, PageViewCount, String]#Context, collector: Collector[String]): Unit = {
         //将本条数据的url和count存入状态
         pageCountMap.put(i.url, i.count)
         //注册第一个定时器,用于在watermark达到窗口关闭时间,触发聚合运算
         context.timerService().registerEventTimeTimer(i.windowEnd + 1)
         //注册第二个定时器,用于窗口达到延时关闭时间时,触发MapState清空
         context.timerService().registerEventTimeTimer(i.windowEnd + 60000L)
       }
   
       override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, PageViewCount, String]#OnTimerContext, out: Collector[String]): Unit = {
   
         //判断触发的定时器,若为关窗定时器,则清空状态并结束方法的执行
         if (timestamp == ctx.getCurrentKey + 60000L) {
           pageCountMap.clear()
           return
         }
   
         //若触发的是聚合定时器,则执行聚合计算操作
         val pageViews: ListBuffer[(String, Long)] = ListBuffer()
         val iter: util.Iterator[Map.Entry[String, Long]] = pageCountMap.entries().iterator()
         while (iter.hasNext) {
           val entry: Map.Entry[String, Long] = iter.next()
           pageViews.append((entry.getKey, entry.getValue))
         }
   
         //根据count降序排序后取topN
         val topNList: ListBuffer[(String, Long)] = pageViews.sortBy(_._2)(Ordering.Long.reverse).take(n)
   
         //输出结果
         val result: StringBuilder = new StringBuilder
         result.append("窗口结束时间：").append(new Timestamp(timestamp - 1)).append("\n")
         for (i <- topNList.indices) {
           val currentItemViewCount = topNList(i)
           result.append("NO").append(i + 1).append(": \t")
             .append("页面URL = ").append(currentItemViewCount._1).append("\t")
             .append("热门度 = ").append(currentItemViewCount._2).append("\n")
         }
         result.append("\n==================================\n\n")
   
         Thread.sleep(1000)
         out.collect(result.toString())
       }
     }
   
   }
   ```



## 4. 实时统计总浏览量

1. 需求分析

   ```scala
   //1.数据源格式如下 每行数据字段含义为 用户id,商品id,商品品类id,用户行为类型,事件时间戳
   543462,1715,1464116,pv,1511658000
   662867,2244074,1575622,pv,1511658000
   561558,3611281,965809,pv,1511658000
   
   //2.具体需求说明
   根据用户行为日志数据,每小时计算1次页面总浏览量
   
   //3.实现思路
   1.读取数据源,将每条数据封装为样例类对象,提取时间戳
   2.过滤筛选,保留用户行为为"pv"的数据
   3.开窗聚合同一窗口下的总浏览量(1条数据即为1条浏览量),窗口长度为1h
   4.输出结果
   ```

2. 代码实现

   ```scala
   object PageView {
   
     //封装每条用户行为日志数据的样例类
     case class UserBehavior(userId: Long, itemId: Long, categoryId: Int, behavior: String, timestamp: Long)
     //封装每个window聚合结果的样例类
     case class PvCount(windowEnd: Long, count: Long)
     
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\HotItemsAnalysis\\src\\main\\resources\\UserBehavior.csv")
       val dataStream: DataStream[UserBehavior] = inputStream.map { data =>
         val arr: Array[String] = data.split(",")
         UserBehavior(arr(0).toLong, arr(1).toLong, arr(2).toInt, arr(3), arr(4).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
   
       //同一每个小时内的pageview总量
       val pvStream: DataStream[PvCount] = dataStream
         .filter(_.behavior == "pv")
         .map {
           new MyMapper
         } //使用自定义mapper生成随机key
         .keyBy(_._1) //对随机key分组,实现多线程并行的窗口聚合计算
         .timeWindow(Time.hours(1))
         .aggregate(new PvCountAgg, new PvCountWindowResult)
   
       val resultStream: DataStream[PvCount] = pvStream
         .keyBy(_.windowEnd)
         .process(new TotalPvCountResult)
   
       resultStream.print("rs")
   
       env.execute()
     }
   
     class MyMapper() extends MapFunction[UserBehavior, (String, Long)] {
       override def map(value: UserBehavior): (String, Long) = {
         (Random.nextString(10), 1L)
       }
     }
   
     class PvCountAgg extends AggregateFunction[(String, Long), Long, Long] {
       override def createAccumulator(): Long = 0L
   
       override def add(value: (String, Long), accumulator: Long): Long = accumulator + 1
   
       override def getResult(accumulator: Long): Long = accumulator
   
       override def merge(a: Long, b: Long): Long = a + b
     }
   
   
     class PvCountWindowResult extends WindowFunction[Long, PvCount, String, TimeWindow] {
       override def apply(key: String, window: TimeWindow, input: Iterable[Long], out: Collector[PvCount]): Unit = {
         out.collect(PvCount(window.getEnd, input.toIterator.next()))
       }
     }
   
     class TotalPvCountResult() extends KeyedProcessFunction[Long, PvCount, PvCount] {
   
       //声明状态用于记录当前窗口下的总浏览量
       lazy val totalCount: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("totalCount", classOf[Long]))
   
       override def processElement(i: PvCount, context: KeyedProcessFunction[Long, PvCount, PvCount]#Context, collector: Collector[PvCount]): Unit = {
         //将数据累加到状态中
         val currentTotalCount: Long = totalCount.value() + i.count
         totalCount.update(currentTotalCount)
         //注册定时器,窗口关闭时调用onTime方法,输出窗口的聚合结果
         context.timerService().registerEventTimeTimer(i.windowEnd + 1)
       }
   
       override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, PvCount, PvCount]#OnTimerContext, out: Collector[PvCount]): Unit = {
         //收集聚合结果并返回
         out.collect(PvCount(timestamp - 1, totalCount.value()))
         //清空状态
         totalCount.clear()
       }
     }
   
   }
   ```



## 5. 实时统计独立访客数量

1. 需求分析

   ```scala
   //1.数据源格式如下 每行数据字段含义为 用户id,商品id,商品品类id,用户行为类型,事件时间戳
   543462,1715,1464116,pv,1511658000
   662867,2244074,1575622,pv,1511658000
   561558,3611281,965809,pv,1511658000
   
   //2.需求具体说明
   每小时滚动统计1次,所有不重复用户的访问总次数
   
   //3.实现思路
   1.读取数据源,将每条数据转换为样例类对象,提取时间戳
   2.筛选过滤,保留用户行为为"pv"的数据
   3.开窗分组,窗口长度1h
   4.窗口内的每条数据通过布隆过滤器实现对用户id的去重,将去重后的数据结果聚合计算,并输出
   ```

2. 引入redis依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>2.8.1</version>
       </dependency>
   </dependencies>
   ```

3. 代码实现

   ```scala
   object UvWithBloom {
   
     //封装每条用户行为日志数据的样例类
     case class UserBehavior(userId: Long, itemId: Long, categoryId: Int, behavior: String, timestamp: Long)
   
     //封装窗口内独立用户访问量数据的样例类
     case class UvCount(windowEnd: Long, count: Long)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\HotItemsAnalysis\\src\\main\\resources\\UserBehavior.csv")
       val dataStream: DataStream[UserBehavior] = inputStream.map { data =>
         val arr: Array[String] = data.split(",")
         UserBehavior(arr(0).toLong, arr(1).toLong, arr(2).toInt, arr(3), arr(4).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
   
       val resultStream: DataStream[UvCount] = dataStream
         .filter(_.behavior == "pv")
         .map(data => ("uv", data.userId))
         .keyBy(_._1)
         .timeWindow(Time.hours(1))
         .trigger(new MyTrigger) //使用触发器,每来一条数据就触发1次窗口的计算(process),无需保存状态数据,全部交给redis处理
         .process(new UvCountWithBloom)
   
       resultStream.print()
   
       env.execute()
   
     }
   
     //自定义触发器,泛型参数为[输入数据类型, 窗口数据类型]
     class MyTrigger extends Trigger[(String, Long), TimeWindow] {
       //每条数据触发的操作类型,FIRE_AND_PURGE表示触发窗口计算并清空窗口状态
       override def onElement(t: (String, Long), l: Long, w: TimeWindow, triggerContext: Trigger.TriggerContext): TriggerResult = TriggerResult.FIRE_AND_PURGE
   
       override def onProcessingTime(l: Long, w: TimeWindow, triggerContext: Trigger.TriggerContext): TriggerResult = TriggerResult.CONTINUE
   
       override def onEventTime(l: Long, w: TimeWindow, triggerContext: Trigger.TriggerContext): TriggerResult = TriggerResult.CONTINUE
   
       override def clear(w: TimeWindow, triggerContext: Trigger.TriggerContext): Unit = {}
     }
   
     //自定义布隆过滤器
     class Bloom(size: Long) extends Serializable {
       private val cap = size
   
       def hash(value: String, seed: Int): Long = {
         var result = 0
         for (i <- 0 until value.length) {
           result = result * seed + value.charAt(i)
         }
         (cap - 1) & result
       }
     }
   
     class UvCountWithBloom extends ProcessWindowFunction[(String, Long), UvCount, String, TimeWindow] {
       lazy val jedis = new Jedis("hadoop201", 6379)
       lazy val bloom = new Bloom(1 << 29)
   
       //正常为窗口关闭时调用1次,配合触发器使用每条数据调用一次
       override def process(key: String, context: Context, elements: Iterable[(String, Long)], out: Collector[UvCount]): Unit = {
   
         //使用redis创建2个模型
         //第一个用于记录userId去重后的窗口内总数 key为"UvCount" field为每个窗口的结束时间 value为窗口内的不同用户数量
         //第二个用于记录每个窗口内的bloom过滤器存储位图, key为窗口的结束时间
   
         //使用当前窗口的结束时间从redis的UvCount中获取用户数量
         val redisField: String = context.window.getEnd.toString
         var userCount = 0L
         if (jedis.hget("UvCount", redisField) != null) {
           userCount = jedis.hget("UvCount", redisField).toLong
         }
   
         //使用本条数据的userId,通过bloom过滤器计算后获取offset
         val userId: String = elements.last._2.toString
         val offset: Long = bloom.hash(userId, 77) //任意赋值1个seed
         val ifExist: lang.Boolean = jedis.getbit(redisField, offset)
         if (!ifExist) {
           //如果不存在,就设置为存在,并将count+1
           jedis.setbit(redisField, offset, true)
           jedis.hset("UvCount", redisField, (userCount + 1).toString)
         }
   
       }
     }
   
   }
   ```



## 6. 实时统计各渠道市场推广量

1. 需求分析

   ```scala
   //1.自定义数据源,主要字段为用户id,用户行为,渠道,时间戳
   
   //2.需求具体说明
   以用户行为和渠道组合维度统计近1天内的推广量(1条数据即1条推广),每5秒刷新1次
   
   //3.实现思路
   1.读取数据源数据,提取时间戳
   2.过滤筛选,去除用户行为为"uninstall"的数据
   3.按照用户行为和渠道来源两个组合维度对数据进行分组
   4.对每个分组的数据进行开窗,窗口长度为1d,步长为5s
   5.分组分窗口聚合计算推广量,输出结果
   ```

2. 代码实现

   ```scala
   object AppMarketByChannel {
   
     //封装用户行为渠道模拟数据的样例类
     case class MarketUserBehavior(userId: String, behavior: String, channel: String, timestamp: Long)
   
     //封装开窗聚合后数据的样例类
     case class MarketViewCount(windowStart: String, windowEnd: String, channel: String, behavior: String, count: Long)
   
     //自定义数据源
     class SimulatedSource extends RichSourceFunction[MarketUserBehavior] {
   
       //模拟用户行为数据
       val behaviorSeq: Seq[String] = Seq("view", "click", "download", "install", "uninstall")
       //模拟访问渠道数据
       val channelSeq: Seq[String] = Seq("appstore", "weibo", "wechat", "tieba")
       var flag = true
       val random: Random = Random
   
       override def run(sourceContext: SourceFunction.SourceContext[MarketUserBehavior]): Unit = {
         var count = 0L
         while (flag && count < Long.MaxValue) {
           val id: String = UUID.randomUUID().toString
           val behavior: String = behaviorSeq(random.nextInt(behaviorSeq.size))
           val channel: String = channelSeq(random.nextInt(channelSeq.size))
           val ts: Long = System.currentTimeMillis()
           sourceContext.collect(MarketUserBehavior(id, behavior, channel, ts))
           count += 1
           Thread.sleep(10)
         }
       }
   
       override def cancel(): Unit = {
         flag = false
       }
   
     }
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val dataStream: DataStream[MarketUserBehavior] = env.addSource(new SimulatedSource)
         .assignAscendingTimestamps(_.timestamp)
   
       val resultStream: DataStream[MarketViewCount] = dataStream
         .filter(_.behavior != "uninstall")
         .keyBy(data => (data.channel, data.behavior))
         .timeWindow(Time.days(1), Time.seconds(5))
         .process(new MarketCountByChannel)
   
       resultStream.print()
   
       env.execute()
     }
   
   
     class MarketCountByChannel extends ProcessWindowFunction[MarketUserBehavior, MarketViewCount, (String, String), TimeWindow] {
   
       //窗口关闭时执行1次,直接通过窗口内数据迭代器的长度获取数据量
       override def process(key: (String, String), context: Context, elements: Iterable[MarketUserBehavior], out: Collector[MarketViewCount]): Unit = {
         val windowStart: String = new Timestamp(context.window.getStart).toString
         val windowEnd: String = new Timestamp(context.window.getEnd).toString
         val channel: String = key._1
         val behavior: String = key._2
         val count: Long = elements.size.toLong
   
         out.collect(MarketViewCount(windowStart, windowEnd, channel, behavior, count))
       }
   
     }
   
   }
   ```



## 7. 实时统计各省份的广告点击量（含黑名单机制）

1. 需求分析

   ```scala
   //1.数据源类型如下 字段依次为用户id,广告id,省份,地级市,时间戳
   543462,1715,beijing,beijing,1511658000
   662867,2244074,guangdong,guangzhou,1511658060
   561558,3611281,guangdong,shenzhen,1511658120
   
   //2.需求具体说明
   分别统计各省市内,对不同广告的近1天内的总点击量,5秒刷新1次
   对于1个自然日内重复点击同1广告次数超过一定次数的用户,需要加入黑名单,并筛除该用户的
   ```

2. 代码实现

   ```scala
   object AdClichAnalysis {
   
     //封装用户点击日志数据的样例类
     case class AdClickLog(userId: Long, adId: Long, province: String, city: String, timestamp: Long)
   
     //封装窗口内各省份点击量数据的样例类
     case class AdClickCountByProvince(windowStart: String, windowEnd: String, province: String, count: Long)
   
     //封装黑名单用户的侧输出流样例类
     case class BlackListUser(userId: Long, adId: Long, msg: String)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\MarketAnalysis\\src\\main\\resources\\AdClickLog.csv")
   
       val dataStream: DataStream[AdClickLog] = inputStream.map { data =>
         val datas: Array[String] = data.split(",")
         AdClickLog(datas(0).toLong, datas(1).toLong, datas(2), datas(3), datas(4).toLong)
       }
         .assignAscendingTimestamps(_.timestamp)
   
       //过滤有刷单行为的用户
       val filterStream: DataStream[AdClickLog] = dataStream
         .keyBy(data => (data.userId, data.adId))
         .process(new FilterBlackListUser(10))
   
       val resultStream: DataStream[AdClickCountByProvince] = filterStream
         .keyBy(_.province)
         .timeWindow(Time.days(1), Time.seconds(5))
         .aggregate(new AdCountAgg, new AdCountWindow)
   
       resultStream.print("result")  //正常数据输出
       filterStream.getSideOutput(new OutputTag[BlackListUser]("warning")).print("warning")  //黑名单用户数据输出
   
       env.execute()
   
     }
   
     class FilterBlackListUser(maxCount: Int) extends KeyedProcessFunction[(Long, Long), AdClickLog, AdClickLog] {
   
       //记录本条数据的用户对本条数据的广告的一天内的点击次数
       lazy val count: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("count", classOf[Long]))
       //记录每天0点定时清空count状态的时间戳
       lazy val reset: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("reset", classOf[Long]))
       //标记用户是否进入黑名单(Boolean型状态默认为flase)
       lazy val isBlack: ValueState[Boolean] = getRuntimeContext.getState(new ValueStateDescriptor[Boolean]("isBlask", classOf[Boolean]))
   
       override def processElement(i: AdClickLog, context: KeyedProcessFunction[(Long, Long), AdClickLog, AdClickLog]#Context, collector: Collector[AdClickLog]): Unit = {
         val currentCount: Long = count.value()
   
         //判断是否是当天第一条数据,若是则创建当天结束时间的定时器
         if (currentCount == 0) {
           reset.update((context.timerService().currentProcessingTime() / 86400000 + 1) * 86400000)
           context.timerService().registerEventTimeTimer(reset.value())
         }
   
         //判断count是否达到阈值
         if (currentCount >= maxCount) {
           //判断用户是否已经加入黑名单,若没有则将该用户信息输出到侧输出流(1个黑名单用户仅输出1次)
           if (!isBlack.value()) {
             isBlack.update(true)
             context.output(new OutputTag[BlackListUser]("warning"), BlackListUser(i.userId, i.adId, "warning"))
           }
           //停止该条黑名单用户数据的处理,不进行输出
           return
         }
   
         //正常数据,count+1后输出数据
         count.update(currentCount + 1)
         collector.collect(i)
   
       }
   
       override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[(Long, Long), AdClickLog, AdClickLog]#OnTimerContext, out: Collector[AdClickLog]): Unit = {
         //触发自然日结束的定时器,清空黑名单和点击数状态
         if (timestamp == reset.value()) {
           isBlack.clear()
           count.clear()
         }
       }
   
     }
   
     class AdCountAgg extends AggregateFunction[AdClickLog, Long, Long] {
       override def createAccumulator(): Long = 0L
   
       override def add(value: AdClickLog, accumulator: Long): Long = accumulator + 1
   
       override def getResult(accumulator: Long): Long = accumulator
   
       override def merge(a: Long, b: Long): Long = a + b
     }
   
     class AdCountWindow extends WindowFunction[Long, AdClickCountByProvince, String, TimeWindow] {
       override def apply(key: String, window: TimeWindow, input: Iterable[Long], out: Collector[AdClickCountByProvince]): Unit = {
         val windowStart: Long = window.getStart
         val windowEnd: Long = window.getEnd
         val count: Long = input.iterator.next()
         out.collect(AdClickCountByProvince(windowStart.toString, windowEnd.toString, key, count))
       }
     }
   
   }
   ```

   

## 8. 实时监控恶意登录

1. 需求分析

   ```scala
   //1.数据源类型如下,字段依次为用户id,登陆ip地址,登陆结果,时间戳
   5402,83.149.11.115,success,1558430815
   23064,66.249.3.15,fail,1558430826
   5692,80.149.25.29,fail,1558430833
   
   //2.需求具体说明
   为防止恶意登陆行为,当同一用户短时间内连续出现多次登陆失败行为时(5秒内失败3次),输出报警信息
   
   //3.实现思路
   1.读取数据源数据,转化为样例类对象,提取时间戳并配置watermark延迟
   2.采用CEP自定义模式序列,对数据流进行匹配,将5秒内连续3次登陆失败的复杂事件提取到复杂事件流
   3.提取复杂事件流数据并输出报警信息
   ```

2. 引入CEP依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-cep-scala_${scala.binary.version}</artifactId>
           <version>${flink.version}</version>
       </dependency>
   </dependencies>>
   ```

3. 代码实现

   ```scala
   object LoginFailCep {
   
     //封装登陆日志数据的样例类
     case class LoginEvent(userId: Long, ip: String, eventType: String, timestamp: Long)
   
     //封装登陆异常报警信息数据的样例类
     case class LoginFailWarning(userId: Long, firstTime: Long, lastTime: Long, msg: String)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\LoginFailDetect\\src\\main\\resources\\LoginLog.csv")
   
       val dataStream: DataStream[LoginEvent] = inputStream.map { line =>
         val arr: Array[String] = line.split(",")
         LoginEvent(arr(0).toLong, arr(1), arr(2), arr(3).toLong)
       }
         .assignTimestampsAndWatermarks(
           new BoundedOutOfOrdernessTimestampExtractor[LoginEvent](Time.seconds(3)) {
             override def extractTimestamp(t: LoginEvent): Long = {
               t.timestamp * 1000L
             }
           }
         )
   
       //定义模式序列,此处为用户在5秒内连续失败登陆3次
       val loginFailPattern: Pattern[LoginEvent, LoginEvent] = Pattern
         .begin[LoginEvent]("loginFail").where(_.eventType == "fail").times(3).consecutive().within(Time.seconds(5))
   
       //使用匹配模式筛选数据流的数据,获得符合事件条件的PatternStream
       //本质上是对数据流进行一种高级的filter处理,保留匹配自定义事件的一或多条数据,并将这个事件整体作为新数据流的一条数据输出
       val patternStream: PatternStream[LoginEvent] = CEP.pattern(dataStream.keyBy(_.userId), loginFailPattern)
   
       //对PatternStream的每个事件的数据使用自定义逻辑进行处理
       val resultStream: DataStream[LoginFailWarning] = patternStream.select(new LoginFailEventMatch())
   
       resultStream.print()
   
       env.execute()
   
     }
   
     //自定义类继承PatternSelectFunction类
     //泛型参数为[PatternStream的类型, 输出的数据类型]
     class LoginFailEventMatch() extends PatternSelectFunction[LoginEvent, LoginFailWarning] {
   
       //PatternStream中封装的每个复杂事件调用1次
       //map中每一个key对应事件中的一条自定义的数据名称,value则对应这一条数据
       override def select(map: util.Map[String, util.List[LoginEvent]]): LoginFailWarning = {
         val iter: util.Iterator[LoginEvent] = map.get("loginFail").iterator()
         val firstEvent: LoginEvent = iter.next()
         val secondEvent: LoginEvent = iter.next()
         val thirdEvent: LoginEvent = iter.next()
         LoginFailWarning(firstEvent.userId, firstEvent.timestamp, thirdEvent.timestamp, "login fail")
       }
     }
   
   }
   ```



## 9. 实时订单状态监控

1. 需求分析

   ```scala
   //1.数据源类型如下,字段依次为订单id,订单操作,支付单号,时间戳
   34734,create,,1558430859
   34732,pay,32h3h4b4t,1558430861
   34735,create,,1558430862
   
   //2.需求说明
   实时收集用户下单数据,将15分钟内完成支付的订单和超过15分钟未支付的订单分流输出
   
   //3.实现思路
   1.读取数据源,将数据转换为样例类对象,提取事件时间戳
   2.自定义模式序列,匹配从创建订单起15分钟内完成支付的订单数据
   3.自定义对复杂事件流数据中符合规则的数据和超时未支付数据的处理函数,获取2个输出流并输出结果
   ```

2. 代码实现1（CEP）

   ```scala
   object OrderTimeOut {
   
     //封装订单日志数据的样例类
     case class OrderEvent(orderId: Long, eventType: String, txId: String, timestamp: Long)
   
     //封装订单结果信息的样例类
     case class OrderResult(orderId: Long, resultMsg: String)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\OrderPayDetect\\src\\main\\resources\\OrderLog.csv")
   
       val dataStream: KeyedStream[OrderEvent, Long] = inputStream.map { data =>
         val arr: Array[String] = data.split(",")
         OrderEvent(arr(0).toLong, arr(1), arr(2), arr(3).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
         .keyBy(_.orderId)
   
       //定义模式序列,筛选下单15分钟内支付成功的订单
       val pattern: Pattern[OrderEvent, OrderEvent] = Pattern
         .begin[OrderEvent]("create").where(_.eventType == "create")
         .followedBy("pay").where(_.eventType == "pay")
         .within(Time.minutes(15))
   
       //使用自定义模式序列对数据流进行检索匹配
       val cepStream: PatternStream[OrderEvent] = CEP.pattern(dataStream, pattern)
   
       //自定义侧输出流标签,同样使用OrderResult样例类输出
       val orderTimeoutOutputTag = new OutputTag[OrderResult]("orderTimeout")
   
       //将成功支付的订单数据写入输出流,超时未支付订单的数据写入侧输出流
       val resultStream: DataStream[OrderResult] = cepStream.select(orderTimeoutOutputTag, new OrderTimeoutSelect, new OrderPayedSelect)
   
       resultStream.print("payed_order")
       resultStream.getSideOutput(orderTimeoutOutputTag).print("timeout_order")
   
       env.execute()
   
     }
   
     //自定义PatternTimeoutFunction实现类,用于收集模式序列中within匹配超时的数据,处理后输出到侧输出流
     class OrderTimeoutSelect extends PatternTimeoutFunction[OrderEvent, OrderResult] {
       override def timeout(map: util.Map[String, util.List[OrderEvent]], timestamp: Long): OrderResult = {
         val orderId: Long = map.get("create").iterator().next().orderId
         OrderResult(orderId, "timeout:" + timestamp)
       }
     }
   
     //自定义PatternSelectFunction实现类,用于收集模式序列匹配到的数据,并处理输出
     class OrderPayedSelect extends PatternSelectFunction[OrderEvent, OrderResult] {
       override def select(map: util.Map[String, util.List[OrderEvent]]): OrderResult = {
         val orderId: Long = map.get("create").iterator().next().orderId
         OrderResult(orderId, "payed success")
       }
     }
   
   }
   ```

3. 代码实现2（ProcessFunction）

   ```scala
   object OrderTimeoutWIthProcFunc {
   
     //封装订单日志数据的样例类
     case class OrderEvent(orderId: Long, eventType: String, txId: String, timestamp: Long)
   
     //封装订单结果信息的样例类
     case class OrderResult(orderId: Long, resultMsg: String)
   
     def main(args: Array[String]): Unit = {
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       val inputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\OrderPayDetect\\src\\main\\resources\\OrderLog.csv")
   
       val dataStream: KeyedStream[OrderEvent, Long] = inputStream.map { data =>
         val arr: Array[String] = data.split(",")
         OrderEvent(arr(0).toLong, arr(1), arr(2), arr(3).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
         .keyBy(_.orderId)
   
       val resultStream: DataStream[OrderResult] = dataStream.process(new OrderPayedStateResult)
   
       resultStream.print("payed success")
       resultStream.getSideOutput(new OutputTag[OrderResult]("timeout")).print()
   
       env.execute()
     }
   
     class OrderPayedStateResult extends KeyedProcessFunction[Long, OrderEvent, OrderResult] {
   
       //保存订单是否创建的状态
       lazy val createState: ValueState[Boolean] = getRuntimeContext.getState(new ValueStateDescriptor[Boolean]("isCreate", classOf[Boolean]))
       //保存订单是否支付的状态
       lazy val payState: ValueState[Boolean] = getRuntimeContext.getState(new ValueStateDescriptor[Boolean]("isPayed", classOf[Boolean]))
       //保存订单创建后开始计时的时间戳
       lazy val timer: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("timer", classOf[Long]))
       //侧输出流标签
       val timeoutOutputTag = new OutputTag[OrderResult]("timeout")
   
   
       override def processElement(value: OrderEvent, ctx: KeyedProcessFunction[Long, OrderEvent, OrderResult]#Context, out: Collector[OrderResult]): Unit = {
   
         //获取状态信息
         val isCreate: Boolean = createState.value()
         val isPayed: Boolean = payState.value()
         val timestamp: Long = timer.value()
   
         //分情况处理数据
         if (value.eventType == "create"){  //本条数据为创建订单数据
           if (isPayed){ //该订单的支付信息已获取,输出成功支付信息,并清空所有状态和定时器
             out.collect(OrderResult(value.orderId, "payed success"))
             createState.clear()
             payState.clear()
             timer.clear()
             ctx.timerService().deleteEventTimeTimer(timestamp)
           } else {  //该订单的支付信息未获取,修改创建状态,并注册15分钟的定时器
             createState.update(true)
             val waitTimestamp: Long = value.timestamp * 1000 + 15 * 60 * 1000
             ctx.timerService().registerEventTimeTimer(waitTimestamp)
             timer.update(waitTimestamp)
           }
         } else {  //本条数据为支付订单数据
           if (isCreate){ //该订单的创建信息已经获取,输出成功支付信息,并清空所有状态和定时器
             out.collect(OrderResult(value.orderId, "payed success"))
             createState.clear()
             payState.clear()
             timer.clear()
             ctx.timerService().deleteEventTimeTimer(timestamp)
           } else { //该订单的创建信息未获取,修改支付状态,直接使用支付时间的时间戳注册定时器
             payState.update(true)
             val waitTimestamp: Long = value.timestamp * 1000
             ctx.timerService().registerEventTimeTimer(waitTimestamp)
             timer.update(waitTimestamp)
           }
         }
   
       }
   
       override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, OrderEvent, OrderResult]#OnTimerContext, out: Collector[OrderResult]): Unit = {
   
         //判断定时器的触发种类
         if (createState.value()){ //创建订单数据注册的定时器,即15分钟未支付
           ctx.output(timeoutOutputTag, OrderResult(ctx.getCurrentKey, "order timeout"))
         } else { //支付订单数据注册的定时器,即丢失订单创建数据
           ctx.output(timeoutOutputTag, OrderResult(ctx.getCurrentKey, "payed but no create order log"))
         }
   
         //清空状态
         createState.clear()
         payState.clear()
         timer.clear()
   
       }
     }
   
   }
   ```



## 10. 实时匹配订单支付信息

1. 需求分析

   ```scala
   //1.数据源类型如下
   //订单状态数据 字段依次为订单id,订单操作,支付单号,时间戳
   34734,create,,1558430859
   34732,pay,32h3h4b4t,1558430861
   34735,create,,1558430862
   //支付到账数据 字段依次为支付单号,支付渠道,时间戳
   ewr342as4,wechat,1558430845
   sd76f87d6,wechat,1558430847
   3hu3k2432,alipay,1558430848
   
   //2.需求具体描述
   对订单状态数据中的支付信息和支付到账数据中的到账数据按照订单编号进行一一匹配,对于一定时间内未匹配到的支付数据或是到账数据需要输出到侧输出流
   
   //3.实现思路
   1.读取两个数据源的数据,分别使用样例类封装,并提取时间戳
   2.将两个数据流按照订单编号分组
   3.对分组后的两个数据流进行connect连接操作,相同订单的数据会自动进入同一分组
   4.使用自定义函数处理连接后的数据,函数中若一定时间内收到2条数据则认为匹配成功正常输出,若未在规定时间内匹配成功则输出到侧输出流
   ```

2. 代码实现

   ```scala
   object TxMatch {
   
     //声明封装订单状态数据的样例类
     case class OrderEvent(orderId: Long, eventType: String, txId: String, timestamp: Long)
   
     //声明封装支付到账数据的样例类
     case class ReceiptEvent(txId: String, payChannel: String, timestamp: Long)
   
     def main(args: Array[String]): Unit = {
   
       val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
       env.setParallelism(1)
       env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
       //分别读取两份日志数据并转换为相应样例类的数据流,并提取key
       val orderInputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\OrderPayDetect\\src\\main\\resources\\OrderLog.csv")
       val orderEventStream = orderInputStream.map { data =>
         val arr: Array[String] = data.split(",")
         OrderEvent(arr(0).toLong, arr(1), arr(2), arr(3).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
         .filter(_.eventType == "pay")
         .keyBy(_.txId)
   
       val receiptInputStream: DataStream[String] = env.readTextFile("F:\\atguigu\\flink\\UserBehaviorAnalysis\\OrderPayDetect\\src\\main\\resources\\ReceiptLog.csv")
       val receiptEventStream = receiptInputStream.map { data =>
         val arr: Array[String] = data.split(",")
         ReceiptEvent(arr(0), arr(1), arr(2).toLong)
       }
         .assignAscendingTimestamps(_.timestamp * 1000L)
         .keyBy(_.txId)
   
       //连接两条数据流,使用自定义CoProcessFunction进行匹配
       val resultStream: DataStream[(OrderEvent, ReceiptEvent)] = orderEventStream.connect(receiptEventStream)
         .process(new TxPayMatchResult)
   
       //输出数据
       resultStream.print("matched success")
       resultStream.getSideOutput(new OutputTag[OrderEvent]("receipt timeout")).print("no receipt log")
       resultStream.getSideOutput(new OutputTag[ReceiptEvent]("pay timeout")).print("no pay log")
   
       env.execute()
   
     }
   
     //自定义CoProcessFunction实现类,对同一订单的支付数据和到账数据进行匹配
     //泛型参数为[数据流1的数据类型, 数据流2的数据类型, 输出数据类型]
     class TxPayMatchResult extends CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)] {
   
       //声明用于存储订单支付信息的状态
       lazy val payState: ValueState[OrderEvent] = getRuntimeContext.getState(new ValueStateDescriptor[OrderEvent]("payState", classOf[OrderEvent]))
       //声明用于存储订单到账信息的状态
       lazy val receiptState: ValueState[ReceiptEvent] = getRuntimeContext.getState(new ValueStateDescriptor[ReceiptEvent]("receiptState", classOf[ReceiptEvent]))
       //声明存储定时器触发时间的状态
       lazy val timer: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("timer", classOf[Long]))
   
       //定义侧输出流的数据类型和名称,用于输出仅收到支付数据没收到到账数据的订单
       private val payOutputTag = new OutputTag[OrderEvent]("receipt timeout")
       //定义侧输出流的数据类型和名称,用于输出仅收到到账数据没收到支付数据的订单
       private val receiptOutputTag = new OutputTag[ReceiptEvent]("order timeout")
   
       override def processElement1(value: OrderEvent, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#Context, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {
         //判断是否收到支付到账信息
         if (receiptState.value() != null) { //已收到到账信息,输出支付成功信息,清空状态和定时器
           out.collect((value, receiptState.value()))
           payState.clear()
           receiptState.clear()
           ctx.timerService().deleteEventTimeTimer(timer.value())
         } else { //未收到到账信息,更新支付状态,设置5秒定时器
           ctx.timerService().registerEventTimeTimer(value.timestamp * 1000 + 5000)
           payState.update(value)
           timer.update(value.timestamp * 1000 + 5000)
         }
       }
   
       override def processElement2(value: ReceiptEvent, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#Context, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {
         //判断是否收到支付订单信息
         if (payState.value() != null) { //已收到支付订单信息,输出支付成功信息,清空状态和定时器
           out.collect((payState.value(), value))
           payState.clear()
           receiptState.clear()
           ctx.timerService().deleteEventTimeTimer(timer.value())
         } else { //未收到支付订单信息,更新支付状态,设置5秒定时器
           ctx.timerService().registerEventTimeTimer(value.timestamp * 1000 + 5000)
           receiptState.update(value)
           timer.update(value.timestamp * 1000 + 5000)
         }
   
       }
   
       override def onTimer(timestamp: Long, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#OnTimerContext, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {
         //判断是支付数据还是到账数据注册的定时器
         if (payState.value() != null) { //支付数据注册的定时器,即未匹配到到账数据,输出支付数据到侧输出流
           ctx.output(payOutputTag, payState.value())
         } else { //到账数据注册的定时器,即未匹配到支付数据,输出到账数据到侧输出流
           ctx.output(receiptOutputTag, receiptState.value())
         }
         //清空状态
         payState.clear()
         receiptState.clear()
       }
     }
     
   }
   ```



