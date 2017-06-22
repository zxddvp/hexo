---
title: JAVA高级面试题1
date: 2017-06-22 15:38:45
tags:
---
==【注意，答案都是来自网络，自辨真假】==

1、关于spring的aop的实现方法。

	1.通知(Advice):
	通知定义了切面是什么以及何时使用。描述了切面要完成的工作和何时需要执行这个工作。
	2.连接点(Joinpoint):
	程序能够应用通知的一个“时机”，这些“时机”就是连接点，例如方法被调用时、异常被抛出时等等。
	3.切入点(Pointcut)
	通知定义了切面要发生的“故事”和时间，那么切入点就定义了“故事”发生的地点，例如某个类或方法的名称，spring中允许我们方便的用正则表达式来指定
	4.切面(Aspect)
	通知和切入点共同组成了切面：时间、地点和要发生的“故事”
	5.引入(Introduction)
	引入允许我们向现有的类添加新的方法和属性(Spring提供了一个方法注入的功能）
	6.目标(Target)
	即被通知的对象，如果没有AOP,那么它的逻辑将要交叉别的事务逻辑，有了AOP之后它可以只关注自己要做的事（AOP让他做爱做的事）
	7.代理(proxy)
	应用通知的对象，详细内容参见设计模式里面的代理模式
	8.织入(Weaving)
	把切面应用到目标对象来创建新的代理对象的过程，织入一般发生在如下几个时机:
	(1)编译时：当一个类文件被编译时进行织入，这需要特殊的编译器才可以做的到，例如AspectJ的织入编译器
	(2)类加载时：使用特殊的ClassLoader在目标类被加载到程序之前增强类的字节代码
	(3)运行时：切面在运行的某个时刻被织入,SpringAOP就是以这种方式织入切面的，原理应该是使用了JDK的动态代理技术
	Spring提供了4种实现AOP的方式：
	1.经典的基于代理的AOP
	2.@AspectJ注解驱动的切面
	3.纯POJO切面
	4.注入式AspectJ切面
	
---
2、关于分布式消息队列，分布式缓存

分布式消息队列：

	一、消息队列概述

	消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构。是大型分布式系统不可缺少的中间件。

	目前在生产环境，使用较多的消息队列有ActiveMQ，RabbitMQ，ZeroMQ，Kafka，MetaMQ，RocketMQ等。

	二、消息队列应用场景

	以下介绍消息队列在实际应用中常用的使用场景。异步处理，应用解耦，流量削锋和消息通讯四个场景。

	2.1异步处理

	场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种1.串行的方式；2.并行方式。

	（1）串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。
	
	（2）并行方式：将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。与串行的差别是，并行的方式可以提高处理的时间。
	
	2.2应用解耦

	场景说明：用户下单后，订单系统需要通知库存系统。传统的做法是，订单系统调用库存系统的接口。
	
	传统模式的缺点：

	1）  假如库存系统无法访问，则订单减库存将失败，从而导致订单失败；

	2）  订单系统与库存系统耦合；

	如何解决以上问题呢？引入应用消息队列后的方案
	
	订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。
	
	库存系统：订阅下单的消息，采用拉/推的方式，获取下单信息，库存系统根据下单信息，进行库存操作。
	
	假如：在下单时库存系统不能正常使用。也不影响正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作了。实现订单系统与库存系统的应用解耦。
	
	2.3流量削锋

	流量削锋也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。

	应用场景：秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列。
	
	2.4日志处理

	日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题
	
	日志采集客户端，负责日志数据采集，定时写受写入Kafka队列；
	Kafka消息队列，负责日志数据的接收，存储和转发；
	日志处理应用：订阅并消费kafka队列中的日志数据；
	
	(1)Kafka：接收用户日志的消息队列。

	(2)Logstash：做日志解析，统一成JSON输出给Elasticsearch。

	(3)Elasticsearch：实时日志分析服务的核心技术，一个schemaless，实时的数据存储服务，通过index组织数据，兼具强大的搜索和统计功能。

	(4)Kibana：基于Elasticsearch的数据可视化组件，超强的数据可视化能力是众多公司选择ELK stack的重要原因。
	
	2.5消息通讯

	消息通讯是指，消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。




分布式缓存：
	
	1.1 分布式缓存的特性

	分布式缓存具有如下特性: 

	1) 高性能:当传统数据库面临大规模数据访问时,磁盘I/O 往往成为性能瓶颈,从而导致过高的响应延迟.分布式缓存将高速内存作为数据对象的存储介质,数据以key/value 形式存储,理想情况下可以获得DRAM 级的读写性能; 

	2) 动态扩展性:支持弹性扩展,通过动态增加或减少节点应对变化的数据访问负载,提供可预测的性能与扩展性;同时,最大限度地提高资源利用率; 

	3) 高可用性:可用性包含数据可用性与服务可用性两方面.基于冗余机制实现高可用性,无单点失效(single point of failure),支持故障的自动发现,透明地实施故障切换,不会因服务器故障而导致缓存服务中断或数据丢失.动态扩展时自动均衡数据分区,同时保障缓存服务持续可用; 

	4) 易用性:提供单一的数据与管理视图;API 接口简单,且与拓扑结构无关;动态扩展或失效恢复时无需人工配置;自动选取备份节点;多数缓存系统提供了图形化的管理控制台,便于统一维护; 

	5) 分布式代码执行(distributed code execution):将任务代码转移到各数据节点并行执行,客户端聚合返回结果,从而有效避免了缓存数据的移动与传输.最新的Java 数据网格规范JSR-347中加入了分布式代码执行与Map/reduce 的API 支持,各主流分布式缓存产品,如IBM WebSphere eXtreme Scale,VMware GemFire,GigaSpaces XAP 和Red Hat Infinispan 等也都支持这一新的编程模型. 

	1.2 典型应用场景 

	分布式缓存的典型应用场景可分为以下几类: 

	1) 页面缓存.用来缓存Web 页面的内容片段,包括HTML、CSS 和图片等,多应用于社交网站等; 

	2) 应用对象缓存.缓存系统作为ORM 框架的二级缓存对外提供服务,目的是减轻数据库的负载压力,加速应用访问; 

	3) 状态缓存.缓存包括Session 会话状态及应用横向扩展时的状态数据等,这类数据一般是难以恢复的,对可用性要求较高,多应用于高可用集群; 

	4) 并行处理.通常涉及大量中间计算结果需要共享; 

	5) 事件处理.分布式缓存提供了针对事件流的连续查询(continuous query)处理技术,满足实时性需求; 

	6) 极限事务处理.分布式缓存为事务型应用提供高吞吐率、低延时的解决方案,支持高并发事务请求处理,多应用于铁路、金融服务和电信等领域.


---

3、关于rpc

	RPC(Remote Procedure Call，远程过程调用)是建立在Socket之上的，在一台机器上运行的主程序，可以调用另一台机器上准备好的子程序，就像LPC(本地过程调用)。

	越底层，代码越复杂、灵活性越高、效率越高；越上层，抽象封装的越好、代码越简单、效率越差。

	Socket和RPC的区别再次说明了这点。在传统的编程概念中，过程是由程序员在本地编译完成，并只能局限在本地运行的一段代码，也即其主程序和过程之间的运行关系是本地调用关系。因此这种结构在网络日益发展的今天已无法适应实际需求。众所周知，传统过程调用模式无法充分利用网络上其他主机的资源(如CPU、Memory等)，也无法提高代码在实体间的共享程度，使得主机资源大量浪费。

	通过RPC，我们可以充分利用非共享内存的多处理器环境(例如通过局域网连接得多台工作站)，这样可以简便地将你的应用分布在多台工作站上，应用程序就像运行在一个多处理器的计算机上一样。你可以方便的实现过程代码共享，提高系统资源的利用率，也可以将以大量数值处理的操作放在处理能力较强的系统上运行，从而减轻前端机的负担。

	RPC作为普遍的C/S开发方法，开发效率高效，可靠。但RPC方法的基本原则是：以模块调用的简单性忽略通讯的具体细节，以便程序员不用关心C/S之间的通讯协议，集中精力对付实现过程。这就决定了 RPC生成的通讯包不可能对每种应用都有最恰当的处理办法，与Socket方法相比，传输相同的有效数据，RPC占用更多的网络带宽。

	RPC是在Socket的基础上实现的，它比socket需要更多的网络和系统资源。

	RPC的结构原理及其调用机制

	如前所述，RPC其实也是种C/S的编程模式，有点类似C/S Socket 编程模式，但要比它更高一层。

	当我们在建立RPC服务以后，客户端的调用参数通过底层的RPC传输通道，可以是UDP，也可以是TCP，并根据传输前所提供的目的地址及RPC上层应用程序号转至相应的RPC应用程序服务端，且此时的客户端处于等待状态，直至收到应答或Time Out超时信号。当服务器端获得请求消息，则会根据注册RPC时告诉RPC系统的例程入口地址，执行相应的操作，并将结果返回至客户端。

	当一次RPC调用结束后，相应线程发送相应的信号，客户端程序才会继续运行。

	在这个过程中，一个远程过程是有三个要素来唯一确定的：程序号、版本号和过程号。

	程序号是用来区别一组相关的并且具有唯一过程好的远程过程。一个程序可以有一个或几个不同的版本，而每个版本的程序都包含一系列能被远程调用的过程，通过版本的引入，使得不同版本下的RPC能同时提供服务。每个版本都包含有许多可供远程调用的过程，每个过程则有其唯一标示的过程号。

	基于RPC的应用系统开发

	一般而言在开发RPC时，我们通常分为三个步骤：

	A、定义说明客户/服务器的通信协议:这里所说的通信协议是指定义服务过程的名称、调用参数的数据类型和返回参数的数据类型，还包括底层传输类型(可以是UDP或TCP)，当然也可以由RPC底层函数自动选择连接类型建立TI-RPC。最简单的协议生成的方法是采用协议编译工具，常用的有Rpcgen，我会在后面实例中详细描述其使用方法。

	B、开发客户端程序。

	C、开发服务器端程序。

	开发客户端和服务器端的程序时，RPC提供了我们不同层次的开发例程调用接口。不同层次的接口提供了对RPC不同程度控制。
	
---

4、关于hashmap源码实现

HashMap实现原理及源码分析

　　哈希表（hash table）也叫散列表，是一种非常重要的数据结构，应用场景及其丰富，许多缓存技术（比如memcached）的核心其实就是在内存中维护一张大的哈希表，而HashMap的实现原理也常常出现在各类的面试题中，重要性可见一斑。本文会对java集合框架中的对应实现HashMap的实现原理进行讲解，然后会对JDK7的HashMap源码进行分析。

目录

　　一、什么是哈希表

　　二、HashMap实现原理

　　三、为何HashMap的数组长度一定是2的次幂？

　　四、重写equals方法需同时重写hashCode方法

　　五、总结

一、什么是哈希表

　　在讨论哈希表之前，我们先大概了解下其他数据结构在新增，查找等基础操作执行性能

　　数组：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

　　线性链表：对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

　　二叉树：对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

　　哈希表：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)，接下来我们就来看看哈希表是如何实现达到惊艳的常数阶O(1)的。

　　我们知道，数据结构的物理存储结构只有两种：顺序存储结构和链式存储结构（像栈，队列，树，图等是从逻辑结构去抽象的，映射到内存中，也这两种物理组织形式），而在上面我们提到过，在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的主干就是数组。

　　比如我们要新增或查找某个元素，我们通过把当前元素的关键字 通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作。

　　　　　　　　存储位置 = f(关键字)

　　其中，这个函数f一般称为哈希函数，这个函数的设计好坏会直接影响到哈希表的优劣。举个例子，比如我们要在哈希表中执行插入操作：

　　

　　查找操作同理，先通过哈希函数计算出实际存储地址，然后从数组中对应地址取出即可。

　　哈希冲突

　　然而万事无完美，如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？哈希冲突的解决方案有多种:开放定址法（发生冲突，继续寻找下一块未被占用的存储地址），再散列函数法，链地址法，而HashMap即是采用了链地址法，也就是数组+链表的方式，

二、HashMap实现原理

　HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。

//HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂，至于为什么这么做，后面会有详细分析。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
 Entry是HashMap中的一个静态内部类。代码如下


    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;//存储指向下一个Entry的引用，单链表结构
        int hash;//对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        } 

 所以，HashMap的整体结构如下

　　

　　简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为O(1)，因为最新的Entry会插入链表头部，急需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

其他几个重要字段


	//实际存储的key-value键值对的个数
	transient int size;
	//阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold，后面会详细谈到
	int threshold;
	//负载因子，代表了table的填充度有多少，默认是0.75
	final float loadFactor;
	//用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
	transient int modCount;

HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值

initialCapacity默认为16，loadFactory默认为0.75

我们看下其中一个


	public HashMap(int initialCapacity, float loadFactor) {
	//此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30(230)
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();//init方法在HashMap中没有实际实现，不过在其子类如 linkedHashMap中就会有对应实现
    }

　　从上面这段代码我们可以看出，在常规构造器中，没有为数组table分配内存空间（有一个入参为指定Map的构造器例外），而是在执行put操作的时候才真正构建table数组

　　OK,接下来我们来看看put操作的实现吧

 
    public V put(K key, V value) {
        //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4(24=16)
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
       //如果key为null，存储位置为table[0]或table[0]的冲突链上
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);//对key的hashcode进一步计算，确保散列均匀
        int i = indexFor(hash, table.length);//获取在table中的实际位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;//保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        addEntry(hash, key, value, i);//新增一个entry
        return null;
    }    

 先来看看inflateTable这个方法


	private void inflateTable(int toSize) {
        int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);//此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }


　　inflateTable这个方法用于为主干数组table在内存中分配存储空间，通过roundUpToPowerOf2(toSize)可以确保capacity为大于或等于toSize的最接近toSize的二次幂，比如toSize=13,则capacity=16;to_size=16,capacity=16;to_size=17,capacity=32.

	private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }

roundUpToPowerOf2中的这段处理使得数组长度一定为2的次幂，Integer.highestOneBit是用来获取最左边的bit（其他bit位为0）所代表的数值.

hash函数

	//这是一个神奇的函数，用了很多的异或，移位等运算，对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
	final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

以上hash函数计算出的值，通过indexFor进一步处理来获取实际的存储位置

	/**
     * 返回数组下标
     */
    static int indexFor(int h, int length) {
        return h & (length-1);
    }


h&（length-1）保证获取的index一定在数组范围内，举个例子，默认容量16，length-1=15，h=18,转换成二进制计算为

        1  0  0  1  0
    &   0  1  1  1  1
    __________________
        0  0  0  1  0    = 2
　　最终计算出的index=2。有些版本的对于此处的计算会使用 取模运算，也能保证index一定在数组范围内，不过位运算对计算机来说，性能更高一些（HashMap中有大量位运算）

所以最终存储位置的确定流程是这样的：



再来看看addEntry的实现：


	void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }


　　通过以上代码能够得知，当发生哈希冲突并且size大于阈值的时候，需要进行数组扩容，扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。

三、为何HashMap的数组长度一定是2的次幂？

我们来继续看上面提到的resize方法

	void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

如果数组进行扩容，数组长度发生变化，而存储位置 index = h&(length-1),index也可能会发生变化，需要重新计算index，我们先来看看transfer这个方法

	void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        //for循环中的代码，逐个遍历链表，重新计算索引位置，将老数组数据复制到新数组中去（数组不存储实际数据，所以仅仅是拷贝引用而已）
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                //将当前entry的next链指向新的索引位置,newTable[i]有可能为空，有可能也是个entry链，如果是entry链，直接在链表头部插入。
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
    

　　这个方法将老数组中的数据逐个链表地遍历，扔到新的扩容后的数组中，我们的数组索引位置的计算是通过 对key值的hashcode进行hash扰乱运算后，再通过和 length-1进行位运算得到最终数组索引位置。

　　hashMap的数组长度一定保持2的次幂，比如16的二进制表示为 10000，那么length-1就是15，二进制为01111，同理扩容后的数组长度为32，二进制表示为100000，length-1为31，二进制表示为011111。从下图可以我们也能看到这样会保证低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)，个人理解。

　　

 还有，数组长度保持2的次幂，length-1的低位都为1，会使得获得的数组索引index更加均匀，比如：



　　我们看到，上面的&运算，高位是不会对结果产生影响的（hash函数采用各种位运算可能也是为了使得低位更加散列），我们只关注低位bit，如果低位全部为1，那么对于h低位部分来说，任何一位的变化都会对结果产生影响，也就是说，要得到index=21这个存储位置，h的低位只有这一种组合。这也是数组长度设计为必须为2的次幂的原因。



　　如果不是2的次幂，也就是低位不是全为1此时，要使得index=21，h的低位部分不再具有唯一性了，哈希冲突的几率会变的更大，同时，index对应的这个bit位无论如何不会等于1了，而对应的那些数组位置也就被白白浪费了。

get方法

	public V get(Object key) {
		//如果key为null,则直接去table[0]处去检索即可。
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);
        return null == entry ? null : entry.getValue();
    }
    

get方法通过key值返回对应value，如果key为null，直接去table[0]处检索。我们再看一下getEntry这个方法

	final Entry<K,V> getEntry(Object key) {
            
        if (size == 0) {
            return null;
        }
        //通过key的hashcode值计算hash值
        int hash = (key == null) ? 0 : hash(key);
        //indexFor (hash&length-1) 获取最终数组索引，然后遍历链表，通过equals方法比对找出对应记录
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && 
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }    


　　可以看出，get方法的实现相对简单，key(hashcode)-->hash-->indexFor-->最终索引位置，找到对应位置table[i]，再查看是否有链表，遍历链表，通过key的equals方法比对查找对应的记录。要注意的是，有人觉得上面在定位到数组位置之后然后遍历链表的时候，e.hash == hash这个判断没必要，仅通过equals判断就可以。其实不然，试想一下，如果传入的key对象重写了equals方法却没有重写hashCode，而恰巧此对象定位到这个数组位置，如果仅仅用equals判断可能是相等的，但其hashCode和当前对象不一致，这种情况，根据Object的hashCode的约定，不能返回当前对象，而应该返回null，后面的例子会做出进一步解释。

四、重写equals方法需同时重写hashCode方法

　　关于HashMap的源码分析就介绍到这儿了，最后我们再聊聊老生常谈的一个问题，各种资料上都会提到，“重写equals时也要同时覆盖hashcode”，我们举个小例子来看看，如果重写了equals而不重写hashcode会发生什么样的问题

	/**
 	* Created by chengxiao on 2016/11/15.
 	*/
	public class MyTest {
    private static class Person{
        int idCard;
        String name;

        public Person(int idCard, String name) {
            this.idCard = idCard;
            this.name = name;
        }
        @Override
        public boolean equals(Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()){
                return false;
            }
            Person person = (Person) o;
            //两个对象是否等值，通过idCard来确定
            return this.idCard == person.idCard;
        }

    }
    public static void main(String []args){
        HashMap<Person,String> map = new HashMap<Person, String>();
        Person person = new Person(1234,"乔峰");
        //put到hashmap中去
        map.put(person,"天龙八部");
        //get取出，从逻辑上讲应该能输出“天龙八部”
        System.out.println("结果:"+map.get(new Person(1234,"萧峰")));
    }
	}


实际输出结果：

结果：null

　　如果我们已经对HashMap的原理有了一定了解，这个结果就不难理解了。尽管我们在进行get和put操作的时候，使用的key从逻辑上讲是等值的（通过equals比较是相等的），但由于没有重写hashCode方法，所以put操作时，key(hashcode1)-->hash-->indexFor-->最终索引位置 ，而通过key取出value的时候 key(hashcode1)-->hash-->indexFor-->最终索引位置，由于hashcode1不等于hashcode2，导致没有定位到一个数组位置而返回逻辑上错误的值null（也有可能碰巧定位到一个数组位置，但是也会判断其entry的hash值是否相等，上面get方法中有提到。）

　　所以，在重写equals的方法的时候，必须注意重写hashCode方法，同时还要保证通过equals判断相等的两个对象，调用hashCode方法要返回同样的整数值。而如果equals判断不相等的两个对象，其hashCode可以相同（只不过会发生哈希冲突，应尽量避免）。

五、总结

　　本文描述了HashMap的实现原理，并结合源码做了进一步的分析，也涉及到一些源码细节设计缘由，最后简单介绍了为什么重写equals的时候需要重写hashCode方法。



