1. 实现一个自定义的classloader，加载如下的文件，内容需要解码，读取的字节码需要解码，解码方式：255减去原有值，并执行成功。

```
 import java.io.BufferedInputStream;
 import java.io.ByteArrayOutputStream;
 import java.io.File;
 import java.io.FileInputStream;
 import java.io.FileNotFoundException;
 import java.io.IOException;
 
 /**
  *
  *
  * @author chenlx
  * @version CustomClassLoader.java, v 0.1 2020-11-03 18:47 chenlx
  */
 public class CustomClassLoader extends ClassLoader{
 
     private String filePath;
 
     public CustomClassLoader(String filePath) {
         this.filePath = filePath;
     }
 
     /**
      * 沿用双亲委派模型
      *
      * @param name
      * @return
      * @throws ClassNotFoundException
      */
     @Override
     protected Class<?> findClass(String name) throws ClassNotFoundException {
         try{
             // 加密的字节数组
             byte[] encodeByte = getClassByteArr(filePath);
             // 解密字节数组
             byte[] decodeByte = new byte[encodeByte.length];
             for (int i = 0; i < encodeByte.length; i++) {
                 decodeByte[i] = (byte) ((byte)255 - encodeByte[i]);
             }
             Class<?> c = this.defineClass(name, decodeByte, 0, decodeByte.length);
             return c;
         } catch (Exception e) {
             e.printStackTrace();
         }
         return super.findClass(name);
     }
 
     /**
      *
      * @param filePath
      * @return
      * @throws IOException
      */
     public static byte[] getClassByteArr(String filePath) throws IOException {
 
         File f = new File(filePath);
         if (!f.exists()) {
             throw new FileNotFoundException(filePath);
         }
 
         ByteArrayOutputStream bos = new ByteArrayOutputStream((int) f.length());
         BufferedInputStream in = null;
         try {
             in = new BufferedInputStream(new FileInputStream(f));
             int buf_size = 1024;
             byte[] buffer = new byte[buf_size];
             int len = 0;
             while (-1 != (len = in.read(buffer, 0, buf_size))) {
                 bos.write(buffer, 0, len);
             }
             return bos.toByteArray();
         } catch (IOException e) {
             e.printStackTrace();
             throw e;
         } finally {
             try {
                 in.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
             bos.close();
         }
     }
 }
 ```

```
import java.lang.reflect.Method;

/**
 *
 *
 * @author chenlx
 * @version Demo.java, v 0.1 2020-11-03 19:03 chenlx
 */
public class Demo {

    public static void main(String[] args) throws Exception {
        String filePath = "src/main/resources/Hello.xlass";
        CustomClassLoader customClassLoader = new CustomClassLoader(filePath);
        Class<?> clazz = Class.forName("Hello", true, customClassLoader);
        Object o = clazz.newInstance();
        // 获取构造器函数
        Method method = clazz.getMethod("hello");
        method.invoke(o);
    }
}
```

```
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA CE.app/Contents/lib/idea_rt.jar=60147:/Applications/IntelliJ IDEA CE.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/lib/tools.jar:/Users/longxiang/java-demo/target/classes com.demo.Demo
Hello, classLoader!

Process finished with exit code 0
```

2. 分析以下GC日志，尽可能详细的标注出GC发生时相关的信息。

```
2020-10-29T21:19:19.488+0800: 114.015: [GC (CMS Initial Mark) [1 CMS-initial-mark: 106000K(2097152K)] 1084619K(3984640K), 0.2824583 secs] [Times: user=0.86 sys=0.00, real=0.28 secs]
-- 2020-10-29T21:19:19.488+0800 : 表示 CMS Initial Mark 初始标记 开始时间，触发第一次 STW
-- 114.015 : 指 CMS Initial Mark  开始的时间相对于 JVM 启动的时间，单位：秒，即在 JVM 启动 114.015 秒后发生一次 CMS Initial Mark
-- GC : 表示此次是 Minor GC, 触发条件：Eden 区满了。 还有一种是 Full GC。
-- CMS Initial Mark ：表示是 CMS 垃圾收集器，处于 Initital Mark 阶段，标记所有根对象
-- 106000K : 表示当前老年代已使用容量
-- 2097152K : 表示当前老年代剩余可用容量
-- 1084619K : 表示当前堆的已使用容量
-- 3984640K : 表示整个堆的容量
-- 0.2824583 secs : 标记阶段所花时间
-- Times: user=0.86 sys=0.00, real=0.28 secs
user – Total CPU time that was consumed by Garbage Collector threads during this collection
sys – Time spent in OS calls or waiting for system event
real – Clock time for which your application was stopped. With Parallel GC this number should be close to (user time + system time) divided by the number of threads used by the Garbage Collector. In this particular case 8 threads were used. Note that due to some activities not being parallelizable, it always exceeds the ratio by a certain amount.

2020-10-29T21:19:19.771+0800: 114.298: [CMS-concurrent-mark-start]
-- 2020-10-29T21:19:19.771+0800 : 表示 CMS concurrent-mark-start 并发标记 开始时间
-- 114.298 : 指 CMS 并发标记开始的时间相对于 JVM 启动的时间，单位：秒，即在 JVM 启动 114.298 秒后发生一次 CMS 并发标记

2020-10-29T21:19:19.931+0800: 114.458: [CMS-concurrent-mark: 0.160/0.160 secs] [Times: user=0.32 sys=0.03, real=0.16 secs]
-- CMS-concurrent-mark: 0.160/0.160 secs : 第一个 0.160 表示并发标记耗时 第二个 0.160 是时钟时间

2020-10-29T21:19:19.931+0800: 114.459: [CMS-concurrent-preclean-start]
-- 表示 并发预清理阶段开始，

2020-10-29T21:19:19.998+0800: 114.525: [CMS-concurrent-preclean: 0.065/0.066 secs] [Times: user=0.05 sys=0.01, real=0.06 secs]
-- CMS-concurrent-preclean: 0.065/0.066 secs : 第一个 0.065 表示并发预清理耗时 第二个 0.066 是时钟时间

2020-10-29T21:19:19.998+0800: 114.525: [CMS-concurrent-abortable-preclean-start]CMS: abort preclean due to time 
-- CMS-concurrent-abortable-preclean-start : 可终止的并发预清理，尝试承担 Final Remark 阶段工作，减少 Final Remark 的 STW 时间

2020-10-29T21:19:25.072+0800: 119.599: [CMS-concurrent-abortable-preclean: 5.038/5.073 secs] [Times: user=7.72 sys=0.50, real=5.08 secs]
-- CMS-concurrent-abortable-preclean: 5.038/5.073 secs : 5.038 表示耗时

2020-10-29T21:19:25.076+0800: 119.603: [GC (CMS Final Remark) [YG occupancy: 1279357 K (1887488 K)]
-- CMS Final Remark : 最终标记阶段，第二次 STW 
-- YG occupancy: 1279357 K (1887488 K) : 年轻代当前已使用容量和总的容量

2020-10-29T21:19:25.076+0800: 119.603: [Rescan (parallel) , 0.3120602 secs]
-- STW 时重新标记存活对象（并发标记）, 共耗时0.3120602秒

2020-10-29T21:19:25.388+0800: 119.915: [weak refs processing, 0.0015920 secs]
-- 弱引用处理，共耗时0.0015920秒

2020-10-29T21:19:25.390+0800: 119.917: [class unloading, 0.0517863 secs]
-- 无用类卸载，共耗时0.0517863秒

2020-10-29T21:19:25.441+0800: 119.969: [scrub symbol table, 0.0212825 secs]
-- 清理 symbol talbe 共耗时0.0212825秒

2020-10-29T21:19:25.463+0800: 119.990: [scrub string table, 0.0022435 secs][1 CMS-remark: 106000K(2097152K)] 1385358K(3984640K), 0.3959182 secs] [Times: user=1.33 sys=0.00, real=0.40 secs]
-- scrub string table, 0.0022435 secs :
-- CMS-remark: 106000K(2097152K) : Final Remark 后，老年代已使用的容量和老年代总容量
-- 1385358K(3984640K) :  Final Remark 后，整个堆已使用的容量和堆总容量

2020-10-29T21:19:25.473+0800: 120.000: [CMS-concurrent-sweep-start]
-- CMS-concurrent-sweep-start : 并发清除阶段开始

2020-10-29T21:19:25.540+0800: 120.067: [CMS-concurrent-sweep: 0.067/0.067 secs] [Times: user=0.18 sys=0.02, real=0.06 secs]
-- CMS-concurrent-sweep: 0.067/0.067 secs : 第一个0.067 并发清除阶段耗时

2020-10-29T21:19:25.540+0800: 120.068: [CMS-concurrent-reset-start]
-- CMS-concurrent-reset-start : 并发重置阶段开始

2020-10-29T21:19:25.544+0800: 120.071: [CMS-concurrent-reset: 0.003/0.003 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
-- CMS-concurrent-reset: 0.003/0.003 secs : 第一个 0.003 是并发重置耗时

```

3. 标注以下启动参数每个参数的含义

```
java 
-Denv=PRO : 指定 Spring Boot profile 环境 PRO
-server : 服务端模式
-Xms4g : 指定堆初始值4G
-Xmx4g : 指定堆最大值4G
-Xmn2g : 指定新生代最大值2G
-XX:MaxDirectMemorySize=512m : 设置 NIO direct-buffer 最大为 512m
-XX:MetaspaceSize=128m : 设置元空间为 128m
-XX:MaxMetaspaceSize=512m  : 设置元空间最大为 512m
-XX:-UseBiasedLocking : 禁用偏向锁
-XX:-UseCounterDecay : 禁用JIT调用计数器衰减
-XX:AutoBoxCacheMax=10240 : 设置Integer缓存自动装箱范围，最大由默认的 127 变为 10240
-XX:+UseConcMarkSweepGC : 使用 CMS 垃圾收集器
-XX:CMSInitiatingOccupancyFraction=75 : CMS 垃圾收集器当老年代达到75%时触发GC
-XX:+UseCMSInitiatingOccupancyOnly : 使用 CMSInitiatingOccupancyFraction 设定的值
-XX:MaxTenuringThreshold=6 : 新生代中的对象被标记6次后晋升老年代
-XX:+ExplicitGCInvokesConcurrent : 表示JVM无论什么时候调用系统GC，都执行CMS GC，而不是Full GC
-XX:+ParallelRefProcEnabled : 让GC的时候尽可能并行处理
-XX:+PerfDisableSharedMem : jps 命令将查看不到进程
-XX:+AlwaysPreTouch : -Xms指定的堆内存中每个字节都写入'0'，这样的话，除了在虚拟内存中以内部数据结构保留之外，还会在物理内存中分配。并且由于touch这个行为是单线程的，因此它将会让JVM进程启动变慢。
-XX:-OmitStackTraceInFastThrow : fast throw，当一些异常在代码里某个特定位置被抛出很多次的话，HotSpot Server Compiler（C2）会用fast throw来优化这个抛出异常的地方，直接抛出一个事先分配好的、类型匹配的对象，这个对象的message和stack trace都被清空。
-XX:+ExplicitGCInvokesConcurrent : 重复了
-XX:+ParallelRefProcEnabled : 重复了
-XX:+HeapDumpOnOutOfMemoryError : 设置内存溢出时保存打印堆栈日志
-XX:HeapDumpPath=/home/devjava/logs/ : dump 日志保存目录
-Xloggc:/home/devjava/logs/lifecircle-tradecore-gc.log : gc日志目录
-XX:+PrintGCApplicationStoppedTime  : GC时打印应用暂停时间
-XX:+PrintGCDateStamps : GC时打印时间戳信息
-XX:+PrintGCDetails : GC时打印更多详细信息
-javaagent:/home/devjava/ArmsAgent/arms-bootstrap-1.7.0-SNAPSHOT.jar ： arms jar
-jar /home/devjava/lifecircle-tradecore/app/lifecircle-tradecore.jar ： 应用 jar
```