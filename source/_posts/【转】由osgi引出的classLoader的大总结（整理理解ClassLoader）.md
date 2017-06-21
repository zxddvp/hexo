---
title: 【转】由osgi引出的classLoader的大总结（整理理解ClassLoader）
date: 2017-06-21 17:02:02
tags: classloader
---


转载请注明出处（corey）
最近在研究osgi，在osgi里面里面有个很重要的东西，就是ClassLoader，所以，在网上搜集了一些资料，整理一下，
并加入了自己的一些理解；

(1)jvm的装载过程以及装载原理
所谓装载就是寻找一个类或是一个接口的二进制形式并用该二进制形式来构造代表这个类或是这个接口的class对象的过程，
其中类或接口的名称是给定了的。当然名称也可以通过计算得到，但是更常见的是通过搜索源代码经过编译器编译后所得到
的二进制形式来构造。 

在Java中，类装载器把一个类装入Java虚拟机中，要经过三个步骤来完成：装载、链接和初始化，
其中链接又可以分成校验、准备和解析三步，除了解析外，其它步骤是严格按照顺序完成的，各个步骤的主要工作如下：

　　装载：查找和导入类或接口的二进制数据； 
　　
　　链接：执行下面的校验、准备和解析步骤，其中解析步骤是可以选择的； 
　　
　　校验：检查导入类或接口的二进制数据的正确性；
　　 
　　准备：给类的静态变量分配并初始化存储空间； 
　　
　　解析：将符号引用转成直接引用； 
　　
　　初始化：激活类的静态变量的初始化Java代码和静态Java代码块。
　　
(2)：java中的类是什么？

一个类代表要执行的代码，而数据则表示其相关状态。状态时常改变，而代码则不会。当我们将一个特定的状态与一个类相对应起来，也就意味着将一个类事例化。尽管相同的类对应的实例其状态千差万别，但其本质都对应着同一段代码。在JAVA中，一个类通常有着一个.class文件，但也有例外。在JAVA的运行时环境中（Java runtime），每一个类都有一个以第一类(first-class)的Java对象所表现出现的代码，其是java.lang.Class的实例。我们编译一个JAVA文件，编译器都会嵌入一个public, static, final修饰的类型为java.lang.Class，名称为class的域变量在其字节码文件中。因为使用了public修饰，我们可以采用如下的形式对其访问：
java.lang.Class klass = Myclass.class;

一旦一个类被载入JVM中，同一个类就不会被再次载入了（切记，同一个类）。这里存在一个问题就是什么是“同一个类”？正如一个对象有一个具体的状态，即标识，一个对象始终和其代码(类)相关联。同理，载入JVM的类也有一个具体的标识，我们接下来看。

在Java中，一个类用其完全匹配类名(fully qualified class name)作为标识，这里指的完全匹配类名包括包名和类名。但在JVM中一个类用其全名和一个加载类ClassLoader的实例作为唯一标识。因此，如果一个名为Pg的包中，有一个名为Cl的类，被类加载器KlassLoader的一个实例kl1加载，Cl的实例，即C1.class在JVM中表示为(Cl, Pg, kl1)。这意味着两个类加载器的实例(Cl, Pg, kl1) 和 (Cl, Pg, kl2)是不同的，被它们所加载的类也因此完全不同，互不兼容的。那么在JVM中到底有多少种类加载器的实例？下一节我们揭示答案。

(3)：java的几种ClassLoader：

在java中，我们可以取得这么以下三个ClassLoader类：

一．    ClassLoader基本概念

1．ClassLoader分类

类装载器是用来把类(class)装载进JVM的。

JVM规范定义了两种类型的类装载器：启动内装载器(bootstrap)和用户自定义装载器(user-defined class loader)。

JVM在运行时会产生三个ClassLoader:Bootstrap ClassLoader、Extension ClassLoader和AppClassLoader。Bootstrap是用C++编写的，我们在Java中看不到它,是null,是JVM自带的类装载器，用来装载核心类库，如java.lang.*等。

AppClassLoader的Parent是ExtClassLoader，而ExtClassLoader的Parent为Bootstrap ClassLoader。
 
Java提供了抽象类ClassLoader，所有用户自定义类装载器都实例化自ClassLoader的子类。 System Class Loader是一个特殊的用户自定义类装载器，由JVM的实现者提供，在编程者不特别指定装载器的情况下默认装载用户类。系统类装载器可以通过

ClassLoader.getSystemClassLoader() 方法得到。
 
 
例1，测试你所使用的JVM的ClassLoader

	/*LoaderSample1.java*/
	public   class  LoaderSample1 {
     public   static   void  main(String[] args) {
        Class c;
        ClassLoader cl;
        cl  =  ClassLoader.getSystemClassLoader();
        System.out.println(cl);
         while  (cl  !=   null ) {
            cl  =  cl.getParent();
            System.out.println(cl);
        }
         try  {
            c  =  Class.forName( " java.lang.Object " );
            cl  =  c.getClassLoader();
            System.out.println( " java.lang.Object's loader is  "   +  cl);
            c  =  Class.forName( " LoaderSample1 " );
            cl  =  c.getClassLoader();
            System.out.println( " LoaderSample1's loader is  "   +  cl);
        }  catch  (Exception e) {
            e.printStackTrace();
        }
    }
	}

在我的机器上(Sun Java 1.4.2)的运行结果

sun.misc.Launcher$AppClassLoader@1a0c10f
sun.misc.Launcher$ExtClassLoader@e2eec8
null
java.lang.Object's loader is null
LoaderSample1's loader is sun.misc.Launcher$AppClassLoader@1a0c10f

第一行表示，系统类装载器实例化自类

sun.misc.Launcher$AppClassLoader

第二行表示，系统类装载器的parent实例化自类

sun.misc.Launcher$ExtClassLoader

第三行表示，系统类装载器parent的parent为bootstrap

第四行表示，核心类java.lang.Object是由bootstrap装载的

第五行表示，用户类LoaderSample1是由系统类装载器装载的

注意，我们清晰的看见这个三个ClassLoader类之间的父子关系（不是继承关系），父子关系在ClassLoader的实现中有一个ClassLoader类型的属性，我们可以在自己实现自定义的ClassLoader的时候初始化定义，而这三个系统定义的ClassLoader的父子关系分别是

AppClassLoader——————》（Parent）ExtClassLoader——————————》（parent）BootClassLoader(null c++实现)

系统为什么要分别指定这么多的ClassLoader类呢？

答案在于因为java是动态加载类的，这样的话，可以节省内存，用到什么加载什么，就是这个道理，然而系统在运行的时候并不知道我们这个应用与需要加载些什么类，那么，就采用这种逐级加载的方式

(1)首先加载核心API，让系统最基本的运行起来

(2)加载扩展类

(3)加载用户自定义的类

	package org.corey.clsloader;
	import java.net.MalformedURLException;
	import java.net.URL;
	import java.net.URLClassLoader;
	import sun.net.spi.nameservice.dns.DNSNameService;
	public class ClsLoaderDemo {
	 /**
 	 * @param args
 	 */
 	public static void main(String[] args) {
	System.out.println(System.getProperty("sun.boot.class.path"));
	System.out.println(System.getProperty("java.ext.dirs"));
 	System.out.println(System.getProperty("java.class.path"));
 	}
	}

程序结果为：

E:/MyEclipse 6.0/jre/lib/rt.jar;E:/MyEclipse 6.0/jre/lib/i18n.jar;E:/MyEclipse 6.0/jre/lib/sunrsasign.jar;E:/MyEclipse 6.0/jre/lib/jsse.jar;E:/MyEclipse 6.0/jre/lib/jce.jar;E:/MyEclipse 6.0/jre/lib/charsets.jar;E:/MyEclipse 6.0/jre/classes

E:/MyEclipse 6.0/jre/lib/ext

E:/workspace/ClassLoaderDemo/bin

在上面的结果中，你可以清晰看见三个ClassLoader分别加载类的路径；也知道为什么我们在编写程序的时候，要把用到的jar包放在工程的classpath下面啦，也知道我们为什么可以不加载java.lang.*包啦！其中java.lang.*就在rt.jar包中；

(4)ClassLoader的加载机制：

现在我们设计这种一下Demo:

	package java.net;
	public class URL {
 	private String path;
 	public URL(String path) {
 	 this.path = path;
 	}
 	public String toString() {
 	 return this.path + " new Path";
 	}
	}

	package java.net;
	import java.net.*;
	public class TheSameClsDemo {
	 /**
 	 * @param args
  	*/
	 public static void main(String[] args) {
 	 URL url = new URL("http://www.baidu.com");
 	 System.out.println(url.toString());
	 }
	}
	
在这种情况下，系统会提示我们出现异常，因为我们有两个相同的类，一个是真正的URL，一个是我在上面实现的伪类；出现异常是正常的，因为你想想，如果我们在执行一个applet的时候，程序自己实现了一个String的类覆盖了我们虚拟机上面的真正的String类，那么在这个String里面，不怀好意的人可以任意的实现一些功能；这就造成极不安全的隐患；所以java采用了一种名为“双亲委托”的加载模式；

以下是jdk源代码：

	protected synchronized Class<?> loadClass(String name, boolean resolve)
 	throws ClassNotFoundException
    	{
 	// First, check if the class has already been loaded
 	Class c = findLoadedClass(name);
 	if (c == null) {
     	try {
  	if (parent != null) {
    	  c = parent.loadClass(name, false);
  	} else {
     	 c = findBootstrapClass0(name);
  	}
     	} catch (ClassNotFoundException e) {
         // If still not found, then invoke findClass in order
         // to find the class.
         c = findClass(name);
     	}
 	}
 	if (resolve) {
     	resolveClass(c);
	 }
 		return c;
    }
    
在上面的代码中，我们可以清晰的看见，我们调用一个ClassLoader加载程序的时候，这个ClassLoader会先调用设置好的parent ClassLoader来加载这个类，如果parent是null的话，则默认为Boot ClassLoader类，只有在parent没有找的情况下，自己才会加载，这就避免我们重写一些系统类，来破坏系统的安全；
再来看一个明显的例子：

	package org.corey;
	public class MyCls{
 	private String name;
 
 	public MyCls(){
 
 	}
 	public MyCls(String name){
 	this.name=name;
	 }
  
 	public void say(){
 		System.out.println(this.name); 
 	}
	}
	
把上面这个MyCls类打成jar包，丢进ext classLoader的加载路径；
然后写出main类：

	package org.corey.clsloader;
	import org.corey.MyCls;
	public class TheSameClsDemo {
 	/**
 	 * @param args
 	 */
 	public static void main(String[] args) {
 	 MyCls myClsOb=new MyCls("name");
     myClsOb.say(); 
     System.out.println(MyCls.class.getClassLoader());
     System.out.println(System.getProperty("java.class.path"));
     System.out.println(TheSameClsDemo.class.getClassLoader());
 	}
	}
	
并且把MyCls类加入biild-path里面方便引用；
结果是：

name
sun.misc.Launcher$ExtClassLoader@16930e2

E:/workspace/ClassLoaderDemo/bin;E:/MyEclipse 6.0/jre/lib/ext/corey.jar

sun.misc.Launcher$AppClassLoader@7259da

从上面的例子可以清晰的看出ClassLoader之间的这种双亲委托加载模式；

再来看下一个例子(摘自http://bbs.cnw.com.cn/viewthread.php?tid=95389)

下面我们就来看一个综合的例子。首先在eclipse中建立一个简单的java应用工程，然后写一个简单的JavaBean如下：

	package classloader.test.bean;
 
	publicclass TestBean {
 

	public TestBean() {}

	}
	
在现有当前工程中另外建立一测试类（ClassLoaderTest.java）内容如下：
测试一：

	publicclass ClassLoaderTest {
 	publicstaticvoid main(String[] args) {
 	try {
 	//查看当前系统类路径中包含的路径条目
	System.out.println(System.getProperty("java.class.path"));
	//调用加载当前类的类加载器（这里即为系统类加载器）加载TestBean
	Class typeLoaded = Class.forName("classloader.test.bean.TestBean");
	//查看被加载的TestBean类型是被那个类加载器加载的
	System.out.println(typeLoaded.getClassLoader());
 	} catch (Exception e) {
	 	e.printStackTrace();
 	}
	}
	}

对应的输出如下：

	D:"DEMO"dev"Study"ClassLoaderTest"bin

	sun.misc.Launcher$AppClassLoader@197d257
	
（说明：当前类路径默认的含有的一个条目就是工程的输出目录）

测试二：

将当前工程输出目录下的…/classloader/test/bean/TestBean.class打包进test.jar剪贴到< Java_Runtime_Home >/lib/ext目录下（现在工程输出目录下和JRE扩展目录下都有待加载类型的class文件）。再运行测试一测试代码，结果如下：

D:"DEMO"dev"Study"ClassLoaderTest"bin

sun.misc.Launcher$ExtClassLoader@7259da

对比测试一和测试二，我们明显可以验证前面说的双亲委派机制，系统类加载器在接到加载classloader.test.bean.TestBean类型的请求时，首先将请求委派给父类加载器（标准扩展类加载器），标准扩展类加载器抢先完成了加载请求。

测试三：

将test.jar拷贝一份到< Java_Runtime_Home >/lib下，运行测试代码，输出如下：

D:"DEMO"dev"Study"ClassLoaderTest"bin

sun.misc.Launcher$ExtClassLoader@7259da

测试三和测试二输出结果一致。那就是说，放置到< Java_Runtime_Home >/lib目录下的TestBean对应的class字节码并没有被加载，这其实和前面讲的双亲委派机制并不矛盾。虚拟机出于安全等因素考虑，不会加载< Java_Runtime_Home >/lib存在的陌生类，开发者通过将要加载的非JDK自身的类放置到此目录下期待启动类加载器加载是不可能的。做个进一步验证，删除< Java_Runtime_Home >/lib/ext目录下和工程输出目录下的TestBean对应的class文件，然后再运行测试代码，则将会有ClassNotFoundException异常抛出。有关这个问题，大家可以在java.lang.ClassLoader中的loadClass(String name, boolean resolve)方法中设置相应断点运行测试三进行调试，会发现findBootstrapClass0()会抛出异常，然后在下面的findClass方法中被加载，当前运行的类加载器正是扩展类加载器（sun.misc.Launcher$ExtClassLoader），这一点可以通过JDT中变量视图查看验证。

(5)被不同的ClassLoader加载的两个类之间有什么限制和不同?

现在我们来看一下一个现象：

在eclipse里面我是这样做的：

OneCls.java

	package org.corey.one;
	import org.corey.two.TwoCls;
	public class OneCls {
	 public OneCls() {
	 System.out.println();
  	 TwoCls two = new TwoCls();
 	 two.say();
 	}
	}
	
TwoCls.java

	package org.corey.two;
	public class TwoCls {
 
	 public void say() {
	  System.out.println("i am two");
	 }
	}
	
Demo.java:

	package org.corey.Demo;
	import org.corey.one.OneCls;
	public class Demo {
	 /**
 	 * @param args
 	 */
 	public static void main(String[] args) {
 	 OneCls one=new OneCls();
 	}
	}
	
在这里，我们来仔细看下，one引用了two，demo引用了one，这是三个类都是由AppClassLoader加载的；运行正常；

把OneCls打成jar包，放在lib/ext路径下面，然后在工程里面引入这个jar包；运行：异常，这是因为：

Demo是由AppClassLoader载入，委托给双亲加载失败后，由AppClassLoader加载，而加载OneCls的时候，委托给双亲，被ExtClassLoader加载成功，但是在载入OneCls的时候，同时引用了TwoCls,但是ExtClassLoader引用TwoCls失败，但是他只会委托给双亲，而不会委托给AppClassLoader这个儿子，所以会出现异常；

\3. 奇怪的隔离性

我们不难发现，图2中的类装载器AA和AB， AB和BB，AA和B等等位于不同分支下，他们之间没有父子关系，我不知道如何定义这种关系，姑且称他们位于不同分支下。两个位于不同分支的类装载器具有隔离性，这种隔离性使得在分别使用它们装载同一个类，也会在内存中出现两个Class类的实例。因为被具有隔离性的类装载器装载的类不会共享内存空间，使得使用一个类装载器不可能完成的任务变得可以轻而易举，例如类的静态变量可能同时拥有多个值（虽然好像作用不大），因为就算是被装载类的同一静态变量，它们也将被保存不同的内存空间，又例如程序需要使用某些包，但又不希望被程序另外一些包所使用，很简单，编写自定义的类装载器。类装载器的这种隔离性在许多大型的软件应用和服务程序得到了很好的应用。下面是同一个类静态变量为不同值的例子。

package test;
public class A {
  public static void main( String[] args ) {
    try {
      //定义两个类装载器
      MyClassLoader aa= new MyClassLoader();
      MyClassLoader bb = new MyClassLoader();
      //用类装载器aa装载testb.B类
      Class clazz=aa.loadClass("testb. B");
      Constructor constructor= 
        clazz.getConstructor(new Class[]{Integer.class});
      Object object = 
     constructor.newInstance(new Object[]{new Integer(1)});
      Method method = 
     clazz.getDeclaredMethod("printB",new Class[0]);
      //用类装载器bb装载testb.B类
      Class clazz2=bb.loadClass("testb. B");
      Constructor constructor2 = 
        clazz2.getConstructor(new Class[]{Integer.class});
      Object object2 = 
     constructor2.newInstance(new Object[]{new Integer(2)});
      Method method2 = 
     clazz2.getDeclaredMethod("printB",new Class[0]);
      //显示test.B中的静态变量的值 
      method.invoke( object,new Object[0]);
      method2.invoke( object2,new Object[0]);
    } catch ( Exception e ) {
      e.printStackTrace();
    }
  }
}
 
//Class B 必须位于MyClassLoader的查找范围内，
//而不应该在MyClassLoader的父类装载器的查找范围内。
package testb;
public class B {
    static int b ;
    public B(Integer testb) {
        b = testb.intValue();
    }
    public void printB() {
        System.out.print("my static field b is ", b);
    }
}
 
public class MyClassLoader extends URLClassLoader{
  private static File file = new File("c://classes ");
  //该路径存放着class B，但是没有class A
  public MyClassLoader() {
    super(getUrl());
  }
  public static URL[] getUrl() {
    try {
      return new URL[]{file.toURL()};
    } catch ( MalformedURLException e ) {
      return new URL[0];
    }
  }
}
程序的运行结果为：
my static field b is 1
my static field b is 2
程序的结果非常有意思，从编程者的角度，我们甚至可以把不在同一个分支的类装载器看作不同的java虚拟机，因为它们彼此觉察不到对方的存在。程序在使用具有分支的类装载的体系结构时要非常小心，弄清楚每个类装载器的类查找范围，尽量避免父类装载器和子类装载器的类查找范围中有相同类名的类（包括包名和类名），下面这个例子就是用来说明这种情况可能带来的问题。
 
(6) 类如何被装载及类被装载的方式(转自Java类装载体系中的隔离性  作者：盛戈歆)
在java2中，JVM是如何装载类的呢，可以分为两种类型，一种是隐式的类装载，一种式显式的类装载。
2.1 隐式的类装载
隐式的类装载是编码中最常用得方式：
A b = new A();
如果程序运行到这段代码时还没有A类，那么JVM会请求装载当前类的类装器来装载类。问题来了，我把代码弄得复杂一点点，但依旧没有任何难度，请思考JVM得装载次序：
package test;
Public class A{
    public void static main(String args[]){
        B b ＝ new B();
    }
}
class B{C c;}
class C{}
揭晓答案，类装载的次序为A->B，而类C根本不会被JVM理会,先不要惊讶，仔细想想，这不正是我们最需要得到的结果。我们仔细了解一下JVM装载顺序。当使用Java A命令运行A类时，JVM会首先要求类路径类装载器(AppClassLoader)装载A类，但是这时只装载A，不会装载A中出现的其他类(B类)，接着它会调用A中的main函数，直到运行语句b ＝ new B()时，JVM发现必须装载B类程序才能继续运行，于是类路径类装载器会去装载B类，虽然我们可以看到B中有有C类的声明，但是并不是实际的执行语句，所以并不去装载C类，也就是说JVM按照运行时的有效执行语句，来决定是否需要装载新类，从而装载尽可能少的类，这一点和编译类是不相同的。
2.2 显式的类装载
使用显示的类装载方法很多，我们都装载类test.A为例。
使用Class类的forName方法。它可以指定装载器，也可以使用装载当前类的装载器。例如：
Class.forName("test.A");
它的效果和
Class.forName("test.A",true,this.getClass().getClassLoader());
是一样的。
使用类路径类装载装载.
ClassLoader.getSystemClassLoader().loadClass("test.A");
使用当前进程上下文的使用的类装载器进行装载，这种装载类的方法常常被有着复杂类装载体系结构的系统所使用。
Thread.currentThread().getContextClassLoader().loadClass("test.A")
使用自定义的类装载器装载类
public class MyClassLoader extends URLClassLoader{
public MyClassLoader() {
        super(new URL[0]);
    }
}
MyClassLoader myClassLoader = new MyClassLoader();
myClassLoader.loadClass("test.A");
MyClassLoader继承了URLClassLoader类，这是JDK核心包中的类装载器，在没有指定父类装载器的情况下，类路径类装载器就是它的父类装载器，MyClassLoader并没有增加类的查找范围，因此它和类路径装载器有相同的效果。
 
(7)ClassLoader的一些方法实现的功能：
方法 loadClass
 
 
ClassLoader.loadClass() 是 ClassLoader 的入口点。其特征如下：

Class loadClass( String name, boolean resolve ); name 参数指定了 JVM 需要的类的名称，该名称以包表示法表示，如 Foo 或 java.lang.Object。
resolve 参数告诉方法是否需要解析类。在准备执行类之前，应考虑类解析。并不总是需要解析。如果 JVM 只需要知道该类是否存在或找出该类的超类，那么就不需要解析。
在 Java 版本 1.1 和以前的版本中，loadClass 方法是创建定制的 ClassLoader 时唯一需要覆盖的方法。（Java 2 中 ClassLoader 的变动提供了关于 Java 1.2 中 findClass() 方法的信息。）
 

方法 defineClass

defineClass 方法是 ClassLoader 的主要诀窍。该方法接受由原始字节组成的数组并把它转换成 Class 对象。原始数组包含如从文件系统或网络装入的数据。
defineClass 管理 JVM 的许多复杂、神秘和倚赖于实现的方面 -- 它把字节码分析成运行时数据结构、校验有效性等等。不必担心，您无需亲自编写它。事实上，即使您想要这么做也不能覆盖它，因为它已被标记成最终的。
你可以看见native标记，知道defineClass是一个jni调用的方法，是由c++实现数据到内存的加载的；
 

方法 findSystemClass

findSystemClass 方法从本地文件系统装入文件。它在本地文件系统中寻找类文件，如果存在，就使用 defineClass 将原始字节转换成 Class 对象，以将该文件转换成类。当运行 Java 应用程序时，这是 JVM 正常装入类的缺省机制。（Java 2 中 ClassLoader 的变动提供了关于 Java 版本 1.2 这个过程变动的详细信息。）
对于定制的 ClassLoader，只有在尝试其它方法装入类之后，再使用 findSystemClass。原因很简单：ClassLoader 是负责执行装入类的特殊步骤，不是负责所有类。例如，即使 ClassLoader 从远程的 Web 站点装入了某些类，仍然需要在本地机器上装入大量的基本 Java 库。而这些类不是我们所关心的，所以要 JVM 以缺省方式装入它们：从本地文件系统。这就是 findSystemClass 的用途。
其工作流程如下：

请求定制的 ClassLoader 装入类。 
检查远程 Web 站点，查看是否有所需要的类。 
如果有，那么好；抓取这个类，完成任务。 
如果没有，假定这个类是在基本 Java 库中，那么调用 findSystemClass，使它从文件系统装入该类。 
在大多数定制 ClassLoaders 中，首先调用 findSystemClass 以节省在本地就可以装入的许多 Java 库类而要在远程 Web 站点上查找所花的时间。然而，正如，在下一章节所看到的，直到确信能自动编译我们的应用程序代码时，才让 JVM 从本地文件系统装入类。
 

方法 resolveClass
正如前面所提到的，可以不完全地（不带解析）装入类，也可以完全地（带解析）装入类。当编写我们自己的 loadClass 时，可以调用 resolveClass，这取决于 loadClass 的 resolve 参数的值。
方法 findLoadedClass
findLoadedClass 充当一个缓存：当请求 loadClass 装入类时，它调用该方法来查看ClassLoader 是否已装入这个类，这样可以避免重新装入已存在类所造成的麻烦。应首先调用该方法。

三．命名空间及其作用
每个类装载器有自己的命名空间，命名空间由所有以此装载器为创始类装载器的类组成。不同命名空间的两个类是不可见的，但只要得到类所对应的Class对象的reference，还是可以访问另一命名空间的类。
 
例2演示了一个命名空间的类如何使用另一命名空间的类。在例子中，LoaderSample2由系统类装载器装载，LoaderSample3由自定义的装载器loader负责装载，两个类不在同一命名空间，但LoaderSample2得到了LoaderSample3所对应的Class对象的reference，所以它可以访问LoaderSampl3中公共的成员(如age)。
例2不同命名空间的类的访问

/\*LoaderSample2.java*/

	import  java.net. * ;
	import  java.lang.reflect. * ;
	public   class  LoaderSample2 {
     public   static   void  main(String[] args) {
         try  {
            String path  =  System.getProperty( " user.dir " );
            URL[] us  =  { new  URL( " file:// "   +  path  +   " /sub/ " )};
            ClassLoader loader  =   new  URLClassLoader(us);
            Class c  =  loader.loadClass( " LoaderSample3 " );
            Object o  =  c.newInstance();
            Field f  =  c.getField( " age " );
             int  age  =  f.getInt(o);
            System.out.println( " age is  "   +  age);
        }  catch  (Exception e) {
            e.printStackTrace();
        }
    }
	}

/\*sub/Loadersample3.java*/

	public   class  LoaderSample3 {
     static  {
        System.out.println( " LoaderSample3 loaded " );
    }
     public   int  age  =   30 ;
	}
	
编译：javac LoaderSample2.java; javac sub/LoaderSample3.java

运行：java LoaderSample2

LoaderSample3 loaded

age is 30

从运行结果中可以看出，在类LoaderSample2中可以创建处于另一命名空间的类LoaderSample3中的对象并可以访问其公共成员age。

运行时包(runtime package)

由同一类装载器定义装载的属于相同包的类组成了运行时包，决定两个类是不是属于同一个运行时包，不仅要看它们的包名是否相同，还要看的定义类装载器是否相同。只有属于同一运行时包的类才能互相访问包可见的类和成员。这样的限制避免了用户自己的代码冒充核心类库的类访问核心类库包可见成员的情况。假设用户自己定义了一个类java.lang.Yes，并用用户自定义的类装载器装载，由于java.lang.Yes和核心类库java.lang.*由不同的装载器装载，它们属于不同的运行时包，所以java.lang.Yes不能访问核心类库java.lang中类的包可见的成员。

(7)有关ClassLoader的重载

  扩展ClassLoader方法
  
我们目的是从本地文件系统使用我们实现的类装载器装载一个类。为了创建自己的类装载器我们应该扩展ClassLoader类，这是一个抽象类。我们创建一个FileClassLoader extends ClassLoader。我们需要覆盖ClassLoader中的findClass(String name)方法，这个方法通过类的名字而得到一个Class对象。

     	public  Class findClass(String name)    {
         byte [] data  =  loadClassData(name);
         return  defineClass(name, data,  0 , data.length);
    	}

   我们还应该提供一个方法loadClassData(String name)，通过类的名称返回class文件的字节数组。然后使用ClassLoader提供的defineClass()方法我们就可以返回Class对象了。
     
     public   byte [] loadClassData(String name)    {
        FileInputStream fis  =   null ;
         byte [] data  =   null ;
         try  {
            fis  =   new  FileInputStream( new  File(drive  +  name  +  fileType));
            ByteArrayOutputStream baos  =   new  ByteArrayOutputStream();
             int  ch  =   0 ;
             while  ((ch  =  fis.read())  !=   - 1 )  {
                baos.write(ch);              
            }
            data  =  baos.toByteArray();
        }  catch  (IOException e)  {
            e.printStackTrace();
        }       
         return  data;
    }