# Effective Java
## 第一章 引言
    本书的大多数规则都源于少数几条基本的原则：
    * 清晰性和简洁性最为重要：模块的用户永远也不应该被模块的行为所迷惑；
    * 模块要尽可能小，担忧不能太小；
    * 代码应该被重用，而不是被拷贝；
    * 模块之间的依赖性应该尽可能地降到最小；
    * 错误应该尽早被检测出来，最好是在编译时刻。

    同大多数学科一样，学习编程艺术首先要学会基本的规则，然后才能知道什么时候可以打破这些规则。

## 第二章 创建和销毁对象
### 第一条 考虑使用静态工厂方法代替构造器
优点:
>1. 静态工厂方法的第一大优势在于，他们有名称。
>>当一个类需要多个带有相同签名的构造器时，就用静态工厂方法替代，并且慎重地选择名称以便突出他们的区别。

>2. 静态工厂方法的第二大优势在于，不必重复地创建新对象
>>对于不可变类，可以预先创建实例，或者将构建好的实例缓存起来，进行重复利用，避免不必要的重复对象。  
如果程序经常请求创建相同的对象，并且创建对象的代价很高，这可以极大地提升性能。  
有助于类总能控制在某个时刻哪些实例应该存在。这种类被称作 *实例受控的类*(instance-controlled)。  
实例受控类可以确保它是一个Singleton或者是不可实例化的。使得不可变类的客户端可以使用`==`操作符代替equals方法来提升性能。例如，枚举(enum)类型。  

>3. 静态工厂方法的第三大优势在于，他们可以返回原返回类型的任何子类型的对象
>>* 例如EnumSet，参数不同返回不同的子类型对象。元素小于或等于64时，静态工厂返回一个RegalarEnumSet实例，用单个
long进行支持，如果元素有65个或者更多，工厂就返回JumboEnumSet实例，用long数组进行支持。
>>* 静态工厂方法返回的对象所属类，在编写该静态工厂时可以不存在。这中灵活的静态工厂方法构成了*服务提供者框架*（Service Provider Framework）的基础。例如JDBC API。该类框架的三个重要组件：服务接口（Service Interface），供提供者实现；提供者注册API（Provider Registration API），系统用来注册实现，让客户端可以访问它们；服务访问API（Service Access API），是客户端用来获取服务的实例的。第四个组件是可选的：服务提供者接口（Service Provider Interface），这些提供者负责创建其服务实现的实例。如果没有服务提供者接口，实现就按照类名注册，并通过反射方式进行实例化。对与JDBC，Connection就是它的服务接口，DriverManager.registerDriver是提供者注册API，DriverManager.getConnection服务访问API，Driver就是服务提供者接口。
>>* 服务提供者框架还有无数种变体。例如，服务访问API利用适配器模式，返回比提供者需要的更丰富的服务接口。

>4. 第四大优势，在创建参数化类型实例时，使代码更简洁。  
>>类型推导在jdk1.7中已经使用了。

缺点：
>1. 静态工厂方法的主要缺点在于，类如果不含有共有的和受保护的构造器，就不能被子类化。
>>但是这样也许会因祸得福，因为它鼓了程序员使用复合，而不是继承。

>2. 静态工厂方法的第二个缺点在于，它们与其他的静态方法实际上没有任何区别。
>>在API文档中没有像构造器一样被明显的标识出来。使得查明如何实例化这样一个类非常困难。
>>可以通过在类或者接口注释中关注静态工厂，同时准守标准的命名习惯。下面是静态工厂方法的一些惯用名称：
>>>* valueOf——不严格地讲，该方法返回的实例与它的参数具有相同的值。这样的静态工厂方法实际上是类型转换方法
>>>* of——valueOf的一种更为简洁的替代，在EnumSet中使用并流行起来
>>>* getInstance——返回的实例是通过方法的参数来描述的，但是不能够说与参数具有同样的值。对于Singleton来说，该方法没有参数，并返回唯一的实例。
>>>* newInstance——像getInstance一样，但newInstance能够确保返回的每个实例都与所有其他实例不同
>>>* get *Type*——像getInstance一样，但是在工厂方法处于不同的类中的时候使用。*Type*表示工厂方法所返回的对象类型。
>>>* new *Type*——像newInstance一样，但是在工厂方法处于不同的类中的时候使用。*Type*表示工厂方法所返回的对象类型。