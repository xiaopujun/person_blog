# 内部类

内部类指的是在类的里面还有类的存在。Java中内部类主要分为成员内部类（非静态内部类、静态内部类）、匿名内部类、局部内部类

## 成员内部类

```java
class Outer {
    private String name;
    private int age;

    /**
     * 1. 成员内部类可以直接访问外部类的成员，包括私有
     */
    class Inner {
        private String value;
        private int id;

        //编译报错：非静态内部类不能有静态成员，因为非静态内部类的对象依赖于外部类的对象；放一个静态成员，就破坏了这种依赖关系
        static int a = 10;

        public void show() {
            System.out.println(name);
            System.out.println(age);
        }
    }

    /**
     * 2. 外部类要访问内部类的成员，必须创建对象
     */
    public void test() {
        Inner inner = new Inner();
        System.out.println(inner.id);
        System.out.println(inner.value);
    }

    public static void staticTest() {
        //不能直接访问内部类的成员，因为内部类的对象依赖于外部类的对象，此时对象还没有创建
    }
}
```

非静态内部类的特点

- 非静态内部类可以直接访问外部类的成员，但是外部类不能直接访问非静态内部类成员
- 非静态内部类不能有静态方法、静态属性、静态初始化块儿
- 外部类的静态方法、静态代码块不能访问非静态内部类、包括不能使用非静态内部类定义变量和创建实例

#### 创建内部类对象

```java
public class Demo01 {
    String name;

    //非静态内部类
    class Inner {
        int age;
    }
}

class Demo02 {
    public static void main(String[] args) {
        Demo01.Inner inner = new Demo01().new Inner();
    }
}
```

## 静态内部类

```java
public class Demo {
    public static void main(String[] args) {
        //创建静态内部类对象，通过外部类.内部类的方式创建
        Outer.Inner inner = new Outer.Inner();
        //通过外部类.内部类.静态成员的方式可以“直接”访问静态成员
        Outer.Inner.a = 20;
        //访问静态内部类的普通成员，还是需要创建内部类对象后才能访问
        System.out.println(inner.id);
    }
}

class Outer {
    private String name;
    private int age;

    /**
     * 1. 成员内部类可以直接访问外部类的成员，包括私有
     */
    static class Inner {
        String value;
        int id;
        static int a = 10;

        public void show() {
            //编译报错，静态内部类不能直接访问外部类的成员，因为静态内部类不依赖于外部类的对象，没法“直接”访问外部类的成员
            //System.out.println(name);
            //System.out.println(age);
        }
    }

    /**
     * 2. 外部类要访问内部类的成员，必须创建对象
     */
    public void test() {
        Inner inner = new Inner();
        System.out.println(inner.id);
        System.out.println(inner.value);
    }

    public static void staticTest() {
        //不能直接访问内部类的成员，因为内部类的对象依赖于外部类的对象，此时对象还没有创建
    }
}
```

特点：

- 当静态内部类对象存在时，不一定外部类的对象存在，因为静态内部类不依托于外部类，因此静态内部类的方法不能直接访问外部类的方法
- 静态内部类看作一个外部类的静态成员，因此外部类的方法可以通过：静态内部类.名字 的方式访问静态内部类的静态成员。通过new静态内部类访问静态内部类的实例

### 匿名内部类

适合那些只需使用一次的类

```java
new 父类构造器/实现的接口(){

        }
```

具体代码

```java
public interface Demo01 {
    String hello();
}

class Demo02 {
    public static void main(String[] args) {
        Demo02 demo02 = new Demo02();
        demo02.say(new Demo01() {
            @Override
            public String hello() {
                return null;
            }
        });
    }

    public void say(Demo01 demo) {
        System.out.println("say hello");
    }
}
```

如上代码，在第8行的时候，new一个Demo01的时候默认就实现了Demo01这个接口，并且直接写出方法体，实现了接口的具体方法。因为这个类只需用一次，故就不用定义它的名字等其他内容。此为匿名内部类。

### 局部内部类

定义在方法中的内部类，作用域范围仅限于方法内部，用的极少。