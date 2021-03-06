## IOC

> https://javadoop.com/post/spring-ioc

IoC（Inversion of Control），即控制反转。
之前是在对象内部 new 创建其他对象，然后使用。
而现在 Spring 中有一个容器可以在创建管理这些对象，并且将对象依赖的其他对象注入到这个对象中，这些对象的创建、销毁都由 Spring 进行管理。
相比以前来说，不再由自己控制其他对象的生命周期，这个过程就叫做控制反转。而负责统一管理这些类的容器就叫做 IoC 容器。

使用者之前使用B对象的时候都需要自己去创建和组装，而现在这些创建和组装都交给spring容器去给完成了，使用者只需要去spring容器中查找需要使用的对象就可以了；这个过程中B对象的创建和组装过程被反转了，之前是使用者自己主动去控制的，现在交给spring容器去创建和组装了，对象的构建过程被反转了，所以叫做控制反转；IOC是是面相对象编程中的一种设计原则，主要是为了降低系统代码的耦合度，让系统利于维护和扩展。

DI依赖注入，表示spring容器中创建对象时给其设置依赖对象的方式，通过某些注入方式可以让系统更灵活，比如自动注入等可以让系统变的很灵活，这个后面的文章会细说。

ApplicationContext 接口其实就是 Spring IoC 容器。当然 ApplicationContext 是一个接口，它有很多实现，而它也继承了 BeanFactory。虽然 BeanFactory 是 IoC 容器的最基本的形式，但是 ApplicationContext 对其进行了很多扩展，并具有 BeanFactory 的所有功能，通常建议优先使用 ApplicationContext。


1、到这里已经初始化了 Bean 容器，<bean /> 配置也相应的转换为了一个个 BeanDefinition，然后注册了各个 BeanDefinition 到注册中心，并且发送了注册事件。

说到这里，我们回到 refresh() 方法，我重新贴了一遍代码，看看我们说到哪了。是的，我们才说完 obtainFreshBeanFactory() 方法。

2、准备 Bean 容器: prepareBeanFactory。finishBeanFactoryInitialization(beanFactory); 这个巨头了，这里会负责初始化所有的 singleton beans。

FactoryBean 适用于 Bean 的创建过程比较复杂的场景，比如数据库连接池的创建。

第一步：配置元数据

Spring中配置元数据的方式有以下三种：xml java 注解

### XML启动

第二步：配置解析

完成元数据的配置之后，那么接下来就需要对配置进行解析，显而易见，不同配置方式的语法和对数据的描述是不一样的。
所以Spring封装了不同的配置方式的解析器，比如：
XML：使用ClasspathXmlApplicationContext作为入口，XmlBeanDefinitionReader进行Bean的元数据解析
Java代码、注解：使用AnnotationConfigApplicationContext作为入口，AnnotatedBeanDefinitionReader进行Bean的元数据解析

三：IOC中的几个关键类，分别是：
BeanFactory： 它是最顶层的一个接口，见名知意，典型的工厂模式。
ApplicationContext： 上面BeanFactory工厂帮我们拿到对象，那么工厂是如何产生对象的，那就要看它，它提供了具体的IOC实现，GenericApplicationContext、ClasspathXmlApplicationContext等，同时还提供了以下的附加服务：
BeanDefinition： Bean对象在Spring中的描述定义，可以理解为容器中每个Bean对象都是以BeanDefinition来保存的。
BeanDefinitionReader： 配置Bean的元数据的解析器，比如XmlBeanDefinitionReader、AnnotatedBeanDefinitionReader


我们主要看obtainFreshBeanFactory()方法，Bean的加载注册就是这里面实现的，之后的代码都是容器的初始化信息源和生命周期的一些事件。到这里Bean的资源文件就完成定位了，接下来我们就看加载的具体代码实现。

到这里，基于Xml的IOC容器初始化流程就已经全部完成了，基本看完整个流程之后也梳理了初始化的时序图以及流程图，来加深对整个容器初始化流程的理解，同时我们知道了IOC容器的初始化主要就三步：定位、加载、注册。

### 注解启动

Annotation来完成IOC容器的初始化。前面IOC源码解析中，我们已经知道第一步会调用scan()方法来扫描配置的指定包路径下的所有Bean对象，并封装成BeanDefinition对象存放beanDefinitionMap中，之后会调用refresh()方法去载入Bean的配置资源
```
public static void main(String[] args) {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext("com.test.spring");
        AopA aopA = ac.getBean(AopA.class);
        aopA.print();
}
```

从顶层设计的层面来梳理一下整个IOC的初始化流程：
通过ResourceLoader从类路径、文件系统、URL等方式来定位资源文件位置；
配置Bean数据的文件会被抽象成Resource来处理；
容器通过BeanDefinitionReader来解析Resource，而实际处理过程是委托BeanDefinitionParserDelegate来完成的；
实现BeanDefinitionRegistry接口的子类将BeanDefinition注册到容器中；
容器的本质就是维护一个ConcurrentHashMap来保存BeanDefinition，后续Bean的操作都是围绕这个ConcurrentHashMap来实现的；
使用的时候我们通过BeanFactory和ApplicaitonContext来获得这些Bean对象。

## DI

DI（Dependency Injection）依赖注入：当对象内包含对其他对象的引用时，Spring会帮我们创建或定义这些依赖对象，而无需知道依赖对象的位置甚至具体实现类，实现更有效的解耦。

在IOC的初始化中其实包含了依赖注入的过程，同时通过对getBean()的延伸，进一步得到了第一个问题的答案，依赖注入的时机：
非延迟加载的Bean： IOC初始化中对进行预实例化（也是调用getBean()方法）。
延迟加载的Bean： 第一次调用getBean()方法时。

第二步：注入策略

Spring中注入策略有两种：

有参构造方法注入： 带有参数的构造函数来完成实例化或者带参数的static工厂方法，因为需要对参数进行解析转换，所以过程复杂。
无参构造方法注入： 默认实例化方法，例如使用setter方法注入，通过无参数构造函数或无参数的static工厂方法，过程简单。
第三步：自动装配方式

Spring官网中已经有详细的介绍，关于自动装配的方式有四种：

no：默认没有自动装配，必须由ref配置Bean的引用。
byName：根据属性名称在IOC容器中查找Bean，然后通过setter方法来注入属性（前提是必须有）。
byType：根据属性类型在IOC容器中查找Bean，只存在一个才会注入，多个会引发异常，没有则不注入。
constructor：根据构造函数的参数类型在IOC容器中查找Bean，没有则引发异常。

FactoryBean： 看到这个类，我们很容易想到BeanFactory，其实他两没有关系，IOC中我们已经了解BeanFactory是一个工厂的顶层接口，定义了IOC容器的基本行为，获取bean及bean的各种属性。而FactoryBean是一个工厂Bean接口，目的是用来隐藏复杂Bean的实例化细节，子类通过定制来实现实例化的逻辑，用来产生其他Bean实例。
### 依赖注入流程

如果没有spring，我们的依赖就是自己new，自己写各种各样的构造方法。
注入说明有依赖

spring中依赖注入：手动注入、自动注入
本文我们主要说一下手动注入，手动注入需要我们明确配置需要注入的对象。
将被依赖方注入到依赖方，通常有2种方式：构造函数的方式、set属性的方式，spring中也是通过这两种方式实现注入的，下面详解2种方式。

#### 手动注入
本文主要讲解了xml中bean的依赖注入，都是采用硬编码的方式进行注入的，这种算是手动的方式
注入普通类型通过value属性或者value元素设置注入的值；注入对象如果是容器的其他bean的时候，需要使用ref属性或者ref元素或者内置bean元素的方式
还介绍了其他几种类型List、Set、Map、数组、Properties类型的注入，多看几遍加深理解
后面我们将介绍spring为我们提供的更牛逼的自动注入

#### 自动注入
自动注入是采用约定大约配置的方式来实现的，程序和spring容器之间约定好，遵守某一种都认同的规则，来实现自动注入。

xml中可以在bean元素中通过autowire属性来设置自动注入的方式，自动装配，要是没有autowire的话，那就是写完xml后，写value哈哈，大部分的话，还是再 basePackages="xxx", 使用了 @Autowired 来装配的，用注解来装配，还有用Java代码来装配,话说 @Autowired 也是 5种自动装配的方式之一？只是包装了 byType ，还有 byName这样？
@Autowired，需要注册 AutowiredAnnotationBeanPostProcessor
RequiredAnnotationBeanPostProcessor 是 Spring 中的后置处理用来验证被
@Required 注解的 bean 属性是否被正确的设置了
```xml
<bean id="" class="" autowire="byType|byName|constructor|default" />
```
xml中手动注入存在的不足之处，可以通过自动注入的方式来解决，本文介绍了3中自动注入：通过名称自动注入、通过类型自动注入、通过构造器自动注入
按类型注入中有个比较重要的是注入匹配类型所有的bean，可以将某种类型所有的bean注入给一个List对象，可以将某种类型的所有bean按照`bean名称->bean对象`的映射方式注入给一个Map对象，这种用法比较重要，用途比较大，要掌握
spring中还有其他自动注入的方式，用起来会更爽，后面的文章中我们会详细介绍。

#### @Quanlifier

```java
Caused by:
org.springframework.beans.factory.NoSuchBeanDefinitionException:
No unique bean of type [com.howtodoinjava.common.Person] is defined:
expected single matching bean but found 2: [personA, personB]
```

要解决上面的问题，需要使用 @Quanlifier 注解来告诉 Spring 容器要装配哪个 bean：
```java
public class Customer{
    @Autowired
    @Qualifier("personA")
    private Person person;
}
```

#### spring处理
所以，DI其实是 IOC 的一个部分。

0、我们知道它是一个顶层接口，实现肯定是在它的子类中。顺着准备工作中介绍的DefaultListableBeanFactory类中的preInstantiateSingletons()方法中调用的getBean()方法，点进去我们发现实际调用的是AbstractBeanFactory的getBean()方法。
1、转换真正的beanName
2.从缓存中获取Bean。1、“三级缓存”。2、第一步缓存没有拿到Bean的话，就会再从当前容器的存在的父容器的缓存
3.对依赖的Bean先进行初始化
4.根据不同Scope的Bean进行实例化(重点)
4.1.实例化Bean
4.2.提前缓存Bean
4.3.属性注入
4.4.初始化Bean
4.5.注册Bean
5.获取真正需要的Bean

验证了Bean的实例化入口离不开BeanFactory#getBean()方法，底层通过反射或者CGLIB代理完成对象创建，属性根据名称和类型自动注入等等。

## 循环依赖

从大到小的包含：1、ioc容器初始化。2、DI注入。3、循环依赖。

明白循环依赖及其解决的实现才是更深层次的了解Spring中Bean实例化过程的核心。同时Spring还有其他很多重要功能及实现，像AOP、BeanPostProcessor、Aware等等，如果说Bean是心脏，那它们更像是围绕着的血管一样前后遍布，加强扩展了Spring整体的功能。

有参构造方法注入

所以不管单例还是多例，这种有参构造方法注入（也就是Spring中说的构造函数注入）的场景导致的循环依赖在Spring中常规情况是无法解决的。但是Spring中允许通过一个非常规办法来绕开这个死结，就是@Lazy注解，待会我们再详细讲。

无参构造方法注入的场景，也是就Spring中说的setter注入，先准备测试类，
看来单例的构造器注入是正常的。我们再来看看原型类型，报错

@DependsOn注解
我们先介绍下@DependsOn注解的作用：任何指定此注解的依赖Bean都保证要在被依赖Bean之前由容器创建。

@Lazy的方案和逻辑

这时候返回的BeanA的代理对象就不是null了，所以BeanB也可以完成实例化，接着BeanA也完成实例化了。但是回到main方法内，在打印BeanB时，如果你打开debug，你会发现BeanB中的属性BeanA还是一个代理类，但是接着打印beanB.getBeanA()时发现它就是指向BeanA的引用地址。其实BeanB中的属性BeanA拿到真正BeanA实例化的引用就是在第一次调用getBeanA()的时候，它就会调用之前自定义封装的TargetSource的getTarget()的方法，最后还是通过调用getBean()方法，从缓存中拿到BeanA的实例。

1、可以看出actualTimeBean这个bean是在容器启动过程中被创建好的。
2、代码结合输出可以看出来，LazyInitBean在容器启动过程中并没有创建，当我们调用context.getBean方法的时候，LazyInitBean才被创建的。

## AOP

AOP（Aspect Oriented Programming）：面向切面编程，与面向对象编程OOP的关键单位是类不一样，它的关键单位是切面，它通过提供改变程序结构的方式来补充OOP。通俗点就是说我们可以通过预编译或者运行时动态代理在不修改方法源码的情况下增强方法的功能。

评论：刚刚才恍惚间发现，AOP的功能比 Node.js 里面之前说的 koa.js 的 middleware 的功能更强，middleware 其实更像是 hook 和函数，积木，加功能；而Java里的AOP，AspectJ可以直接字节码增强，动态性更强。

Spring中的代理机制分为两种：

JDK动态代理：内置在JDK中，通过拦截和反射来实现，被代理的对象必须要实现接口。
CGLIB动态代理：一个开源类的定义库，通过ASM字节码生成的方式生成代理类，默认代理没有实现的接口的对象，不能对final修饰的方法进行代理；Spring中可以通过设置@EnableAspectJAutoProxy(proxyTargetClass = true)强制使用用CGLIB进行动态代理。

我们知道 createAopProxy 方法有可能返回 JdkDynamicAopProxy 实例，也有可能返回 ObjenesisCglibAopProxy 实例，这里总结一下：

如果被代理的目标类实现了一个或多个自定义的接口，那么就会使用 JDK 动态代理，如果没有实现任何接口，会使用 CGLIB 实现代理，如果设置了 proxy-target-class="true"，那么都会使用 CGLIB。

JDK 动态代理基于接口，所以只有接口中的方法会被增强，而 CGLIB 基于类继承，需要注意就是如果方法使用了 final 修饰，或者是 private 方法，是不能被增强的。

而完成 AOP的初始化 其实也是穿插在IOC容器的初始化过程中。

BeanPostProcessor的作用是：定义了Bean的初始化回调方法，在其实例化、配置和初始化之后实现一些自定义逻辑。你可以理解为Bean对象的拦截器，如果你想扩展Bean的功能或对其修改包装等，就可以通过实现它去完成。

那我们为什么启动仅仅是通过配置类最终把AnnotationAwareAspectJAutoProxyCreator注册到容器中去呢？其实我们看下它的类图结构就知道了。

启动和调用流程：

1、最终完成自动代理的配置的生效启用。
2、通过配置的切面类来对匹配规则的目标Bean在其初始化之后对其做代理的相应处理。
解析@Aspect切面配置（获取容器中注册的所有beanName；
遍历所有beanName，并找到配置@Aspect注解的切面类；
解析获取切面类中配置的通知方法；
缓存最后获取的通知方法。）
3、生成代理对象，createProxy
4、调用代理方法。完成前后会调用invokeJoinpoint()方法，而invokeJoinpoint()方法的本质就是直接通过反射调用被代理类中的目标方法。

Bean就是普通的java对象，和我们自己new的对象其实是一样的，只是这些对象是由spring去创建和管理的

通过反射调用构造方法创建bean对象

通过静态工厂方法创建bean对象

通过实例工厂方法创建bean对象

通过FactoryBean创建bean对象

Spring容器内部创建bean实例对象常见的有4种方式。

多例bean每次获取的时候都会重新创建，如果这个bean比较复杂，创建时间比较长，会影响系统的性能，这个地方需要注意。
下面要介绍的3个：request、session、application都是在spring web容器环境中才会有的。
PS: spring首先是容器，才会是web容器

spring容器自带的有2种作用域，分别是singleton和prototype；还有3种分别是spring web容器环境中才支持的request、session、application
singleton是spring容器默认的作用域，一个spring容器中同名的bean实例只有一个，多次获取得到的是同一个bean；单例的bean需要考虑线程安全问题
prototype是多例的，每次从容器中获取同名的bean，都会重新创建一个；多例bean使用的时候需要考虑创建bean对性能的影响
一个应用中可以有多个spring容器
自定义scope 3个步骤，实现Scope接口，将实现类注册到spring容器，使用自定义的scope

## depend-on, bean的创建和销毁的顺序，如何来干预bean的创建和销毁的顺序
```java
public void normalBean() {
        System.out.println("容器启动中!");
        String beanXml = "classpath:/com/javacode2018/lesson001/demo7/normalBean.xml";
        ClassPathXmlApplicationContext context = IocUtils.context(beanXml);
        System.out.println("容器启动完毕，准备关闭spring容器!");
        //关闭容器
        context.close();
        System.out.println("spring容器已关闭!");
    }

    public static class Bean3 implements DisposableBean {

        public Bean3() {
                // IocUtils println
            System.out.println(this.getClass() + " constructor!");
        }

        @Override
        public void destroy() throws Exception {
                // close println
            System.out.println(this.getClass() + " destroy()");
        }
    }
```
上面代码中使用到了DisposableBean接口，这个是spring容器提供的一个接口，这个接口中有个destroy方法，我们的bean类可以实现这个接口，当我们调用容器的close方法关闭容器的时候，spring会调用容器中所有bean的destory方法，用来做一些清理的工作，这个以后还会细讲的。

通过定义的顺序确实可以干预bean的创建顺序，通过强依赖也可以干预bean的创建顺序。

#### 因为顺序问题，人工调整顺序是不可能的，所以引出了 depend-on

那么如果xml中定义的bean特别多，而有些bean之间也没有强依赖关系，此时如果想去调整bean的创建和销毁的顺序，得去调整xml中bean的定义顺序，或者去加强依赖，这样是非常不好的，spring中可以通过depend-on来解决这些问题，在不调整bean的定义顺序和强加依赖的情况下，可以通过通过depend-on属性来设置当前bean的依赖于哪些bean，那么可以保证depend-on指定的bean在当前bean之前先创建好，销毁的时候在当前bean之后进行销毁。

### primary
这个详细说出了错误原因：spring容器中定义了2个bean，分别是serviceA和serviceB，这两个bean对象都实现了IService接口，而用例中我们想从容器中获取IService接口对应的bean，此时容器中有2个候选者（serviceA和serviceB）满足我们的需求，此时spring容器不知道如何选择，到底是返回serviceA呢还是返回serviceB呢？spring容器也懵逼了，所以报错了。

当从容器中查找一个bean的时候，如果容器中出现多个Bean候选者时，可以通过primary="true"将当前bean置为首选者，那么查找的时候就会返回主要的候选者，否则将抛出异常。

### autowire-candidate
autowire-candidate：设置当前bean在被其他对象作为自动注入对象的时候，是否作为候选bean，默认值是true。

PS: 一些 xml的配置，不要记得太清楚，要用的时候再查询就好了

## Spring 中 Bean 的⽣命周期、Bean 的实例化和初始化
### Bean生命周期-描述1
1-类加载校验
2-⽅法覆盖校验和准备
3-如果 Bean 配置了实例化的前置处理器，则返回对应的代理对象
4-创建 Bean 的关键⽅法

4、结合代码来看，Bean 的⽣命周期概括起来就是4个阶段：
实例化（Instantiation）
属性填充（Populate），属性填充涉及到依赖注⼊
初始化（Initialization）
销毁（Destruction）

结合代码来看，Bean 的初始化阶段⼲了四件事情：
Aware相关回调
初始化前置处理
初始化
初始化后置处理

### Bean生命周期-描述2
阶段1：Bean元信息配置阶段
阶段2：Bean元信息解析阶段
阶段3：将Bean注册到容器中
阶段4：BeanDefinition合并阶段
阶段5：Bean Class加载阶段
阶段6：Bean实例化阶段（2个小阶段）
Bean实例化前阶段
Bean实例化阶段
阶段7：合并后的BeanDefinition处理
阶段8：属性赋值阶段（3个小阶段）
Bean实例化后阶段
Bean属性赋值前阶段
Bean属性赋值阶段
阶段9：Bean初始化阶段（5个小阶段）
Bean Aware接口回调阶段
Bean初始化前阶段
Bean初始化阶段
Bean初始化后阶段
阶段10：所有单例bean初始化完成后阶段
阶段11：Bean的使用阶段
阶段12：Bean销毁前阶段
阶段13：Bean销毁阶段

注入，是把Bean注入到 context 里面
装配，是Bean之间的关系，有依赖和合作

baseService可以拎出来，用的时候用`parent`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">

    <bean id="serviceA" class="com.javacode2018.lesson001.demo12.ServiceA"/>

    <bean id="baseService" abstract="true">
        <property name="name" value="路人甲Java"/>
        <property name="serviceA" ref="serviceA"/>
    </bean>

    <bean id="serviceB" class="com.javacode2018.lesson001.demo12.ServiceB" parent="baseService"/>

    <bean id="serviceC" class="com.javacode2018.lesson001.demo12.ServiceC" parent="baseService"/>

</beans>
```

此外singleton的生命周期由容器来管理，但是prototype的生命周期得你自己管理。  

他这个文章说法本来就有问题，HttpServeltRequest之所以没问题是因为：注入的这个HttpServletRequest，其实只是一个代理类，每次调用它的方法时，会用RequestContextHolder.currentRequestAttributes().getRequest() 来获取到当前的实际http请求，然后再调用实际request的对应的方法。

lookup-method replaced-method，就是scope寻找的时候用的，有时候用单例，有时候用多例，有时候用 request，单例 == 在整个spring容器内，这个class只有1个实例

### 父子容器
前提，理解一下 mvc & boot 的区别

Spring Boot只是承载者，辅助你简化项目搭建过程的。如果承载的是WEB项目，使用Spring MVC作为MVC框架，那么工作流程和你上面描述的是完全一样的，因为这部分工作是Spring MVC做的而不是Spring Boot。对使用者来说，换用Spring Boot以后，项目初始化方法变了，配置文件变了，另外就是不需要单独安装Tomcat这类容器服务器了，maven打出jar包直接跑起来就是个网站，但你最核心的业务逻辑实现与业务流程实现没有任何变化。所以，用最简练的语言概括就是：Spring 是一个“引擎”； Spring MVC 是基于Spring的一个 MVC 框架 ；Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。

#### 特点
父容器和子容器是相互隔离的，他们内部可以存在名称相同的bean
子容器可以访问父容器中的bean，而父容器不能访问子容器中的bean
调用子容器的getBean方法获取bean的时候，会沿着当前容器开始向上面的容器进行查找，直到找到对应的bean为止
子容器中可以通过任何注入方式注入父容器中的bean，而父容器中是无法注入子容器中的bean，原因是第2点

问题1：springmvc中只使用一个容器是否可以？

只使用一个容器是可以正常运行的。

问题2：那么springmvc中为什么需要用到父子容器？

通常我们使用springmvc的时候，采用3层结构，controller层，service层，dao层；父容器中会包含dao层和service层，而子容器中包含的只有controller层；这2个容器组成了父子容器的关系，controller层通常会注入service层的bean。

采用父子容器可以避免有些人在service层去注入controller层的bean，导致整个依赖层次是比较混乱的。

父容器和子容器的需求也是不一样的，比如父容器中需要有事务的支持，会注入一些支持事务的扩展组件，而子容器中controller完全用不到这些，对这些并不关心，子容器中需要注入一下springmvc相关的bean，而这些bean父容器中同样是不会用到的，也是不关心一些东西，将这些相互不关心的东西隔开，可以有效的避免一些不必要的错误，而父子容器加载的速度也会快一些。

## @Value

我：@Value可以标注在字段上面，可以将外部配置文件中的数据，比如可以将数据库的一些配置信息放在配置文件中，然后通过@Value的方式将其注入到bean的一些字段中
我：嗯，我们项目最常用更多就是通过@Value来引用Properties文件中的配置
可以将配置信息放在db或者其他存储介质中，容器启动的时候，可以将这些信息加载到Environment中，@Value中应用的值最终是通过Environment来解析的，所以只需要扩展一下Environment就可以实现了。
springboot中有个@RefreshScope注解就可以实现你说的这个功能

```java
public enum ScopedProxyMode {
    DEFAULT,
    NO,
    INTERFACES,
    TARGET_CLASS;
}
```
当自定义的Scope中proxyMode=ScopedProxyMode.TARGET_CLASS的时候，会给这个bean创建一个代理对象，调用代理对象的任何方法，都会调用这个自定义的作用域实现类（上面的BeanMyScope）中get方法来重新来获取这个bean对象。
#### 自定义一个Scope：RefreshScope


注意spring的4个阶段：bean定义阶段、BeanFactory后置处理阶段、BeanPostProcessor注册阶段、单例bean创建组装阶段
BeanDefinitionRegistryPostProcessor会在第一个阶段被调用，用来实现bean的注册操作，这个阶段会完成所有bean的注册
BeanFactoryPostProcessor会在第2个阶段被调用，到这个阶段时候，bean此时已经完成了所有bean的注册操作，这个阶段中你可以对BeanFactory中的一些信息进行修改，比如修改阶段1中一些bean的定义信息，修改BeanFactory的一些配置等等
阶段2的时候，2个禁止操作：禁止注册bean、禁止从容器中获取bean
本文介绍的2个接口的实现类可以通过PriorityOrdered接口或者Ordered接口来指定顺序

### @EnableAsync
默认情况下，@EnableAsync使用内置的线程池来异步调用方法，不过我们也可以自定义异步执行任务的线程池。

一个系统中可能有很多业务，比如充值服务、提现服务或者其他服务，这些服务中都有一些方法需要异步执行，默认情况下他们会使用同一个线程池去执行，如果有一个业务量比较大，占用了线程池中的大量线程，此时会导致其他业务的方法无法执行，那么我们可以采用线程隔离的方式，对不同的业务使用不同的线程池，相互隔离，互不影响。
@Async注解有个value参数，用来指定线程池的bean名称，方法运行的时候，就会采用指定的线程池来执行目标方法。
使用步骤
在spring容器中，自定义线程池相关的bean
@Async("线程池bean名称")
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@EnableAsync //@0：启用方法异步调用
@ComponentScan
public class MainConfig5 {

    //@1：值业务线程池bean名称
    public static final String RECHARGE_EXECUTORS_BEAN_NAME = "rechargeExecutors";
    //@2：提现业务线程池bean名称
    public static final String CASHOUT_EXECUTORS_BEAN_NAME = "cashOutExecutors";

    /**
     * @3：充值的线程池，线程名称以recharge-thread-开头
     * @return
     */
    @Bean(RECHARGE_EXECUTORS_BEAN_NAME)
    public Executor rechargeExecutors() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(100);
        //线程名称前缀
        executor.setThreadNamePrefix("recharge-thread-");
        return executor;
    }

    /**
     * @4: 充值的线程池，线程名称以cashOut-thread-开头
     *
     * @return
     */
    @Bean(CASHOUT_EXECUTORS_BEAN_NAME)
    public Executor cashOutExecutors() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(100);
        //线程名称前缀
        executor.setThreadNamePrefix("cashOut-thread-");
        return executor;
    }
}
```

内部使用aop实现的，@EnableAsync会引入一个bean后置处理器：AsyncAnnotationBeanPostProcessor，将其注册到spring容器，这个bean后置处理器在所有bean创建过程中，判断bean的类上是否有@Async注解或者类中是否有@Async标注的方法，如果有，会通过aop给这个bean生成代理对象，会在代理对象中添加一个切面：org.springframework.scheduling.annotation.AsyncAnnotationAdvisor，这个切面中会引入一个拦截器：AnnotationAsyncExecutionInterceptor，方法异步调用的关键代码就是在这个拦截器的invoke方法中实现的.


spring bean的整个生命周期中穿插了各种各样的组件，工具等信息，spring在各个阶段也为我们提供了各式各样的扩展点，其中最核心的分为两大类，PostProcessor类型接口的实现类和Aware接口的实现类
PostProcessor类型接口常用类分为两种，一种是BeanFactoryPostProcessor，另一种是BeanPostProcessor
BeanFactoryPostProcessor与BeanPostProcessor的使用区别
PostProcessor名称	作用
BeanFactoryPostProcessor	关于对象工厂BeanFactory创建完毕的回调处理
BeanPostProcessor	关于通过对象工厂BeanFactory创建对象前后的回调处理

```java
public interface BeanPostProcessor {
    //bean初始化方法调用前被调用
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    //bean初始化方法调用后被调用
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

### Aware

Aware是一个标记性的超接口（顶级接口）,中文翻译感知感觉，就是对于Bean的直接透底

- 实现BeanFactoryAware接口的Bean，可以获取到BeanFactory组件对象。
- 实现EnvironmentAware接口的Bean，可以获取到Environment组件对象。
- BeanNameAware 实现该接口的Bean，可以获取到Bean的名称。