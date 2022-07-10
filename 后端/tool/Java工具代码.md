# 倒计时原理

模拟倒计时的时候可以使用线程休眠的方式

```java
public class Countdown {

    private static int counter = 10;

    public static void main(String[] args) {
        do {
            System.out.println("倒计时：" + counter--);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } while (counter != 0);
    }

}
```

# 显示当前时间原理

显示当前时间也可以使用线程休眠指定时间来办到

```java
public class CurrentTime {
    public static void main(String[] args) {
        Date time = new Date(System.currentTimeMillis());
        while (true){
            try {
                Thread.sleep(1000);
                System.out.println(new SimpleDateFormat("HH:mm:ss").format(time));
                time = new Date(System.currentTimeMillis());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```