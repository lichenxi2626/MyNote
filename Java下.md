## 第十章 Java常用类

### 10.1 字符串的使用

#### 10.1.1 String的特性

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

- String类代表字符串，Java中所有字符串的字面值都作为此类的实例实现；
- **String类是一个final类，不可被继承**；
- **String类对象字符内容存储在一个字符数组value[]中**，该数组为常量，**创建后不能更改**。 



#### 10.1.2 String的实例化过程

- 方式一：使用字面值创建

  ```java
  String str = "abc";
  //内存中直接在常量池创建字符串对象
  //str指向常量池字符串
  ```

- 方式二：使用构造器创建

  ```java
  String str = new String("abc");
  //内存在堆空间中开辟空间用于存储字符串对象，字符串对象再在常量池中创建字符串对象
  //str先指向堆空间对象，堆空间对象再指向常量池对象
  
  Person p = new Person();
  p.name = "abc";
  //p.name存储在堆空间的p对象中，直接指向常量池对象
  ```

  

#### 10.1.3 String的连接运算

```java
public void test(){
    String s1 = "java";
    String s2 = "hadoop";

    String s3 = "javahadoop";
    String s4 = "java" + "hadoop";
    String s5 = "java" + s2;
    String s6 = s1 + "hadoop";
    String s7 = s1 + s2;

    String s8 = s7.intern();
    String s9 = s5.intern();

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//false
    System.out.println(s3 == s6);//false
    System.out.println(s3 == s7);//false
    System.out.println(s5 == s6);//false
    System.out.println(s5 == s7);//false

    System.out.println(s3 == s8);//true
    System.out.println(s3 == s9);//true
}
```

- **常量与常量的连接运算结果直接存储在常量池**，且常量池中不会存在相同内容的常量；
- **含有变量的连接运算结果存储在堆中**；
- 如果连接运算的结果调用了intern()方法，则返回结果存储在常量池中。



#### 10.1.4 String类的常用方法

- String类中的方法

| **方法**                                                     | **说明**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **int length()**                                             | **返回字符串长度**                                           |
| **char charAt(int index)**                                   | 返回索引处的字符                                             |
| boolean isEmpty()                                            | 判断字符串长度是否为0，true-长度为0，flase-长度不为0         |
| String toLowerCase()                                         | 返回将所有大写字符转为小写字符后的字符串                     |
| String toUpperCase()                                         | 返回将所有小写字符转为大写字符后的字符串                     |
| String trim()                                                | 返回将前导空白和尾部空白去除后的字符串                       |
| **boolean equals(Object obj)**                               | 比较字符串内容是否相同                                       |
| boolean equalsIgnoreCase(String str)                         | 比较字符串内容是否相同（忽略大小写）                         |
| **String concat(String str)**                                | 返回当前字符串尾连接指定字符串后的字符串                     |
| **int compareTo(String str)**                                | 比较两个字符串大小（从两字符串的首字符开始逐字计算编码值差，若全部相等则返回0，若出现不等则返回第一次出现不等的差值） |
| **String substring(int beginIndex)**                         | 返回一个当前字符串从beginIndex索引位置起截取到结尾的字符串   |
| String substring(int beginIndex, int endIndex)               | 返回一个当前字符串从beginIndax索引位置起截取到endIndex索引位置（不包含endIndex）的字符串 |
| boolean endWith(String suffix)                               | 判断字符串是否以suffix结尾                                   |
| boolean startWith(String prefix)                             | 判断字符串是否以prefix开始                                   |
| boolean startWith(String prefix, int toffset)                | 判断字符串在toffset索引位置是否以prefix开始                  |
| boolean contains(CharSeqquence s)                            | 当且仅当字符串包含指定的char值序列时，返回true               |
| **int indexOf(String str)**                                  | 返回指定字符串在字符串中第一次出现时首字符的索引位置，若未找到返回-1 |
| int indexOf(String str, int  fromIndex)                      | 返回从fromIndex索引位置起，指定字符串在字符串中第一次出现时首字母的索引位置，若未找到返回-1 |
| **int lastIndexOf(String str)**                              | 返回指定字符串在字符串中最后一次出现时首字符的索引位置，若未找到返回-1 |
| int lastIndexOf(String str, int fromIndex)                   | 返回从fromIndex索引位置起向前，指定字符串在字符串第一次出现时首字符的索引位置，若未找到返回-1 |
| **String replace(char oldChar, char newChar)**               | 将字符串中所有的oldchar替换为newchar后返回                   |
| String replace(CharSequence target, CharSequence replacement) | 使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串 |

- **String与char[]的相互转换**
  - 使用String类的构造器将char[]转换为String
    - String(char[])：使用char[]中的全部字符创建字符串
    - String(char[], int offset, int length)：使用char[]中的部分字符创建字符串
  - 使用String类的方法将String转换为char[]
    - public char[] toCharArray()：返回字符串所有字符组成的char[]
    - public void getChars(int srcBegin, int srcEnd, char[] dst)：返回指定起始、结束索引范围内的字符串组成的char[]
- String与byte[]的相互转换
  - 解码：使用String类的构造器将byte[]转换为String
    - String(byte[])：按照默认字符集对byte[]解码，转换为String；
    - String(byte[], "UTF-8")：按照指定字符集对byte[]解码，转换为String；
      - ASKII：使用1字节存储；
      - UTF-8：兼容ASKII部分使用1字节存储，汉字使用3字节存储；
      - GBK/ANSI：兼容ASKII部分使用1字节存储，汉字使用2字节存储。
  - 编码：使用String类方法将String转换为byte[]
    - public byte[] getBytes()：返回字符串按照默认字符集编码后的byte[]
    - public byte[] getBytes(String charsetName)：返回字符串按照指定字符集编码后的byte[]



#### 10.1.5 StringBuffer和StringBuilder类

- StringBuffer类和StringBuilder类的构造器相同

  ```java
  //StringBuffer类的对象必须通过构造器创建
  StringBuffer sb1 = new StringBuffer();//初始化一个长度为16的字符串缓冲区
  StringBuffer sb2 = new StringBuffer(int size);//初始化一个指定长度的字符串缓冲区
  StringBuffer sb3 = new StringBuffer(String str);//初始化指定字面值的字符串
  ```

- String/StringBuffer/StringBuilder三者对比

  - String：字符序列不可变，底层使用char[]存储；
  - StringBuffer：字符序列可变，线程安全（方法使用synchronized同步），效率低，底层使用char[]存储；
  - StringBuilder：字符序列可变，线程不安全，效率高，底层使用char[]存储；
  - jdk9开始，三者底层均改为使用byte[]+字符集存储，节省内存空间。
  - 三者执行效率：StringBuilder >  StringBuffer >> String

- StringBuffer和StringBuilder类的常用方法

  | 方法                                    | 说明                           |
  | --------------------------------------- | ------------------------------ |
  | append(xxx)                             | 拼接字符串                     |
  | delete(int start, int end)              | 删除指定起始位置的字符         |
  | replace(int start, int end, String str) | 替换指定起始位置的字符串       |
  | insert(int offset, xxx)                 | 在指定位置插入xxx              |
  | reverse()                               | 翻转字符串                     |
  | int indexOf(String str)                 | 返回指定字符串第一次出现的索引 |
  | setCharAt(int n, char ch)               | 替换指定位置的字符             |

  - String中的大部分方法在StringBuffer类中也能使用
  
- **StringBuffer和StringBuilder中没有重写equals()方法，若需要对比字符串内容，需要使用toString()转型后再比**。



### 10.2 比较器

- 实现比较器的两种方法
  - 实现Comparable接口
  - 实现Comparator接口

#### 10.2.1 Comparable自然排序

- **原理：Arrays.sort()方法通过调用Comparable实现类重写的CompareTo()方法对实现类对象进行排序。**

- 使用步骤

  ```java
  //1.定义Comparable实现类的子类
  public class Computer implements Comparable{
  
      private String logo;
      private double price;
  
      public Computer(String logo, double price) {
          super();
          this.logo = logo;
          this.price = price;
      }
  
      public String getLogo() {
          return logo;
      }
  
      public double getPrice() {
          return price;
      }
  
      //2.重写compareTo()方法，
      //当前对象大--返回正数
      //当前对象小--返回负数
      //两对象相等--返回0
      @Override
      public int compareTo(Object o) {
          if (o instanceof Computer) {
              Computer computer = (Computer)o;
              return Double.compare(this.price, computer.price);
          }else {
              throw new RuntimeException("形参数据类型异常");
          }
  
      }
  }
  
  //3.调用Arrays.sort()方法对实现类的实例数组排序
  Arrays.sort(arr);
  ```



#### 10.2.2 Comparator定制排序

- **原理：Arrays.sort(Object[] o, Comparator cmp)方法接收数组和Comparator接口实现类对象的重写的compare()方法，按照方法要求对接收数组进行排序。**

- 使用步骤

  ```java
  //1.新建Comparator接口实现类的对象,并重写compare()方法，返回值规则与CompareTo()相同
  Comparator cmp = new Comparator() {
  
      @Override
      public int compare(Object o1, Object o2) {
          if (o1 instanceof Mouse && o2 instanceof Mouse) {
              Mouse m1 = (Mouse)o1;
              Mouse m2 = (Mouse)o2;
              if (Double.compare(m1.getPrice(), m2.getPrice()) != 0) {
                  return Double.compare(m2.getPrice(), m1.getPrice());
              }else {
                  return m1.getLogo().compareTo(m2.getLogo());
              }
          }else {
              throw new RuntimeException("数据类型不匹配");
          }
      }
  
  };
  
  //2.使用Arrays.sort(Object[] o, Comparator cmp)方法对数组进行排序
  Arrays.sort(arr, cmp);
  ```

- Comparable与Comparator的区别

  - Comparable接口将排序规则写在排序对象所属类中，属于一劳永逸的方式；
  - Comparator接口将排序规则写接口实现类中，属于一种临时调用的方式。
  
- **当Comparable与Comparator同时使用时，默认按照Comparator的规则排序。**



### 10.3 JDK8之前的日期、时间API

- 10.3.1 System静态方法
  - java.lang.System类中的静态方法currentTimeMillis()返回当前时间与1970年1月1日0时0分0秒之间的毫秒数，数据类型为long。



- Date类

  - java.util.Date类

    - 构造器

    ```java
    Date()//获取本地当前时间
    Date(long date)//形参为1970年1月1日0时0分0秒至指定对象时间的毫秒数
    ```

    - 方法

    ```java
    long getTime()//返回自1970年1月1日0时0分0秒到Date对象时间的毫秒数
    String toString()//返回格式化后的时间信息
    ```

  - java.sql.Date类

    - 构造器

    ```java
    Date(long date)//形参为1970年1月1日0时0分0秒至指定对象时间的毫秒数
    ```

    - 方法（同上）

    

- SimpleDateFormat类：格式化日期类

  - 构造器

    ```java
    SimpleDateFormat()//使用默认方式格式化对象日期格式，输出示例：20-3-13 下午4:45
    SimpleDateFormat(String pattern)//使用指定方式格式化对象日期格式
    ```

  - 方法

    ```java
    String format(Date date)//返回符合对象格式的日期字符串
    Date parse(String source)//返回字符串描述时间对应Date对象，注意形参的格式必须与调用对象设置的格式一致
    ```

    

- Calendar类

  - Calendar类是一个抽象类，要获取实例对象需要调用使用方法调用其子类的构造器

    ```java
    Calendar calendar = Calendar.getInstance();//默认生成当前日期的日历对象
    ```

  - Calendar中的方法

    ```java
    //get()
    System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
    System.out.println(calendar.get(Calendar.DAY_OF_WEEK));
    System.out.println(calendar.get(Calendar.DAY_OF_YEAR));
    		
    //set()
    calendar.set(Calendar.DAY_OF_MONTH, 23);
    System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
            
    //add()
    calendar.add(Calendar.DAY_OF_MONTH, -2);
    System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
            
    //getTime():获取当前日历对象对应的日期Date对象
    Date date = calendar.getTime();
    System.out.println(date);//Sat Mar 21 17:03:56 GMT+08:00 2020
    		
    //setTime(Date d):使用指定的日期Date对象重置Calendar对象
    calendar.setTime(new Date());
    System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
    ```




### 10.4 JDK8中新日期时间API

#### 10.4.1 LocalDate、LocalTime、LocalDateTime

```java
public void test() {

    // now():获取当前时间的对象
    LocalDate localDate = LocalDate.now();
    LocalTime localTime = LocalTime.now();
    LocalDateTime localDateTime = LocalDateTime.now();

    System.out.println(localDate);// 2020-03-14
    System.out.println(localTime);// 10:23:29.084
    System.out.println(localDateTime);// 2020-03-14T10:23:29.084

    // of():获取指定时间的对象
    LocalDateTime localDateTime1 = LocalDateTime.of(2020, 3, 14, 11, 24, 34);

    // getXxx():
    int dayOfYear = localDateTime.getDayOfYear();

    // withXxx():体现了不可变性
    LocalDateTime localDateTime2 = localDateTime.withDayOfMonth(24);

    // plusXxx():
    LocalDateTime localDateTime3 = localDateTime.plusDays(3);

}
```



#### 10.4.2 Instant（时间戳）

```java
public void test() {
    // now()：获取当前时间的instant的实例
    Instant instant = Instant.now();
    System.out.println(instant);

    //设置时区
    OffsetDateTime dateTime = instant.atOffset(ZoneOffset.ofHours(8));

    //获取时间戳
    long epochMilli = instant.toEpochMilli();

    //获取指定毫秒数的instant实例
    Instant instant1 = Instant.ofEpochMilli(234325435234L);
}
```



#### 10.4.3 DateTimeFormat

```java
public void test() {
    // 方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME
    DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;    
    // 格式化:日期-->字符串
    LocalDateTime localDateTime = LocalDateTime.now();
    String str = formatter.format(localDateTime);
    // 解析：字符串 -->日期
    TemporalAccessor parse = formatter.parse("2019-02-18T15:42:18.797");

    // 方式二：本地化相关的格式。如：ofLocalizedDateTime()
    // FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT:适用于LocalDateTime
    DateTimeFormatter formatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
    // 格式化
    String str = formatter1.format(localDateTime);

    // 本地化相关的格式。如：ofLocalizedDate()
    // FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM /
    // FormatStyle.SHORT: 适用于LocalDate
    DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL);
    // 格式化
    String str3 = formatter2.format(LocalDate.now());

    //方式三：自定义的方式(关注、重点)
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy/MM/dd hh:mm:ss");
    //格式化
    String strDateTime = dateTimeFormatter.format(LocalDateTime.now());
    //解析
    TemporalAccessor accessor = dateTimeFormatter.parse("2020/03/14 10:45:24");
}
```



### 10.5 其他常用API

#### 10.5.1 System

- currentTimeMillis()：获取当前时间
- exit(0)：退出程序
- gc()：强制调用垃圾回收器

#### 10.5.2 Math

```java
abs()						绝对值
acos,asin,atan,cos,sin,tan	  三角函数
sqrt()						平方根
pow(double a,doble b)		 a的b次幂
log()    					自然对数
exp()    					e为底指数
max(double a,double b)
min(double a,double b)
random()      				返回0.0到1.0的随机数
long round(double a)    	 double型数据a转换为long型（四舍五入）
toDegrees(double angrad)     弧度—>角度
toRadians(double angdeg)     角度—>弧度
```

#### 10.5.3 BigInteger/BigDecimal

- BigInteger:可以表示任何精度的整数
- BigDecimal:可以表示任何精度的浮点型值

```java
public void testBigInteger() {
    BigInteger bi = new BigInteger("12433241123");
    BigDecimal bd1 = new BigDecimal("12435.351");
    BigDecimal bd2 = new BigDecimal("11");
    System.out.println(bi);
    System.out.println(bd.divide(bd2, BigDecimal.ROUND_HALF_UP));
    System.out.println(bd.divide(bd2, 15, BigDecimal.ROUND_HALF_UP));
}
```



## 第十一章 Java集合

### 11.1 Java集合框架概述

- 集合框架结构

  ```java
  |-----Collection:存储一个一个的数据
  	|-----List:存储有序的、可以重复的数据: 替换数组，"动态数组"
      	|-----ArrayList/LinkedList/Vector
  	|-----Set:存储无序的、不可重复的数据
   		|-----HashSet
          	|-----LinkedHashSet
          |-----TreeSet
  |-----Map:存储一对一对的数据（key-value)
 	|-----HashMap/LinkedHashMap/TreeMap/Hashtable/Properties
  ```
  
  

### 11.2 Collection接口方法

```java
public void test(){

    //直接调用实现类构造器创建
    Collection c1 = new ArrayList();

    //1. add(Object obj):添加元素obj到当前集合中
    c1.add(123);//自动装箱
    c1.add("AA");
    c1.add(new Date(234234324L));
    c1.add("BB");

    //2.size():获取集合中元素的个数
    System.out.println(c1.size());

    //3.addAll(Collection coll):将coll集合中的元素添加到当前集合中。
    Collection c2 = new ArrayList();
    c1.addAll(c2);

    //4.clear():清空当前集合
    c1.clear();

    //5.isEmpty():判断当前集合是否为空。
    c1.isEmpty();

    //6.contains(Object obj):判断当前集合中是否包含obj元素
    //需要调用obj对象所在类的equals(Object o)
    c1.contains(new String("AA"));

    //7.containsAll(Collection coll):判断当前集合中是否包含coll集合中的所有元素
    Collection c2 = new ArrayList();
    c2.add(1234);
    c2.add(new String("BB"));
    System.out.println(c1.containsAll(c2));

    //8.remove(Object obj):删除当前集合中首次出现的obj元素
    c1.remove(1234);

    //9.removeAll(Collection coll):差集：从当前集合中移除其与coll集合共有的元素
    c1.removeAll(c2);

    //10.retainAll(Collection coll):交集：获取当期集合与coll集合共有的元素
    c1.retainAll(c2);

    //11. equals(Object obj):判断当前集合与obj元素是否相等
    //要想返回true，要求形参obj也是一个同类型的集合对象，同时集合元素都相同。（如果是List的话，要求顺序也相同）
    System.out.println(c1.equals(c2));

    //12.hashCode():返回当前集合对象的哈希值
    System.out.println(c1.hashCode());

    //13.toArray():将集合转换为数组
    Object[] arr = c1.toArray();

    //14.Arrays.asList():将数组转换为集合
    String[] arr1 = {"AA","CC","MM","GG"};
    List list = Arrays.asList(arr1);
}
```



### 11.3 Iterator迭代器接口

- Iterator迭代器使用步骤

  ```java
  //1.调用集合对象的iterator()方法创建Iterator对象
  Iterator iterator = c1.iterator();
  //2.使用hasNext()方法判断集合是否还有下个元素
  while(iterator.hasNext()){
      //3、调用iterator.next()方法获取集合中的下个元素，返回元素的同时下移haxNext()方法的指针
      Object obj = iterator.next();
      System.out.println(obj);
  }
  ```

- 增强for循环/foreach循环

  ```java
  //格式：for(集合元素类型 变量名 : 待遍历的集合对象的引用)
  for(Object obj : c1){		
  	System.out.println(obj);
  }
  ```

  - 集合c1中的每个元素依次赋给obj后打印；
  - 数组也可以使用foreach循环。



### 11.4 Collection接口实现类：List接口

#### 11.4.1 List接口中的常用方法

```java
void add(int index, Object ele):在index位置插入ele元素
boolean addAll(int index, Collection eles):从index位置开始将eles中的所有元素添加进来
Object get(int index):获取指定index位置的元素
int indexOf(Object obj):返回obj在集合中首次出现的位置
int lastIndexOf(Object obj):返回obj在当前集合中末次出现的位置
Object remove(int index):移除指定index位置的元素，并返回此元素
Object set(int index, Object ele):设置指定index位置的元素为ele
List subList(int fromIndex, int toIndex):返回从fromIndex到toIndex位置的子集合
```



#### 11.4.2 ArrayList类的源码分析

- jdk7前

  ```java
  ArrayList list = new ArrayList();
  //初始化底层的elementDate的Object[]数组，长度为10.
  list.add(123);
  //elementDate[0] = new Integer(123);
  //若元素个数超过10，自动扩容，每次扩容后为扩容前的1.5倍
  ```

- jdk8后

  ```java
  ArrayList list = new ArrayList();
  //初始化底层的elementDate的Object[]数组为{}.
  list.add(123);
  //先创建长度为10的数组，再赋值elementDate[0] = new Integer(123);
  //若元素个数超过10，自动扩容，每次扩容后为扩容前的1.5倍
  ```



#### 11.4.3 Vector类的源码分析

- 各jdk版本相同

  ```java
  Vector v = new Vector();
  //初始化底层的elementDate的Object[]数组，长度为10.
  v.add(123);
  //elementDate[0] = new Integer(123);
  //若元素个数超过10，自动扩容，每次扩容后为扩容前的2倍
  ```



#### 11.4.4 LinkedList类的源码分析

```java
LinkedList list = new LinkedList();
list.add(123); 
//创建Node类的对象，Node类的属性包括要记录的元素和前一个、后一个Node对象
//linked类的对象在内存中不连续存储
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;// 前向指针的产生
    }
}
```



#### 11.4.5 总结

- 若确定底层数组的容量，尽量使用带参构造器创建集合，减少自动扩容操作，提升效率；

  ```java
  ArrayList list = new ArrayList(int intialCapacity);
  ```

- 数组的查找操作时间复杂度是o(1)，插入或删除操作的时间复杂度是o(n)；

  链表的查找操作时间复杂度是o(n)，插入或删除操作的时间复杂度是o(1)；



### 11.5 Collection实现类：Set接口

#### 11.5.1 Set接口的特性和方法	

- Set接口中声明的方法都是Collection接口声明过的。HashSet能使用的就是Collection中定义的方法。
- 无序性：元素的存储位置参考hashCode值，不是有序的；
- 不可重复性：根据hashcode和equals方法判断数据是否重复，若重复则不会添加。



#### 11.5.2 HashSet类的内存解析

- **HashSet底层使用数组+链表+红黑树存储**，数组的初始容量为16，当使用率超过0.75，扩容为原来的2倍；

- **必须重写对象所属的类的hashCode()和equals()方法，保证其不可重复性，相等的对象必须具有相同的散列码**；

- 添加元素过程

  ```
  1.将元素e1添加到HashSet中,首先调用e1所在类的hashCode(),获取e1对象的哈希值。
  2.此哈希值,经过某种算法以后,获取其在HashSet底层数组中的存放位置。
  3.如果此位置上,没有其他任何元素，则e1添加成功(以数组形式添加);
    如果此位置上,已经存在某个或某几个元素e2,则继续比较：
  	4.比较e1和e2的哈希值,如果两个哈希值不相同,则e1添加成功(以链表形式添加)
  	  比较e1和e2的哈希值,如果两个哈希值相同,则调用e1所在类的equals()方法继续表
  		5.equals()方法返回false,则e1添加成功(以链表形式添加)
  		  equals()方法返回true,则判断为重复数据,e1添加失败
  
  jdk7中，以链表形式添加的元素直接添加到数组中并指向原有元素
  jdk8中，以链表形式添加的元素添加到链表末尾并被原有元素指向———"七上八下"
  ```

  ![image-20200316215736909](java下.assets/image-20200316215736909.png)



####  11.5.2 LinkedHashSet类

- LinkedHashSet是HashSet的子类，区别是增加了用于双向链表维护元素的次序，使得元素可以按照插入顺序输出。
- LinkedHashSet的插入性能略逊于HashSet，迭代访问性能好于HashSet。



#### 11.5.3 TreeSet类

- 底层使用红黑树结构存储数据，可以保证元素处于排序状态；

- TreeSet在创建时默认采用自然排序，但也可通过带参构造器实现定制排序，当同时实现两种排序方式时，默认采用定制排序规则。

  ```java
  TreeSet set = new TreeSet();//使用对象的compareTo()方法排序
  TreeSet set = new TreeSet(cmp);//使用cmp中重写的compare()方法排序
  ```

- **若TreeSet创建时未指定compare()方法，则未重写comparTo()方法的类的对象不能添加到TreeSet集合，否则报Error**；

- **因为TreeSet的元素在添加时就会排序，因此添加的多个对象的数据类型必须一致，否则报数据类型转换异常**。



### 11.6 Map的实现类

- Map的架构

  ```java
  |-----Map:存储一对一对的数据（key-value)
  	|-----HashMap:主要实现类；线程不安全，效率高，可以存储null的key和value
  		|-----LinkedHashMap:HashMap的子类，在HashMap结构的基础上，给前后添加的key-value额外添加了一对指针，记录添加的先后顺序。
  	|-----TreeMap:可以按照key-value中的key的大小实现排序遍历。底层使用红黑树实现的
  	|-----Hashtable:古老实现类；线程安全的，效率低，不可以存储null的key或value
  		|-----Properties:Hashtable的子类，key和value都是String类型，常用来处理属性文件
  ```



#### 11.6.1 HashMap类

- 元素特点
  - key：不可重复、无序的数据，使用HashSet存储；
  - value：可重复，无序的数据，使用Collection存储；
  - key指向value构成键值对，一对key-value构成一个Entry（JDK8后改为Node）。
  
- **底层实现原理**

  ```java
  1.存储结构
  	JDK7:数组+链表
  	JDK8:数组+链表+红黑树
  
  2.初始化HashMap对象
  	JDK7:内存中创建长度为16的Entry[]数组，Entry为HashMap内部类，实现了Map中的Entry内部接口
  	JDK8:内存中创建空的Node[]数组，Node为HashMap内部类，实现了Map中的Entry内部接口
  	
  3.为HashMap对象添加首个元素
  	JDK7:调用key所在类的hashcode方法，经过计算确认插入索引位置，插入键值对
  	JDK8:先创建长度为16的table,之后同上
  	
  4.对已有一定数量元素的HashMap对象再添加元素
  	JDK7:根据key的hashcode值确定索引位置
  		 |--若索引位置为null，直接将元素添加至索引位置
  		 |--若索引位置已有元素，则比较二者hashcode值
  		    |--若hashcode值不等，继续比较链表结构下一元素，以此类推
  		    |--若hashcode值相等，调用key所在类的equals()比较数据值
  		       |--若equals结果为false，继续比较下一元素，以此类推
  		       |--若equals结果为true，将添加元素的value赋给原有元素，完成替换
  		 	   |--当链表中元素都不相同，将新元素装入数组索引位置并指向原有元素
  	JDK8:最后一步当链表中元素都不相同，将新元素装入链表末尾，并被上一元素指向
  	
  5.HashMap的扩容
  当集合中的元素总数(数组+链表)达到临界值(当前数组长度*默认的加载因子0.75f)时，再添加元素会触发resize()自动扩容，将数组长度扩容为原来的2倍，并将原有元素重新添加到新集合中替换原有集合。
  --加载因子越小，效率越高，内存占用越大
  --加载因子越大，效率越低，内存占用越小
  
  6.链表自动转换红黑树结构
  JDk8:
  当某索引位置上的元素超过8，且数组长度不足64时，会触发集合的自动扩容
  当某索引位置上的元素超过8，且数组长度已达64时，该索引的存储结构由单向索引自动转换为红黑树结构
  ```



#### 11.6.2 LinkedHashMap类

- LinkedHashMap中重写了父类的newNode()方法，使每一个新的Node对象比原来增加了双向指针，在调用put()方法添加元素时，由于多态性会直接调用子类的newNode()方法；
- LinkedhashMap对象的遍历速度要高于HashMap。



#### 11.6.3 TreeMap类

- 与TreeSet相似的，要求存放的所有键值对的key必须为同一类的对象；
- 同样的，TreeMap底层采用红黑树的结构，因此添加的对象必须实现排序功能（Comparable或Comparator）。



#### 11.6.4 Properties类

- Properties是Hashtable的子类；
- Properties中的key和value都是String型数据；
- 在开发中，多用来处理属性文件。



#### 11.6.5 Map接口的常用方法

```java
Object put(Object key,Object value):将指定key-value添加到(或修改)当前map对象中
void putAll(Map m):将m中的所有key-value对存放到当前map中
Object remove(Object key):移除指定key的key-value对，并返回value
void clear():清空当前map中的所有数据

Object get(Object key):获取指定key对应的value
boolean containsKey(Object key):是否包含指定的key
boolean containsValue(Object value):是否包含指定的value
int size():返回map中key-value对的个数
boolean isEmpty():判断当前map是否为空
boolean equals(Object obj):判断当前map和参数对象obj是否相等

Set keySet():返回所有key构成的Set集合
Collection values():返回所有value构成的Collection集合
Set entrySet():返回所有key-value对构成的Set集合
```



### 11.7 集合的工具类Collection

- 可操作的对象：List、Set、Map接口的所有实现类对象

- 常用方法

  ```java
  reverse(List):反转 List 中元素的顺序
  shuffle(List):对 List 集合元素进行随机排序
  sort(List):根据元素的自然顺序对指定 List 集合元素按升序排序
  sort(List, Comparator):根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
  swap(List,int,int):将指定 list 集合中的 i 处元素和 j 处元素进行交换
  Object max(Collection):根据元素的自然顺序，返回给定集合中的最大元素
  Object max(Collection, Comparator):根据 Comparator 指定的顺序，返回给定集合中的最大元素
  Object min(Collection)
  Object min(Collection, Comparator)
  int frequency(Collection, Object):返回指定集合中指定元素的出现次数
  void copy(List dest, List src):将src中的内容复制到dest中
  boolean replaceAll(List list, Object oldVal, Object newVal):使用新值替换 List 对象的所有旧值
  ```

  


## 第十二章 泛型

### 12.1 泛型的理解

- 泛型是JDK5中引入的概念；

- 所谓泛型，就是允许在定义类、接口时通过一个标识表示类中某个属性的类型或某个方法的返回值以及形参列表类型。这个类型将在使用时（通过这个类声明对象，或是继承或实现该类时）最终确定。



### 12.2 泛型在集合中的使用

```java
ArrayList<String> list = new ArrayList<String>();
//实际上是将String作为参数传入了ArrayList类中使用泛型修饰的方法(如add),使得list在添加数据时限制类型为String
list.add(123);//编译不通过

//JDK7后,依靠自动类型推断,可省略后尖括号中的内容
ArrayList<Integer> list = new ArrayList<>();
```

- 若集合在实例化时没有指定泛型，则默认的泛型类型为java.lang.Object类型
- 泛型参数只能使用引用数据类型，需要基本数据类型时应采用相应包装类
- 对于TreeSet集合对象可以考虑对Comparable和Comparator使用泛型



### 12.3 自定义泛型结构

#### 12.3.1 自定义泛型类、泛型接口

```java
public class Order<T>{
    private T t;

    //定义构造器中不需要使用泛型<>
    public Order(){
        t = (T)(new Object());
    }

    public T getT(){
        return T
    }
}
```

- **泛型类的对象不能使用new+构造器的格式创建**，必须使用Object()创建，再强转为泛型类对象；
- 因为泛型在泛型类创建时才确定，因此**静态方法中不能使用类的泛型**；
- 若一个类包含多个泛型参数，可以并列声明<T1,T2,T3>;
- **不同泛型参数的同一泛型类对象之间不能互相赋值**；
- 异常类不能使用泛型;
- 子类可以选择是否继承父类的泛型。



#### 12.3.2 自定义泛型方法

```java
public <E> E getE(int i. E e){
    return E;
}
```

- 泛型方法可以声明在泛型或非泛型类中，因为其泛型参数与类本身无关；
- 泛型方法可以是静态方法；
- 泛型方法的泛型参数在调用该方法时确定。



#### 12.3.3 泛型与继承

- 具有子父类关系的泛型参数分别修饰同一泛型类，该泛型类的两种引用不构成子父类关系，且无法相互转换

  ```java
  A<String> a1;
  A<object> a2;
  //A<String>，A<object>不是子父类
  ```

- 具有继承关系的两泛型类若泛型参数一致，则依然构成子父类关系

  ```java
  List<String> list1;
  ArrayList<String> list2;
  //List<String>、ArrayList<String>是子父类
  ```



#### 12.3.4 通配符的使用

- 通配符<?>
  - List<?>是List\<String>、List\<Integer>等List所有泛型类的父类；
  - List<?>中的所有元素都可以用Object类的实例通过get()方法接收；
  - 不能向List\<?>中写入除null外的任何元素，因为List<?>中的元素可能是任意子类的对象。
  
- <? extends classA>
  
  - LIst<? extends Person>中的元素可以使用任何Person类或其父类的对象通过get()接收；
  - 不能向LIst<? extends Person>中写入除null外的任何元素。
  
- <? super classA>
  
  - List<? super Person>中的元素可以使用Object类的对象通过get()接收；
  - 可以向List<? super Person>中添加任何Person类及其子类的对象元素。
  
- 通配符在参数列表中使用，代表**泛型的上限和下限**

  ```java
  public void test1(List<? extends Person> list) {}
  //所有泛型为Person及其子类类型的List可以作为参数传入方法test1,如List<Person>,List<Man>
  
  public void test2(List<? super Person> list) }{}
  //所有泛型为Person及其父类类型的List可以作为参数传入方法test2,如List<Person>,List<Object>
  ```

  



## 第十三章 IO流

### 13.1  File类的使用

#### 13.1.1  File类的介绍

- File类及所有的io相关类都声明在java.io包下；
- File类的一个对象可以表示一个文件或一个目录；
- File类的对象通常是作为IO流的端点出现，常作为参数传递给IO流类的构造器。

#### 13.1.2 File类的实例化

- 使用绝对路径或相对路径创建；

  ```java
  File file1 = new file("D:\\io\\abc.txt");//绝对路径
  File file2 = new file("abc.txt");//相对路径
  //main方法中创建file对象时的相对路径从当前工程下开始
  //@Test单元测试方法中创建file对象时的相对路径从当前Module下开始
  ```

- 使用父路径和子路径创建；

  ```java
  File file1 = new file("D:\\io", "abc.txt");
  ```

- 使用父File对象和子路径创建。

  ```java
  File file1 = new File("D:\\io");
  File file3 = new File(file1, "abc.txt")
  ```

#### 13.1.3 File类的常用方法

- File类的所有方法都不涉及对文件内容的读写

```java
public String getAbsolutePath()：获取绝对路径
public File getAbsoluteFile()：获取绝对路径表示的文件
public String getPath() ：获取路径
public String getName() ：获取名称
public String getParent()：获取上层文件目录路径。若无(以相对路径创建的文件对象没有父路径)，返回null
public long length() ：获取文件长度（即：字节数），目录的长度为0。
public long lastModified() ：获取最后一次的修改时间，毫秒值
    
public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组
public boolean renameTo(File dest):把文件重命名为指定的文件路径
    
public boolean isDirectory()：判断是否是文件目录
public boolean isFile() ：判断是否是文件
public boolean exists() ：判断是否存在
public boolean canRead() ：判断是否可读
public boolean canWrite() ：判断是否可写
public boolean isHidden() ：判断是否隐藏
    
public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
public boolean mkdir() ：创建文件目录。若此文件目录存在，或此文件目录的上层目录不存在，则不创建
public boolean mkdirs() ：创建文件目录。如果上层文件目录不存在，一并创建
public boolean delete()：删除文件或者文件夹，若为文件目录，则目录下必须为空才能删除
```

 

### 13.2 IO流

- IO流的分类
  - 按照操作数据的单位不同分类
    - 字节流（8bit）
    - 字符流（16bit）
  - 按照数据的流向分类
    - 输入流（Input）
    - 输出流（Output）
  - 按照流的角色、作用分类
    - 节点流
    - 处理流
- IO流的4个抽象基类
  - InputStream：字节输入流
  - OutputStream：字节输出流
  - Reader：字符输入流
  - Writer：字符输出流



### 13.3 节点流

#### 13.3.1 FileReader和FileWriter

- FileReader

  ```java
  @Test
  public void testFileReader2() {
      FileReader fr = null;
  
      try {
          //1.创建文件对象
          File file = new File("hello.txt");
  
          //2. 创建流对象
          fr = new FileReader(file);
  
          //3. 读入数据的过程
          char[] cbuf = new char[5];
          int len;//记录每次读入到cbuf数组中的字符的个数
          while((len = fr.read(cbuf)) != -1){
              String str = new String(cbuf,0,len);
              System.out.print(str);
          }
  
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          //4.对于流的关闭操作，必须手动实现
          try {
              if (fr != null)
                  fr.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- FileWriter

  ```java
  @Test
  public void testFileWriter() {
      FileWriter fw = null;
      try {
          //1. 创建输出到的File对象
          File file1 = new File("info.txt");
          //2. 创建输出流
          fw = new FileWriter(file1,true);
          //3. 输出数据的过程
          fw.write("I love You!\n".toCharArray());
          fw.write("You love him".toCharArray());
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          //4. 手动的关闭资源
          try {
              if(fw != null)
                  fw.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- 结论

  - 输入节点流传入的文件对象必须存在，否则抛出异常FileNotFoundException；

  - 输出节点流传出的文件可以不存在，会自动创建；

  - 若数据节点流传入的文件存在，可以通过传参决定写入方式

    ```java
    new FileWrite(file, true);//清空原文件后写入
    new FileWrite(file, false);//在原文件内容末尾追加写入
    ```

  - **IO流资源在使用后必须调用close()方法关闭**；

  - FileReader和FileWriter只能处理文本文件，对于非文本文件的处理需要使用FileInputStream和FileOutputStream

#### 13.3.2 FileInputStream和FileOutputStream

- **使用方法与字符节点流基本一致，相应read()和write()方法的形参需改为字节数组byte[]。**



### 13.4 缓冲流

- 为了提高数据的读写速度，Java提供了有缓冲功能的流类，使用缓冲流时，内存会创建一个8Kb的缓冲区，相当于加大了一次读取/写入的数据量。

- **缓冲流类为相应字符流类的子类**；

- BufferedReader和BufferedWriter

  ```java
  @Test
  public void testBufferedReaderWriter() {
      BufferedReader br = null;
      BufferedWriter bw = null;
      try {
          //1. 造文件、造流，将字节流对象作为形参传给缓冲流
          br = new BufferedReader(new FileReader(new File("dbcp.txt")));
          bw = new BufferedWriter(new FileWriter(new File("dbcp-1.txt")));
          //2. 读写文件的细节
          String data;
          //缓冲类新增的readLine方法，可以直接读取一整行的文件字符（不包括换行符）
          while((data = br.readLine()) != null){
              bw.write(data);
              bw.newLine();//换行方法
          }
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          //3.关闭资源，关闭缓冲流会自动关闭相应的字节流
          try {
              if (bw != null)
                  bw.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
          try {
              if (br != null)
                  br.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- BufferedInputStream和BufferedOutputStream

  - 使用方法和字符型缓冲流一致，无readLine()方法，依然需要依靠byte[]传递



### 13.5 转换流

- InputStreamReader：将输入型的字节流转换为字符流

- OutputStreamWriter：将输入型的字符流转换为字节流

- 代码实现

  ```java
  @Test
  public void testInputStreamReaderOutputStreamWriter() throws IOException {
  
      //转换流对象通过传入字节流对象创建
      //创建转换流对象时指明编码时的字符集，默认使用开发环境设置的字符集
      InputStreamReader isr = new InputStreamReader(new FileInputStream("aaa.txt"));
      OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("bbb.txt"),"gbk");
  
      char[] cbuf = new char[1024];
      int len;
      while((len = isr.read(cbuf)) != -1){
          osw.write(cbuf,0,len);
      }
  
      osw.close();
      isr.close();
  }
  ```



### 13.6 对象流

- 使用对象流可以把对象及其具有的属性以序列化的方式写入磁盘，同样的可以从磁盘通过反序列化的方式读取并对象，实现了内存与磁盘数据的交互；

- ObjectOutputStream序列化

  ```java
  @Test
  public void testObjectOutputStream() throws Exception {
  
      //1.创建文件和流
      ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.dat"));
  
      //操作基本数据类型
      oos.writeByte();
      oos.writeBoolean();
  
      //2.操作对象
      oos.writeObject(new String("Tom"));
      oos.flush();//刷新
      oos.writeObject(new String("王辰硕"));
      oos.flush();//刷新
  
      //3.关闭资源
      oos.close();
  }
  ```

- ObjectInputStream反序列化

  ```java
  @Test
  public void testObjectInputStream() throws IOException, ClassNotFoundException {
  
      ObjectInputStream ois = new ObjectInputStream(new FileInputStream("object.dat"));
  
      String s1 = (String) ois.readObject();
      System.out.println(s1);
  
      String s2 = (String) ois.readObject();
      System.out.println(s2);
  
      ois.close();
  }
  ```

- 自定义类的序列化要求

  - 序列化的类的对象必须实现Serialized接口；
  - （可选）序列化的类必须声明全局常量SerialVersionUID，用于唯一标识该类；
  - 序列化的类的对象的属性所在类也必须实现Serialized接口。
  - 注：不能序列化使用 static 和 transient 关键字修饰的类的属性。



### 13.7 其他处理流

#### 13.7.1 标准的输入输出流

- 标准的输入流：System.in

- 标准的输出流：System.out

- 代码实现

  ```java
  @Test
  public void test() {
      System.out.println("请输入信息(退出输入e或exit):");
      // 把"标准"输入流(键盘输入)这个字节流包装成字符流,再包装成缓冲流
      BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
      String s = null;
      try {
          while ((s = br.readLine()) != null) { // 读取用户输入的一行数据 --> 阻塞程序
              if ("e".equalsIgnoreCase(s) || "exit".equalsIgnoreCase(s)) {
                  System.out.println("安全退出!!");
                  break;
              }
              // 将读取到的整行字符串转成大写输出
              System.out.println("-->:" + s.toUpperCase());
              System.out.println("继续输入信息");
          }
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          try {
              if (br != null) {
                  br.close(); // 关闭过滤流时,会自动关闭它包装的底层节点流
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```



#### 13.7.2 打印流

- printStream和printWriter

  - 提供一系列重载的print()和println()方法，用于不同数据类型的输出；
  - System.out返回的是printStream的实例

- 代码实现

  ```java
  @Test
  public void test() {
      PrintStream ps = null;
      try {
          FileOutputStream fos = new FileOutputStream(new File("D:\\IO\\text.txt"));
          // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
          ps = new PrintStream(fos, true);
          if (ps != null) {// 把标准输出流(控制台输出)改成文件
              System.setOut(ps);
          }
          for (int i = 0; i <= 255; i++) { // 输出ASCII字符
              System.out.print((char) i);
              if (i % 50 == 0) { // 每50个数据一行
                  System.out.println(); // 换行
              }
          }
      } catch (FileNotFoundException e) {
          e.printStackTrace();
      } finally {
          if (ps != null) {
              ps.close();
          }
      }
  }
  ```

  

#### 13.7.3 随机存取文件流

- RandomAccessFile类的使用

  - 直接继承于java.lang.Object类；

  - 实现了DataInput和DataOutput接口，既可以作为输入流，也可以作为输出流；

  - 如果输出的目标文件存在，则会对文件中的具体目标内容进行覆盖；

  - 可以使用seek()从文件的具体指定位置开始写入数据

    ```java
    raf.seek(5);
    raf.write("xyz".getBytes());
    //从索引位置5开始替换写入xyz
    ```

- 代码实现

  ```java
  @Test
  public void test() {
      RandomAccessFile raf1 = null;
      RandomAccessFile raf2 = null;
      try {
          raf1 = new RandomAccessFile("baby.jpg", "r");
          raf2 = new RandomAccessFile("baby1.jpg", "rw");
  
          byte[] buffer = new byte[1024];
          int len;
          while ((len = raf1.read(buffer)) != -1) {
              raf2.write(buffer, 0, len);
          }
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          try {
              if (raf2 != null)
                  raf2.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
          try {
              if (raf1 != null)
                  raf1.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```



### 13.8 NIO

- Java NIO（New IO）是从jdk1.4开始引入的一套新的API，可以替代标准的IO API，NIO支持面向缓冲区的、基于通道的IO操作，将以更加高效的方式进行文件的读写操作；

- NIO相关的API

  ```java
  |-----java.nio.channels.Channel
  	|-----FileChannel:处理本地文件
  	|-----SocketChannel：TCP网络编程的客户端的Channel
  	|-----ServerSocketChannel:TCP网络编程的服务器端的Channel
  	|-----DatagramChannel：UDP网络编程中发送端和接收端的Channel
  ```

- NIO2：jdk7中对NIO进行了极大地拓展，增强了对文件处理和文件系统特性的支持，以至于我们称他们为NIO.2。

- NIO中的常用类

  - Path：替换原有的File类，表示一个文件或目录，提供了丰富的方法
  - Paths：用来实例化Path的工具类
  - Files：用来操作文件或文件目录的工具类



## 第十四章 网络编程

### 14.1 网络编程概述

- 网络通信的两大要素
  - IP和端口号：在网络中准确定位一台或多台主机
  - 网络通信协议：不同主机之间先进行可靠高效的数据传输
- TPC/IP协议模型：国际标准



### 14.2 IP和端口号

#### 14.2.1 IP

- IP：Internet上计算机的唯一标识

- IP分类

  - 按字节数分类
    - IPV4（4字节）：总数量约42亿，如192.168.0.1
    - IPV6（16字节）：用8个连续的十六进制数表示，如3ffe:3201:1401:1280:c8ff:fe4d:db39:1984
  - 按公私分类
    - 公网地址（万维网使用）
    - 私网地址（局域网使用）：范围192.168.0.0-192.168.255.255

- 可使用一个域名表示一个具体的IP地址，如：www.baidu.com

- 本地的IP地址（localhost）：127.0.0.1

- 代码中使用**InetAddress类的对象表示具体的IP地址**

  ```java
  try {
  	InetAddress inetAddress = InetAddress.getByName("192.168.23.43");//通过IP地址创建IP对象
  	InetAddress inetAddress1 = InetAddress.getByName("www.atguigu.com");//通过域名创建IP对象
  	System.out.println(inetAddress1);//输出www.atguigu.com/124.200.113.114
  
  	InetAddress localHost = InetAddress.getLocalHost();//获取本地IP地址对象
  	System.out.println(localHost);
  
  	//测试方法：
  	System.out.println(inetAddress1.getHostName());//获取域名
  	System.out.println(inetAddress1.getHostAddress());//获取IP地址
  
  } catch (UnknownHostException e) {
  	e.printStackTrace();
  }
  ```



#### 14.2.2 端口号

- 端口号标识计算机上每个正在运行的进程，规定为16位整数（0-65535）；
- 端口分类
  - 公认端口：0-1023，被预定义的服务通信占用（如：HTTP占用端口80，FTP占用端口21，Telnet占用端口23）
  - 注册端口：1024-49151，分配给用户进程或应用程序（如：Tomcat占用端口8080，MySQL占用端口3306，Oracle占用端口1521等）
  - 动态/私有端口：49152-65535
- **IP地址+端口号=Socket**



### 14.3 TCP和UDP编程

#### 14.3.1 概述

- TCP协议
  - 使用TCP协议前，必须先建立TCP连接，形成数据传输通道；
  - 传输前，采用**三次握手**方式，点对点通信，是可靠的；
  - TCP协议进行通信的两个应用进程：客户端、服务端；
  - 在连接中可进行**大数据量的传输**；
  - 传输完毕，需要**释放已建立的连接，效率低**。
- UDP协议
  - 间数据、源、目的封装成数据包，**不需要建立连接**；
  - 每个数据包的大小限制在**64Kb内**；
  - 发送无需确认对方状态，也无需收到回复，**不可靠**；
  - 可以广播发送；
  - 发送完毕，**无需释放资源，效率高**。

#### 14.3.2 TCP网络编程

- 代码实现1：客户端发送内容给服务端，服务端输出内容到控制台

  ```java
  public class TCPTest1 {
  
      @Test//客户端
      public void client() throws Exception{
          //1.使用服务端IP地址+端口号实例化Socket对象
          Socket socket = new Socket("127.0.0.1",9526);
          //2.获取输出流，创建输出流
          OutputStream os = socket.getOutputStream();
          OutputStreamWriter osw = new OutputStreamWriter(os);
          //3.输出信息
          osw.write("你好，我是客户端".toCharArray());
          //4.关闭流
          osw.close();
          socket.close();
      }
  
      @Test//服务端
      public void sever() throws Exception{
          //1.使用端口号实例化ServerSocket对象
          ServerSocket serverSocket = new ServerSocket(9526);
          //2.使用serverSocket类对象的accept()实例化Socket对象
          Socket socket = serverSocket.accept();
          //3.获取输入流
          InputStream is = socket.getInputStream();
          InputStreamReader isr = new InputStreamReader(is);
          //4.创建字符输出流对象接收信息并打印
          CharArrayWriter caw = new CharArrayWriter();
          int len = 0;
          char[] chars = new char[1024];
          while((len = isr.read(chars)) != -1){
              caw.write(chars, 0, len);
          }
          System.out.println(caw.toString());
          //5.关闭流
          caw.close();
          isr.close();
          //服务器端流实际使用时一般不关闭
          socket.close();
          serverSocket.close();
      }
  
  }
  ```

- 代码实现2：客户端将本地文件发送给服务端，服务端接收后写入本地文件

  ``` java
  public class TCPTest2 {
  
      @Test
      public void client() throws Exception {
          
          Socket socket = new Socket("127.0.0.1",9526);
  
          FileInputStream fi = new FileInputStream("src/hello.txt");
          OutputStream os = socket.getOutputStream();
  
          byte[] bt = new byte[1024];
          int len = 0;
          while((len = fi.read(bt)) != -1){
              os.write(bt,0,len);
          }
  
          os.close();
          fi.close();
          socket.close();
  
      }
  
      @Test
      public void server() throws Exception{
          
          ServerSocket serverSocket = new ServerSocket(9526);
          Socket socket = serverSocket.accept();
  
          InputStream is = socket.getInputStream();
          FileOutputStream fo = new FileOutputStream("hello1.txt");
  
          byte[] bt = new byte[1024];
          int len = 0;
          while((len = is.read(bt)) != -1){
              fo.write(bt,0,len);
          }
  
          fo.close();
          is.close();
          socket.close();
          serverSocket.close();
      }
  
  }
  ```

- 代码实现3：客户端发送文件给服务器，服务器保存到本地后返回成功信息给客户端，客户端输出返回信息

  ``` java
  public class TCPTest3 {
      @Test
      public void client() throws Exception{
          //客户端从本地读取图片并发送
          Socket socket = new Socket("127.0.0.1", 9526);
          OutputStream os = socket.getOutputStream();
          FileInputStream fi = new FileInputStream("IU1.jpeg");
          byte[] bt = new byte[1024];
          int len = 0;
          while((len = fi.read(bt)) != -1){
              os.write(bt, 0, len);
          }
          socket.shutdownOutput();//必须手动调用shutdownOutput()结束传输
          //客户端接收服务端信息
          InputStream is = socket.getInputStream();
          ByteArrayOutputStream bs = new ByteArrayOutputStream();
          while((len = is.read(bt)) != -1){
              bs.write(bt, 0, len);
          }
          System.out.println(bs.toString());
          //关闭流
          bs.close();
          is.close();
          fi.close();
          os.close();
          socket.close();
      }
  
      @Test
      public void server() throws Exception{
          //服务端接收图片并保存到本地
          ServerSocket serverSocket = new ServerSocket(9526);
          Socket socket = serverSocket.accept();
          InputStream is = socket.getInputStream();
          FileOutputStream fo = new FileOutputStream("IU2.jpeg");
          byte[] bt = new byte[1024];
          int len = 0;
          while((len = is.read(bt)) != -1){
              fo.write(bt, 0, len);
          }
          //服务端返回接收成功信息
          OutputStream os = socket.getOutputStream();
          os.write("服务端已收到".getBytes());
          //关闭流
          os.close();
          fo.close();
          is.close();
          socket.close();
          serverSocket.close();
      }
  }
  ```

#### 14.3.3 UDP网络编程

```java
public class UDPTest {

    @Test
    public void sender() throws IOException {
        //1.创建DatagramSocket流对象
        DatagramSocket socket = new DatagramSocket();
        //2.创建DatagramPacket类的的数据包对象，并传入数据、服务端的IP地址和端口号
        byte[] buffer = "hello".getBytes();
        DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length,"127.0.0.1",9090);
        //3.使用send()方法发送数据
        socket.send(packet);
        //4.关闭流
        socket.close();
    }

    @Test
    public void receiver() throws IOException {
        //1.使用端口号创建DatagramSocket流对象
        DatagramSocket socket = new DatagramSocket(9090);
        //2.创建DatagramPacjet类的的数据包对象，准备接收数据
        byte[] buffer = new byte[1024];
        DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
        //3.使用receive()方法接收数据并输出到控制台
        socket.receive(packet);
        String str = new String(packet.getData(),0,packet.getLength());
        System.out.println(str);
        //4.关闭流
        socket.close();
    }

}
```



14.4 URL编程

- URL（Uniform Resource Locator）:统一资源定位符，表示Internet上某一资源的地址。

- 测试

  ```java
  try {
       URL url = new URL("http://localhost:8080/examples/playgirl.jpg?keyword=girl");
  
  	//获取该URL的协议名
  	System.out.println(url.getProtocol());//输出http
      
      //获取该URL的主机名
  	System.out.println(url.getHost());//输出localhost
      
      //获取该URL的端口号
  	System.out.println(url.getPort());//输出8080
      
      //获取该URL的文件路径
  	System.out.println(url.getPath());//输出/examples/playgirl.jpg
      
      //获取该URL的文件名
  	System.out.println(url.getFile());//输出/examples/playgirl.jpg?keyword=girl
      
      //获取该URL的查询名
  	System.out.println(url.getQuery());//输出keywork=girl
  
  } catch (MalformedURLException e) {
  	e.printStackTrace();
  }
  ```

- 下载数据

  ``` java
  public class URLTest{
      public static void main(String[] args) {
          HttpURLConnection urlConnection = null;
          InputStream is = null;
          FileOutputStream fos = null;
          try {
              //1.传入数据文件对象，创建url对象
              URL url = new URL("http://localhost:8080/examples/playgirl.jpg");
              //2.调用openConnection()方法创建urlConnection对象
              urlConnection = (HttpURLConnection) url.openConnection();
              //3.创建流
              is = urlConnection.getInputStream();
              fos = new FileOutputStream("playgirl.jpg");
              //4.读取数据并写出
              byte[] buffer = new byte[1024];
              int len;
              while ((len = is.read(buffer)) != -1) {
                  fos.write(buffer, 0, len);
              }
              System.out.println("下载成功");
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              //5.关闭流
              try {
                  if (is != null)
                      is.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
              try {
                  if (fos != null)
                      fos.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
              if (urlConnection != null)
                  urlConnection.disconnect();
          }
      }
  }
  ```



## 第十五章 Java反射机制

### 15.1 反射机制概述

- java提供了一套api（java.lang.reflect）,用于实现

  - 动态获取内存中的运行时类；
  - 动态创建运行时类的对象；
  - 动态调用运行时类中指定的结构：属性和方法。

- 反射的应用

  - 在运行时判断任意一个对象所属的类；
  - 在运行时构造任意一个类的对象；
  - 在运行时判断任意一个类所具有的成员变量和方法；
  - 在运行时获取泛型信息；
  - 在运行时调用任意一个对象的成员变量和方法；
  - 在运行时处理注解；
  - 生成动态代理。

- 相关API

  ```java
  java.lang.Class//代表一个类
  java.lang.reflect.Method//代表类的构造器
  java.lang.reflect.Field//代表类的成员变量
  java.lang.reflect.Constructor//代表类的构造器
  ```

  

### 15.2 Class类的理解和使用

#### 15.2.1 理解

- **java源程序经过javac.exe编译后，生成了一个或多个字节码文件。接着使用java.exe命令，将字节码文件代表的类（使用类的加载器）加载到内存中（方法区中缓存）。加载到内存中的类，称为运行时类。运行时类可以看作是Class类的实例**；
- 加载到内存中的一个运行时类会被缓存，在整个jvm生命周期内，若未进行方法区gc，则该运行时类有且仅有一份。

#### 15.2.2 获取Class实例的方式

- 调用类的静态属性class

  ```java
  Class clazz = Person.class;
  ```

- 调用类的对象的方法getClass()

  ```java
  Person p = new Person();
  Class clazz = p.getClass();
  ```

- **调用Class类的静态方法forName(String name) —— 体现反射动态性，常用**

  ```java
  Class clazz = Class.forName("com.atguigu.java.Person");
  ```

- 调用类的加载器（了解）

  ```java
  ClassLoader classLoader = ReflectionTest.class.getClassLoader();
  Class clazz = classLoader.loadClass("com.atguigu.java.Person");
  ```

#### 15.2.3 Class的对象范围

1. class：外部类和内部类
2. interface：接口
3. []：数组，元素和维度都相同即为同一个Class对象
4. enum：枚举类
5. annotation：注解
6. primitive type：基本数据类型
7. void



### 15.3 类的加载与ClassLoader

#### 15.3.1 类的加载过程

1. 加载环节：字节码文件加载到内存中，生成Class的实例；
2. 链接环节：
   - 验证：字节码是否合规、安全
   - **准备：给当前类的static属性默认初始化赋值**
   - 解析：将符号引用转化为直接引用
3. 初始化：为当前类的static属性进行显式初始化和静态代码块初始化，在<clinit>方法中执行。

#### 15.3.2 类的加载器

- 启动类加载器（Bootstap Classloader）
  - 使用C++编写，是JVM自带的类加载器，负责Java平台核心库，用来装载核心类库，无法直接获取该加载器；
- 扩展类加载器（Extension Classloader）
  - 负责jre/lib/ext目录下的jar包或指定目录下的har包装入工作库；
- 系统类加载器（System Classloader）
  - 负责java -classpath或指定目录下的类与jar包装入的工作，是最常用的加载器。

``` java
public static void main(String[] args) {
        //获取系统类加载器（应用程序类加载器）
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
        //获取扩展类加载器
        ClassLoader extentionClassLoader = systemClassLoader.getParent();
        System.out.println(extentionClassLoader);//sun.misc.Launcher$ExtClassLoader@6d6f6e28
        //获取引导类加载器：失败，返回值为null
        ClassLoader bootstrapClassLoader = extentionClassLoader.getParent();
        System.out.println(bootstrapClassLoader);//null

        //用户自定义类默认使用的是系统类加载器
        ClassLoader loader1 = Person.class.getClassLoader();//
        System.out.println(loader1);//sun.misc.Launcher$AppClassLoader@18b4aac2
        //java核心类库使用引导类加载器
        ClassLoader loader2 = String.class.getClassLoader();
        System.out.println(loader2);//null
    }
```

- JVM加载类的双亲委派机制
  - 类加载器的父子关系：启动类加载器 > 扩展类加载器 > 应用类加载器
  - 过程
    1. 类加载器收到加载类请求
    2. 类加载器将请求发送给父加载器，一直向上发送，直到发送给启动类加载器
    3. 若启动类加载器成功加载类，则加载结束，否则会向下通知子加载器加载，直到类成功被加载
    4. 若所有加载器都加载失败，则抛出异常
  - 作用
    1. 加载前检查类是否已加载，减少重复的加载工作
    2. 防止用户自定义的同名类篡改java的核心类（如Object，String），导致程序崩溃

#### 15.3.3 读取配置文件的内容

- **方式1：使用输入流读取配置文件**

```java
@Test
public void test(){
    FileInputStream f = null;
    try {
        //1.实例化Properties对象，用于加载配置文件内容
        Properties properties = new Properties();
        //2.使用配置文件对象创建流，用于传输配置文件内容
        f = new FileInputStream("src/jdbc.properties");
        //3.properties对象调用load()方法加载输入流的数据
        properties.load(f);
        //4.使用getProperty()方法和key值获得value
        String name = properties.getProperty("name");
        System.out.println(name);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            //5.关闭流
            f.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- 方式2：使用类的加载器读取配置文件

``` java
@Test
public void test() throws IOException {
    //1.实例化Properties对象，用于加载配置文件内容
    Properties pros = new Properties();
    //2.使用配置文件对象和系统类加载器的getResourceAsStream()创建流，用于传输配置文件内容
    InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
    //3.properties对象调用load()方法加载输入流的数据
    pros.load(is);
    //4.使用getProperty()方法和key值获得value
    String name = pros.getProperty("name");
    System.out.println(name);
    //5.关闭流
    is.close();
}
```



### 15.4 创建运行时类的对象

- 方式1：通过Class类的newInstance()方法创建

  ```java
  //1.获取Person类的Class对象
  Class clazz = Class.forName("com.atguigu.java.Person");
  //2.使用Class类的newInstance()方法创建Person类对象
  Person p = (Person)Class.newInstance();
  //要求：Person类必须具备空参构造器，且构造器访问权限为public
  ```

- 方式2：通过Class类的getDeclaredConstructor()方法获取指定构造器对象后再调用newInstance()方法创建

  ```java
  //1.获取Person类的Class对象
  Class clazz = Class.forName("com.atguigu.java.Person");
  //2.使用Class类对象的getDeclaredConstructor()方法获取指定构造器
  Constructor con = clazz.getDeclaredConstructor();
  //3.设置构造器对象的访问权限为true
  con.setAccessable(true);
  //4.使用Constructor类对象的newInstance()方法创建Person类的对象
  Person p = (Person)con.newInstance();
  ```



### 15.5 获取运行时类的完整结构

- 获取所有接口

  ```java
  Class clazz = Person.class;
  Class[] interfaces = clazz.getInterfaces;//获取Person类实现的所有接口构成的数组
  ```

- 获取父类

  ```java
  Class clazz = Person.class;
  Class superClass = clazz.getSuperClass();//获取Person类继承的的父类的Class对象
  ```

- 获取构造器

  ```java
  Class clazz = Person.class;
  Constructor[] cons = clazz.getConstructors();//获取Person类的所有权限为public的构造器构成的数组
  Constructor[] cons = clazz.getDeclaredConstructors();//获取Person类的所有构造器构成的数组
  
  //对于Constructor类
  int mod = cons[0].getModifiers();//获取构造器的权限修饰符 public-1 protected-4 缺省-0 private-2
  String name = cons[0].getName();//获取构造器名(xxx.xxx.xxx.Person)
  Class[] paraType = cons[0].getParameterTypes();//获取构造器的形参列表数据类型构成的数组
  ```

- 获取方法

  ```java
  Class clazz = Person.class;
  Method[] methods = clazz.getMethods();//获取Person类及其父类的所有public权限的方法构成的数组(包括静态)
  Method[] methods = clazz.getDeclaredMethods();//获取Person类自身的所有方法构成的数组(包括静态)
  
  //对于Method类
  int mod = cons[0].getModifiers();//获取方法的权限修饰符 public-1 protected-4 缺省-0 private-2
  Class ruturnType = methods[0].getReturnType();//获取方法的返回值类型
  Class[] paraType = methods[0].getParameterTypes();//获取方法的形参列表数据类型构成的数组
  Class[] exceps = methods[0].getExceptionTypes();//获取方法抛出的异常类型构成的数组
  ```

- 获取属性

  ``` java
  Class clazz = Person.class;
  Field[] fields = clazz.getFields();//获取Person类及其父类的所有public权限的属性构成的数组(包括静态)
  Field[] fields = clazz.getDeclaredFields();//获取Person类自身所有的属性构成的数组(包括静态)
  
  int mod = fields[0].getModifiers();//获取属性的权限修饰符 public-1 protected-4 缺省-0 private-2
  Class type = fields[0].getType();//获取属性的数据类型
  String name = fields[0].getName();//获取Field的名称(不包含路径)
  ```
  
- 获取类所在的包

  ```java
  Class clazz = Person.class;
  Package pack = clazz.getPackage();
  ```

- 获取类的注解

  ```java
  Class clazz = Person.class;
  Annotation[] annos = clazz.getAnnotations();//获取类的所有注解构成的数组
  ```

- 获取带泛型的父类

  ``` java
  Class clazz = Person.class;
  Type genericSuperclass = clazz.getGenericSuperclass();
  //输出com.atguigu.java1.Creature<java.lang.String>
  ```

- 获取父类的泛型

  ```java 
  Class clazz = Person.class;
  Type genericSuperclass = clazz.getGenericSuperclass();
  ParameterizedType paramsType = (ParameterizedType) genericSuperclass;
  Type[] arguments = paramsType.getActualTypeArguments();//获取泛型构成的数组
  ```



### 15.6 获取运行时类的指定结构

#### 15.6.1 调用指定方法

- 格式步骤

  ``` java
  //1.调用Class类中的getDeclaredMethod(String name, Class...parameterTypes)方法获取Method类对象
  //2.调用Method对象的setAccessible(true)方法确保方法可访问
  //3.调用Method对象的invoke(Object obj, Object[] args)方法调用指定方法
  Object invoke(Object obj, Object[] args)
  	//Object 对应原方法返回值，若原方法无返回值，则返回null
  	//Object obj 传入目标类的一个对象，若原方法为静态方法，则传入null
  	//Object[] args 传入原方法的形参，若原方法形参列表为空，则不传该参数
  ```

- 代码实现

  ```java
  //前提：获取Class类和Person类实例
  Class clazz = Person.class;
  Person p1 = (Person) clazz.newInstance();
  
  //非静态方法的调用 private String showNation(String nation,int year)
  //1.通过方法名、形参列表类型获取指定的方法
  Method showNation = clazz.getDeclaredMethod("showNation", String.class, int.class);
  //2.确保方法可访问
  showNation.setAccessible(true);
  //3.使用invoke()调用此方法，注意返回值数据类型为Object
  String nation = (String)showNation.invoke(p1,"CHINA",10);
  
  //静态方法的调用 public static void showInfo()
  //1.通过方法名、形参列表类型获取指定的方法
  Method showInfo = clazz.getDeclaredMethod("showInfo");
  //2.确保方法可访问
  showInfo.setAccessible(true);
  //3.使用invoke()调用此方法，注意返回值数据类型为Object
  Object returnVal = showInfo.invoke(null);
  ```

  

#### 15.6.2 调用指定属性

- 格式步骤

  ``` java
  //1.调用Class类中的getField(String name)方法获取Field类对象
  //2.调用Field对象的setAccessible(true)方法确保属性可访问
  //3.调用Filed对象的get()set()方法进行属性的读写
  Object get(Object obj)
      //Object 返回的属性值，数据类型为Object，可强转
      //Object obj 传入目标类的一个对象，若原属性为静态属性，则传入null
  void set(Object obj, Object value)
      //Object obj 传入目标类的一个对象，若原属性为静态属性，则传入null
      //Object value 传入属性值
  ```

- 代码实现

  ```java
  //前提：获取Class类和Person类实例
  Class clazz = Person.class;
  Person p1 = (Person) clazz.newInstance();
  
  //调用静态属性 public int age
  //1.获取目标属性
  Field ageField = clazz.getField("age");
  //2.确保目标属性可访问
  ageField.setAccessible(true);
  //3.属性的读写
  int age = (int) ageField.get(p1);
  ageField.set(p1,10);
  
  //调用非静态属性 private String name
  //1.获取目标属性
  Field nameField = clazz.getDeclaredField("name");
  //2.确保目标属性可访问
  nameField.setAccessible(true);
  //3.属性的读写
  String name = (String) nameField.get(null);
  nameField.set(null,"赵四");
  ```



#### 15.6.3 调用指定构造器

- 格式步骤

  ```java
  //1.调用Class类中的getDeclaredConstructor(Class...parameterTypes)方法获取Constructor类对象
  //2.调用Constructor对象的setAccessible(true)方法确保构造器可访问
  //3.调用Constructor对象的newInstance(Object[] args)方法调用指定构造器
  ```

- 代码实现

  ```java
  //前提：获取Class类实例
  Class clazz = Person.class;
  
  //1.通过形参列表类型获取指定的构造器
  Constructor constructor = clazz.getDeclaredConstructor(String.class, int.class);
  //2.确保指定的构造器可访问
  constructor.setAccessible(true);
  //3.使用构造器的newInstance()方法创建对象，注意对象类型为Object，需要强转
  Person p = (Person)constructor.newInstance("赵四",24);
  ```

  

### 15.7 动态代理