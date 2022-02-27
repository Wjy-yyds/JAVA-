## 适配器分析

### 适配器的作用
将一个类的接口转化为我们希望使用的接口——接口的转换

### 实现过程
* 实现目标接口
* 内部持有一个待转换接口的引用
* 在目标接口的实现方法内部，调用待转换接口的方法

### 例子实现

定义一个实现接口的类，想通过一个线程来执行这个类

```
public class Task implements Callable<Long> {
    private long num;
    public Task(long num) {
        this.num = num;
    }

    public Long call() throws Exception {  //注意这是Task实现Callable接口的方法
        long r = 0;
        for (long n = 1; n <= this.num; n++) {
            r = r + n;
        }
        System.out.println("Result: " + r);
        return r;
    }
}
```
尝试用`Thread`执行该类
```
Callable<Long> callable = new Task(123450000L);
Thread thread = new Thread(callable); // compile error!
thread.start();
```

无法执行通过，因为`Thread`类只接受`Runnable`接口

**定义一个`adapter`来转换接口类型**

```
public class RunnableAdapter implements Runnable { //通过实现 Runnable 类来转换接口类型？
    // 引用待转换接口:
    private Callable<?> callable;

    public RunnableAdapter(Callable<?> callable) {
        this.callable = callable;
    }

    // 实现指定接口:
    public void run() {
        // 将指定接口调用委托给转换接口调用:
        try {
            callable.call();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```
*注意`Runnable`利用反射来在一个与`Task` 和 `Callable`都不相关的类中成功的使用了 `Callable`类要实现的方法从而达到‘接口的转换’*