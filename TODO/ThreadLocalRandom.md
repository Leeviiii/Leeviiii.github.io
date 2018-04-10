## Random 与 ThreadLocalRandom
《阿里巴巴JAVA开发手册》有这样一段话
>【推荐】避免Random实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一seed 导致的性能下降。 说明：Random实例包括java.util.Random 的实例或者 Math.random()的方式。 正例：在JDK7之后，可以直接使用API ThreadLocalRandom，而在 JDK7之前，需要编码保证每个线程持有一个实例。

之前没有意识到这问题，看到了就写写代码总结一下。代码很简单直接开10个thread,每个thread循环一定次数的random操作，打印时间看结果。

```
import java.util.Random;
import java.util.concurrent.ThreadLocalRandom;
public class RandomThread extends Thread {
	@Override
	public void run() {
		long begint = System.currentTimeMillis();
		ThreadLocalRandom r = ThreadLocalRandom.current();
		//Random r = new Random(); // 150ms
		for (int i = 0 ; i < 1000000 ; i++ ) {
			//Math.random(); //  2.3s
			r.nextDouble(); // 45ms
		}
		long endt = System.currentTimeMillis();
		System.out.println(this.getName() + "costtime:" + (endt - begint));
	}
}

import java.util.Random;
public class Test {
	public static void main(String[] argvs) throws java.lang.InterruptedException {
		int tasknum = 10;
		Thread[] t = new Thread[tasknum];
		for (int i = 0; i < 10 ; i++) {
			t[i] = new RandomThread();
			t[i].setName("name:" + i);
		}
		long a1 = System.currentTimeMillis();
		for (int i = 0; i < 10 ; i++) {
			t[i].start();
		}
		for (int i = 0; i < 10 ; i++) {
			t[i].join();
		}
		long a2 = System.currentTimeMillis();
		System.out.println("costtime:" + (a2 - a1));
	}

}
```

上面的代码分别以三种方式进行测试
>1. 线程共享一个随机种子，采用Math.random进行测试，运行时间2000多毫秒
>2. 每一个线程独立一个Random实例，运行时间150毫秒左右
>3. 直接使用1.8的ThreadLocalRandom，运行时间只有45毫秒左右，比第一张方式快了四五十倍，很客观。

TODO:有空看看内部实现逻辑