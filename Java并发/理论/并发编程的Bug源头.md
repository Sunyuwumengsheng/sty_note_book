### 可见性

__概念：线程对共享变量的修改其他线程可以立即看到即可见性__

![[Pasted image 20231130102650.png]]
多cpu架构中cpu-1中的变量  对cpu2不可见

### 原子性问题

__概念：是指一个操作或一组操作被视为不可分割的单个单位，要么全部执行成功，要么全部失败回滚，在编程中值描述操作不可分割，如 在多线程中原子操作是指 可以在单个步骤中完成的操作，不会被其他线程中断和干扰__

例子： count +=1  执行最少需要三条指令
```java
	public class Test {
  private long count = 0;
  private void add10K() {
    int idx = 0;
    while(idx++ < 10000) {
      count += 1;
    }
  }
  public static long calc() {
    final Test test = new Test();
    // 创建两个线程，执行add()操作
    Thread th1 = new Thread(()->{
      test.add10K();
    });
    Thread th2 = new Thread(()->{
      test.add10K();
    });
    // 启动两个线程
    th1.start();
    th2.start();
    // 等待两个线程执行结束
    th1.join();
    th2.join();
    return count;
  }
}
```



1. count从内存加载到CPU的寄存器
2. 在寄存器中执行 +1操作
3. 将结果写入内存
线程切换是时机可能在2之前  这会导致  th1 取到的值是0  th2取到的值也是0 th1  count +1  得到1 写入内存 是1   th2也是 0+1 =1 写入内存  也是1 这样 执行了两次加的操作得到是1而不是2这就是线程切换导致的问题  因此  +=并不是原子性的操作 

### 有序性问题
__程序有序性：是指程序按照代码的先后顺序执行 ，编译器为了优化性能有时后会改变程序中的先后顺序  如：a=1,b=2 变为了b=2,a=1

例子：java单列模式中创建双重校验创建单列对象

```java
public class Singleton {
  static Singleton instance;
  static Singleton getInstance(){
    if ( instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
}
```


假设A,B两个线程  同时调用  getInstance   [[锁#^1a6b8c|使用Singleton.class]] ，只会保证一个线程能够加锁成功 假设A加锁成功 B等待  A创建实例后释放锁  B加锁后发现instance不为空返回创建过的实例 但是可能会出现一种情况  编译器优化  new Singleton
new的操作流程因该是这样
1. 分配内存  M
2. 在内存M上初始化Singleton
3. 将M的地址赋给instance
实际上优化的路径为
132  先将m的地址赋给instance 
指令的重排序（列子：单列模式的双重检测，new指令也是3步操作，①分内存②初始化③赋值给引用变量，可能会发生①③②的重排序，这时候如果又有操作系统的分时操作的加持，导致A操作①③后挂起，时间片被分配给了B线程，而B线程甚至都不需要进行锁的获取，因为此时instance已经不等于null了，但是此时的instance可能未初始化）
修改方案  添加volatile 采用饿汉式写法 
```java
public class Singleton {
  static volatile Singleton instance;
  static Singleton getInstance(){
    if ( instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    return instance;
  }
}
//饿汉式写法
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHolder {
        private static final Singleton instance = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}

```





