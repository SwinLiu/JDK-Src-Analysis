
# 源码摘要

```java
package java.lang;

public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    public final native Class<?> getClass();

    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }

    protected native Object clone() throws CloneNotSupportedException;

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public final native void notify();

    public final native void notifyAll();

    public final native void wait(long timeout) throws InterruptedException;

    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }

    public final void wait() throws InterruptedException {
        wait(0);
    }

    protected void finalize() throws Throwable { }
}

```

# 源码解析：

Object类是Java中其他所有类的祖先。

## 构造函数

未定义构造函数,默认一个无参数的构造函数,`public Object(){};`

## 属性
无

## 方法

### `private static native void registerNatives();`

registerNatives函数前面有native关键字修饰，Java中，用native关键字修饰的函数表明该方法的实现并不是在Java中去完成，而是由C/C++去完成，并被编译成了.dll，由Java去调用。方法的具体实现体在dll文件中，对于不同平台，其具体实现应该有所不同。用native修饰，即表示操作系统，需要提供此方法，Java本身需要使用。具体到registerNatives()方法本身，其主要作用是将C/C++中的方法映射到Java中的native方法，实现方法命名的解耦。

Object 对象创建时调用注册。 由静态方法在初始化执行。
```java
    private static native void registerNatives();
    static {
        registerNatives();
    }
```

### `public final native Class<?> getClass();`
native的方法

### `public native int hashCode();`
native的方法

### `equals`方法
```java
	public boolean equals(Object obj) {
        return (this == obj);
    }
```

### `protected native Object clone() throws CloneNotSupportedException;`
native的方法。Java术语表述为：clone函数返回的是一个引用，指向的是新的clone出来的对象，此对象与原对象分别占用不同的堆空间。


> [Java总结篇系列：java.lang.Object](https://www.cnblogs.com/lwbqqyumidi/p/3693015.html)中的案例
明白了clone的含义后，接下来看看如果调用clone()函数对象进行此克隆操作。
首先看一下下面的这个例子：
```java
package com.corn.objectsummary;

import com.corn.Person;

public class ObjectTest {

    public static void main(String[] args) {

        Object o1 = new Object();
        // The method clone() from the type Object is not visible
        Object clone = o1.clone();
    }

}
```

例子很简单，在main()方法中，new一个Oject对象后，想直接调用此对象的clone方法克隆一个对象，但是出现错误提示："The method clone() from the type Object is not visible"

why? 根据提示，第一反应是ObjectTest类中定义的Oject对象无法访问其clone()方法。回到Object类中clone()方法的定义，可以看到其被声明为protected，估计问题就在这上面了，protected修饰的属性或方法表示：__在同一个包内或者不同包的子类可以访问__。显然，Object类与ObjectTest类在不同的包中，但是ObjectTest继承自Object，是Object类的子类，于是，现在却出现子类中通过Object引用不能访问protected方法，原因在于对"不同包中的子类可以访问"没有正确理解。

__"不同包中的子类可以访问"，是指当两个类不在同一个包中的时候，继承自父类的子类内部且主调（调用者）为子类的引用时才能访问父类用protected修饰的成员（属性/方法）。 在子类内部，主调为父类的引用时并不能访问此protected修饰的成员。！（super关键字除外）__

于是，上例改成如下形式，我们发现，可以正常编译：
```java
package com.corn.objectsummary;

public class ObjectTest {

    public static void main(String[] args) {
        ObjectTest ot1 = new ObjectTest();

        try {
            ObjectTest ot2 = (ObjectTest) ot1.clone();
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}
```
是的，因为此时的主调已经是子类的引用了。

上述代码在运行过程中会抛出"java.lang.CloneNotSupportedException",表明clone()方法并未正确执行完毕，问题的原因在与Java中的语法规定：

clone()的正确调用是需要实现Cloneable接口，如果没有实现Cloneable接口，并且子类直接调用Object类的clone()方法，则会抛出CloneNotSupportedException异常。

Cloneable接口仅是一个表示接口，接口本身不包含任何方法，用来指示Object.clone()可以合法的被子类引用所调用。

于是，上述代码改成如下形式，即可正确指定clone()方法以实现克隆。
```java
package com.corn.objectsummary;

public class ObjectTest implements Cloneable {

    public static void main(String[] args) {

        ObjectTest ot1 = new ObjectTest();

        try {
            ObjectTest ot2 = (ObjectTest) ot1.clone();
            System.out.println("ot2:" + ot2);
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}
```


### `toString`方法
```java
	public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```


### `public final native void notify();`
native的方法。

### `public final native void notifyAll();`
native的方法。

### `wait`方法

`public final native void wait(long timeout) throws InterruptedException;`
native的方法。

```java
	public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
java


```java
    public final void wait() throws InterruptedException {
        wait(0);
    }
```



### `protected void finalize() throws Throwable { }`




参考文献：
[https://www.cnblogs.com/lwbqqyumidi/p/3693015.html](https://www.cnblogs.com/lwbqqyumidi/p/3693015.html)



