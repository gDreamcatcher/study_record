## 引论

1. java5支持泛型机制，自动装箱和拆箱
    自动拆箱和装箱主要是针对基础类型而言的
    自动装箱：如果一个int型变量被传递到一个需要Integer对象的地方，那么编译器就会自动插入一个对Integer构造方法的调用
    自动拆箱：如果一个Inerger对象放到了需要int的地方，编译器会自动插入一个对intValue方法的调用
2. 泛型类和泛型接口
    一般做包装类，为某些类添加一些属性和方法
3. 泛型方法  没搞明白
4. 类型擦除
    泛型在很大程度上是Java语言的成分而不是虚拟机的结构，泛型类可以由编译器通过所谓的**类型擦除**转变成非泛型的类，由此生成与泛型类同名的**原始类**。
5. 泛型的限制
    基本类型不能做类型参数
    instanceof检测工作只对原始类进行
    static方法和static域均不可以引用类的类型变量
    不能创建一个泛型类型的实例，也不可以创建一个泛型的数组
    
    
    
## 算法分析

## 表、栈和队列

### 嵌套类

嵌套类是指定义在一个类内部的类，包括静态内部类，非静态内部类，匿名内部类和局部内部类。
嵌套类的用处：

1. java不支持多继承，可以使用多个内部类变相的实现多继承；
2. 内部类可以访问外部类的所有对象和方法
3. 控制类之间的可见性；
4. 方便对接口实现自由的定义--匿名内部类；

```java
class Outter{
    // 非静态内部类，内部不可以有static修饰的对象，方法和类
    class Inner{};
    // 静态内部类
    static class StaticInner{};

    public static void test() {
        // 创建非静态内部类对象
        Outter outter = new Outter();
        Outter.Inner inner = outter.new Inner();

        // 创建静态内部类对象
        Outter.StaticInner staticInner = new Outter.StaticInner();

        // 匿名内部类
        new Thread(new Runnable(){
            @Override
            public void run() {
                System.out.println("Thread start");
            }
        }).start();

        // 局部内部类
        class MyTask implements Runnable{
            @Override
            public void run(){
                System.out.println("局部内部类");
            }
        };
        new Thread(new MyTask()).start();
    }
}
```