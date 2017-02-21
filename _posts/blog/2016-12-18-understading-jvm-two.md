---
layout: post
title: 深入理解Java虚拟机（二）——Java内存溢出
categories: 深入理解JVM
description: 深入理解Java虚拟机（二）——Java内存溢出
keywords: Java虚拟机，JVM
---

按照Java内存的结构，发生内存溢出的地方常在于**堆、栈、方法区、直接内存**。

## 1、堆溢出

堆溢出原因莫过于对象太多导致，看代码。

```java
import java.util.ArrayList;  
import java.util.List;  
  
/** 
 * java 堆溢出 
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError 
 */  
public class HeapOOM {  
  
    static class OOMObject {  
    }  
  
    public static void main(String[] args) {  
        List<OOMObject> list = new ArrayList<OOMObject>();  
          
        while (true) {  
            list.add(new OOMObject());  
        }  
    }  
}  
  
/** 
 * java.lang.OutOfMemoryError: Java heap space 
Dumping heap to java_pid1820.hprof ... 
Heap dump file created [24787111 bytes in 0.346 secs] 
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space 
    at java.util.Arrays.copyOf(Arrays.java:2760) 
    at java.util.Arrays.copyOf(Arrays.java:2734) 
    at java.util.ArrayList.ensureCapacity(ArrayList.java:167) 
    at java.util.ArrayList.add(ArrayList.java:351) 
    at baby.oom.HeapOOM.main(HeapOOM.java:19) 
 *  
 * 
*/  
```
 
## 2、栈溢出

根据JAVA虚拟机规范描述：

1. 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError
2. 如果虚拟机在扩展栈时无法申请到足够的内存空间，将抛出OutOfMemoryError。

实验表明：
在单线程下，无论是由于栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverflowError。

通过不断的建立新线程的方式可以产生内存溢出。为每个线程的栈分配的内存越大，反而越容易产生内存溢出异常。

如果是建立过多线程导致的内存溢出，在不能减少线程数量或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。

假设32位windows系统虚拟机最大设为2G，虚拟机提供了参数来控制java堆和方法区这两部分最大值，如果虚拟机本身进程内存大小不算在内，省下的内存就有虚拟机和本地方法栈瓜分了。每个线程分配到的栈容量越大，可以建立的线程数量自然就越少。

```java
/** 
 * 栈异常 
 * 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError 
 * 如果虚拟机在扩展栈时无法申请到足够的内存空间，将抛出OutOfMemoryError 
 * VM Args：-Xss128k 
 */  
public class JavaVMStackSOF {  
  
    private int stackLength = 1;  
  
    public void stackLeak() {  
        stackLength++;  
        stackLeak();  
    }  
  
    public static void main(String[] args) throws Throwable {  
        JavaVMStackSOF oom = new JavaVMStackSOF();  
        try {  
            oom.stackLeak();  
        } catch (Throwable e) {  
            System.out.println("stack length:" + oom.stackLength);  
            throw e;  
        }  
    }  
}  
  
/** 
 *  
 * stack length:2403 
Exception in thread "main" java.lang.StackOverflowError 
    at baby.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11) 
    at baby.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:12) 
    at baby.oom.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:12) 
     
    默认情况下，即不加Xss限制，输出的length为8956，加了Xss128k length位2403 
 */  
```
 
```java
/** 
 * VM Args：-Xss2M （这时候不妨设大些） 
 */  
public class JavaVMStackOOM {  
   
       int i=0;  
       private void dontStop() {  
              while (true) {  
              }  
       }  
   
       public void stackLeakByThread() {  
             
              while (true) {  
                     Thread thread = new Thread(new Runnable() {  
                            @Override  
                            public void run() {  
                                   dontStop();  
                            }  
                     });  
                     i++;  
                     System.out.println("i="+i);  
                     thread.start();  
              }  
       }  
   
       public static void main(String[] args) throws Throwable {  
              JavaVMStackOOM oom = new JavaVMStackOOM();  
               
              try {  
                  oom.stackLeakByThread();  
            } catch (Throwable e) {  
                System.out.println("thread num:" + oom.i);  
                throw e;  
            }  
       }  
}  
//i=391  
//thread num:391  
//Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread  
```

 
## 3、方法区溢出

当运行时常量池过大或者类过多时就会导致方法区溢出。


```java  
import java.util.ArrayList;  
import java.util.List;  
  
/** 
 * VM Args：-XX:PermSize=10M -XX:MaxPermSize=10M 
 * @author  
 */  
public class RuntimeConstantPoolOOM {  
  
    public static void main(String[] args) {  
        // 使用List保持着常量池引用，避免Full GC回收常量池行为  
        List<String> list = new ArrayList<String>();  
        // 10MB的PermSize在integer范围内足够产生OOM了  
        int i = 0;   
        while (true) {  
            list.add(String.valueOf(i++).intern());  
        }  
    }  
}  
  
/** 
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space 
    at java.lang.String.intern(Native Method) 
    at baby.oom.RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java:18) 
 
*/  
```
 
```java  
import java.lang.reflect.Method;  
  
import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
/** 
 * VM Args： -XX:PermSize=10M -XX:MaxPermSize=10M 
 * @author  
 */  
public class JavaMethodAreaOOM {  
  
    public static void main(String[] args) {  
        while (true) {  
            Enhancer enhancer = new Enhancer();  
            enhancer.setSuperclass(OOMObject.class);  
            enhancer.setUseCache(false);  
            enhancer.setCallback(new MethodInterceptor() {  
              
                @Override  
                public Object intercept(Object obj, Method method, Object[] arg, MethodProxy proxy) throws Throwable {  
                    // TODO Auto-generated method stub  
                    return proxy.invokeSuper(obj, arg);  
                }  
            });  
            enhancer.create();  
        }  
    }  
  
    static class OOMObject {  
  
    }  
}  
  
/* 
 * Exception in thread "main" net.sf.cglib.core.CodeGenerationException: java.lang.reflect.InvocationTargetException-->null 
    at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:237) 
    at net.sf.cglib.proxy.Enhancer.createHelper(Enhancer.java:377) 
    at net.sf.cglib.proxy.Enhancer.create(Enhancer.java:285) 
    at baby.oom.JavaMethodAreaOOM.main(JavaMethodAreaOOM.java:28) 
Caused by: java.lang.reflect.InvocationTargetException 
    at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source) 
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25) 
    at java.lang.reflect.Method.invoke(Method.java:597) 
    at net.sf.cglib.core.ReflectUtils.defineClass(ReflectUtils.java:384) 
    at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:219) 
    ... 3 more 
Caused by: java.lang.OutOfMemoryError: PermGen space 
    at java.lang.ClassLoader.defineClass1(Native Method) 
    at java.lang.ClassLoader.defineClassCond(ClassLoader.java:631) 
    at java.lang.ClassLoader.defineClass(ClassLoader.java:615) 
    ... 8 more 
 */  
```

 
## 4、直接内存溢出

虽然使用DerictByteBuffer分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配，而是通过计算得知内存无法分配，于是手动抛出异常，真正申请分配内存的方法是unsafe.allocateMemory()。

```java  
import java.lang.reflect.Field;  
      
import sun.misc.Unsafe;   
/** 
 * VM Args：-Xmx20M -XX:MaxDirectMemorySize=10M 
 * @author  
 * Eclipse 默认把这些受访问限制的API设成了ERROR。 
 
解决办法：将Windows->Preferences->Java-Complicer->Errors/Warnings->Deprecated and restricted API，中的Forbidden references(access rules)设置为Warning，即可以编译通过。 
 
 */  
public class DirectMemoryOOM {  
  
    private static final int _1MB = 1024 * 1024;  
  
    public static void main(String[] args) throws Exception {  
          
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];  
        unsafeField.setAccessible(true);  
        Unsafe unsafe = (Unsafe) unsafeField.get(null);  
        while (true) {  
            unsafe.allocateMemory(_1MB);  
        }  
    }  
}  
  
/** 
Exception in thread "main" java.lang.OutOfMemoryError 
    at sun.misc.Unsafe.allocateMemory(Native Method) 
    at baby.oom.DirectMemoryOOM.main(DirectMemoryOOM.java:20) 
*/  
```