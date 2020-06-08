
spring是轻量级开源框架，解决企业开发的复杂性，两个核心部分IOC(控制反转，把创建对象的过程交给spring进行管理，目的是降低耦合度)和AOP(面向切面编程，不修改源代码对应用功能增强)。

IOC：xml解析，工厂模式（BeanFactory），反射

class clazz = Class.forName("class_name");
return (UserBean)clazz.newInstance();

spring实现IOC容器的两种接口：
BeanFactory： spring内部使用的接口，一般不建议开发人员使用
**加载配置文件时不创建对象，获取时才创建对象**
ApplicationContext： BeanFactory的子接口，面向开发人员使用的
**加载配置文件时已经创建**

bean管理
创建对象和注入属性
实现方式：
xml文件方式
    创建对象  <bean id="user" class="com.gs.test.User"></bean>
    注入属性  DI(依赖注入，即注入属性)，set注入和有参构造函数
    <bean id="user" class="com.gs.test.User">
        <constructor-arg name="name" value="dream"></constructor-arg>
    </bean>

    <bean id="user2" class="com.gs.test.User">
        <property name="name" value="dream2"></property>
    </bean>

    <bean id="user2" class="com.gs.test.User">
        <property name="user" ref="user"></property>
    </bean>

    注入集合
        数组类型
        list类型
        map类型
    <bean id="user3" class="com.gs.test.User">
        <property name="names">
            <array>
                <value>1</value>
                <value>2</value>
            </array>
        </property>
        <property name="nameList">
            <list>
                <value></value>
            </list>
        </property>
        <property name="nameMap">
            <map>
                <entry key="nn" value="mm"></entry>
            </map>
        </property>
    </bean>

    集合中设置对象类型的值

    把集合注入的部分提取出来

IOC容器操作Bean管理对象（bean作用域）
    在spring里面设置创建bean默认是单实例，但是可以设置成多实例
    怎么设置多实例
        spring里面有个scope属性用于设置单实例或多实例，scope的取值：singlon单实例，prototype多实例(加载配置文件时不再创建对象，在获取对象是创建对象)
    bean的生命周期
        通过构造函数创建bean实例
        为bean设置属性值
        调用bean的初始化方法（配置init-method执行初始化函数）
        bean可以使用了（context.getBean）
        容器关闭的时候，调用bean的销毁方法(需要配置销毁的方法destory-method, 手动调用context.close())
    bean后置处理器
        通过构造函数创建bean实例
        为bean设置属性值
        **把bean的实例传递bean后置处理器的方法BeanPostProcessor.postProcessBeforeInitialization**
        调用bean的初始化方法（配置init-method执行初始化函数）
        **把bean的实例传递bean后置处理器的方法postProcessAfterInitialization**
        bean可以使用了（context.getBean）
        容器关闭的时候，调用bean的销毁方法(需要配置销毁的方法destory-method, 手动调用context.close())

    IOC操作bean管理-xml自动装配
        什么是自动装配
            根据制定装配规则（属性名称或者类型), Spring自动将匹配的属性值进行注入；
            autowire 值byName根据属性名注入，属性名称要和配置文件中的id名称一样； byType根据属性类型进行注入
            <bean id="user2" class="com.gs.test.User" autowire="byName">

    IOC引入外部文件
    jdbc 
        prop.driverClass=com.mysql.jdbc.Driver
        prop.url=jdbc:mysql://localhost:3306/userDb
        prop.userName=root
        prop.password=root

注解方式
@Component
@Service
@Controller
@Repository
上面的四个注解功能一致，都可以用来创建bean实例
基于注解方式实现对象的创建
    * 引入依赖
    * 开启组件扫描，在配置文件引入context名称空间

基于注解的方式实现属性注入
    * @Autowire
    * @Qualifier
    * @Resource
    * @Value

完全注解开发
    @Configuration


AOP
    什么是AOP：面向切面编程，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
    主要意图：将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。

    aop代理：
        有接口通过JDK动态代理 java.lang.reflect.Proxy.newProxyInstance()
    aop术语：
        连接点：可以被增强的方法称为连接点;
        切入点：实际被增强的方法成为切入点;
        通知(增强): 实际增强的部分叫做通知;
            前置通知，后置通知，环绕通知，异常通知，最终通知(finally)
        切面：把通知应用到切入点的过程就是通知

    注解方式
    配置文件方式：Aspect

        <aop:config>
            <aop:pointcut id="p" expression="execution(* com.gs.test.Book.buy(..))">
            <aop:aspect ref="bookProxy">
                <aop:before method="before", pointcut-ref="p">
            </aop:aspect>
        <aop:config>

JDBCTemplete
    定义：


事务
    定义：



Springboot
    Springboot简化Spring开发的框架，整个Spring技术栈的整合；
    Springboot与微服务
