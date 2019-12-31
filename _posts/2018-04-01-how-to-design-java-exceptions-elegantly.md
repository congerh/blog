---
title: "如何优雅地设计Java异常"
categories:
  - Java
tags:
  - Java
  - Exception
copyright_url: https://lrwinx.github.io/2016/04/28/如何优雅的设计java异常/
author: Lrwin
---

异常处理是程序开发中必不可少操作之一，但如何正确优雅的对异常进行处理确是一门学问，笔者根据自己的开发经验来谈一谈我是如何对异常进行处理的。

由于本文只作一些经验之谈，不涉及到基础知识部分，如果读者对异常的概念还很模糊，请先查看基础知识。

## 如何选择异常类型

### 异常的类别

正如我们所知道的，Java中的异常的超类是java.lang.Throwable(后文省略为Throwable),它有两个比较重要的子类,java.lang.Exception(后文省略为Exception)和java.lang.Error(后文省略为Error)，其中Error由JVM虚拟机进行管理,如我们所熟知的OutOfMemoryError异常等，所以我们本文不关注Error异常，那么我们细说一下Exception异常。

Exception异常有个比较重要的子类，叫做RuntimeException。我们将RuntimeException或其他继承自RuntimeException的子类称为非受检异常(unchecked Exception)，其他继承自Exception异常的子类称为受检异常(checked Exception)。本文重点来关注一下受检异常和非受检异常这两种异常。

### 如何选择异常

从笔者的开发经验来看，如果在一个应用中，需要开发一个方法(如某个功能的service方法)，这个方法如果中间可能出现异常，那么你需要考虑这个异常出现之后是否调用者可以处理，并且你是否希望调用者进行处理，如果调用者可以处理，并且你也希望调用者进行处理，那么就要抛出受检异常，提醒调用者在使用你的方法时，考虑到如果抛出异常时如果进行处理，相似的，如果在写某个方法时，你认为这是个偶然异常，理论上说，你觉得运行时可能会碰到什么问题，而这些问题也许不是必然发生的，也不需要调用者显示的通过异常来判断业务流程操作的，那么这时就可以使用一个RuntimeException这样的非受检异常。

好了，估计我上边说的这段话，你读了很多遍也依然觉得晦涩了。

那么，请跟着我的思路，在慢慢领会一下。

### 什么时候才需要抛异常

首先我们需要了解一个问题，什么时候才需要抛异常？异常的设计是方便给开发者使用的，但不是乱用的，笔者对于什么时候抛异常这个问题也问了很多朋友，能给出准确答案的确实不多。其实这个问题很简单，如果你觉得某些”问题”解决不了了，那么你就可以抛出异常了。比如，你在写一个service,其中在写到某段代码处,你发现可能会产生问题，那么就请抛出异常吧，相信我，你此时抛出异常将是一个最佳时机。

### 应该抛出怎样的异常

了解完了什么时候才需要抛出异常后，我们再思考一个问题，真的当我们抛出异常时，我们应该选用怎样的异常呢？究竟是受检异常还是非受检异常呢(RuntimeException)呢？我来举例说明一下这个问题，先从受检异常说起,比如说有这样一个业务逻辑，需要从某文件中读取某个数据，这个读取操作可能是由于文件被删除等其他问题导致无法获取从而出现读取错误，那么就要从redis或mysql数据库中再去获取此数据,参考如下代码，getKey(Integer)为入口程序。

```java
public String getKey(Integer key){
    String  value;
    try {
        InputStream inputStream = getFiles("/file/nofile");
        //接下来从流中读取key的value指
        value = ...;
    } catch (Exception e) {
        //如果抛出异常将从mysql或者redis进行取之
        value = ...;
    }
}

public InputStream getFiles(String path) throws Exception {
    File file = new File(path);
    InputStream inputStream = null;
    try {
        inputStream = new BufferedInputStream(new FileInputStream(file));
    } catch (FileNotFoundException e) {
        throw new Exception("I/O读取错误",e.getCause());
    }
    return inputStream;
}
```

ok，看了以上代码以后，你也许心中有一些想法，原来受检异常可以控制义务逻辑，对，没错，通过受检异常真的可以控制业务逻辑，但是切记不要这样使用，我们应该合理的抛出异常，因为程序本身才是流程，异常的作用仅仅是当你进行不下去的时候找到的一个借口而已，它并不能当成控制程序流程的入口或出口，如果这样使用的话，是在将异常的作用扩大化，这样将会导致代码复杂程度的增加，耦合性会提高，代码可读性降低等问题。那么就一定不要使用这样的异常吗？其实也不是，在真的有这样的需求的时候，我们可以这样使用，只是切记，不要把它真的当成控制流程的工具或手段。那么究竟什么时候才要抛出这样的异常呢？要考虑，如果调用者调用出错后，一定要让调用者对此错误进行处理才可以，满足这样的要求时，我们才会考虑使用受检异常。

接下来，我们来看一下非受检异常呢(RuntimeException)，对于RuntimeException这种异常，我们其实很多见，比如java.lang.NullPointerException／java.lang.IllegalArgumentException等，那么这种异常我们时候抛出呢？当我们在写某个方法的时候，可能会偶然遇到某个错误，我们认为这个问题时运行时可能为发生的，并且理论上讲，没有这个问题的话，程序将会正常执行的时候，它不强制要求调用者一定要捕获这个异常，此时抛出RuntimeException异常,举个例子，当传来一个路径的时候，需要返回一个路径对应的File对象:

```java
public void test() {
    myTest.getFiles("");
}

public File getFiles(String path) {
    if(null == path || "".equals(path)){
        throw  new NullPointerException("路径不能为空!");
    }
    File file = new File(path);

    return file;
}
```

上述例子表明，如果调用者调用getFiles(String)的时候如果path是空，那么就抛出空指针异常(它是RuntimeException的子类),调用者不用显示的进行try…catch…操作进行强制处理。这就要求调用者在调用这样的方法时先进行验证，避免发生RuntimeException。如下:

```java
public void test() {
    String path = "/a/b.png";
    if(null != path && !"".equals(path)){
        myTest.getFiles("");
    }
}

public File getFiles(String path) {
    if(null == path || "".equals(path)){
        throw  new NullPointerException("路径不能为空!");
    }
    File file = new File(path);

    return file;
}
```

### 应该选用哪种异常

通过以上的描述和举例，可以总结出一个结论，RuntimeException异常和受检异常之间的区别就是:是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检异常，否则就选择非受检异常(RuntimeException)。一般来讲，如果没有特殊的要求，我们建议使用RuntimeException异常。

## 场景介绍和技术选型

### 架构描述

正如我们所知，传统的项目都是以MVC框架为基础进行开发的，本文主要从使用restful风格接口的设计来体验一下异常处理的优雅。

我们把关注点放在restful的api层(和web中的controller层类似)和service层，研究一下在service中如何抛出异常，然后api层如何进行捕获并且转化异常。

使用的技术是:spring-boot,jpa(hibernate),mysql,如果对这些技术不是太熟悉，读者需要自行阅读相关材料。

### 业务场景描述

选择一个比较简单的业务场景，以电商中的收货地址管理为例，用户在移动端进行购买商品时，需要进行收货地址管理，在项目中，提供一些给移动端进行访问的api接口，如:添加收货地址，删除收货地址，更改收货地址，默认收货地址设置，收货地址列表查询，单个收货地址查询等接口。

### 构建约束条件

ok，这个是设置好的一个很基本的业务场景，当然，无论什么样的api操作，其中都包含一些规则:

* 添加收货地址:
  * 入参:
    * 用户id
    * 收货地址实体信息
  * 约束:
    * 用户id不能为空，且此用户确实是存在 的
    * 收货地址的必要字段不能为 空
    * 如果用户还没有收货地址，当此收货地址创建时设置成默认收货地址

  ---

* 删除收货地址:
  * 入参:
    * 用户id
    * 收货地址id
  * 约束:
    * 用户id不能为空，且此用户确实是存在的
    * 收货地址不能为空，且此收货地址确实是存在的
    * 判断此收货地址是否是用户的收货地址
    * 判断此收货地址是否为默认收货地址，如果是默认收货地址，那么不能进行删除

  ---

* 更改收货地址:
  * 入参:
    * 用户id
    * 收货地址id
  * 约束:
    * 用户id不能为空，且此用户确实是存在的
    * 收货地址不能为空，且此收货地址确实是存在的
    * 判断此收货地址是否是用户的收货地址

  ---

* 默认地址设置:
  * 入参:
    * 用户id
    * 收货地址id
  * 约束:
    * 用户id不能为空，且此用户确实是存在的
    * 收货地址不能为空，且此收货地址确实是存在的
    * 判断此收货地址是否是用户的收货地址

  ---

* 收货地址列表查询:
  * 入参:
    * 用户id
  * 约束:
    * 用户id不能为空，且此用户确实是存在的

  ---

* 单个收货地址查询:
  * 入参:
    * 用户id
    * 收货地址id
  * 约束:
    * 用户id不能为空，且此用户确实是存在的
    * 收货地址不能为空，且此收货地址确实是存在的
    * 判断此收货地址是否是用户的收货地址

  ---

### 约束判断和技术选型

对于上述列出的约束条件和功能列表，我选择几个比较典型的异常处理场景进行分析:添加收货地址，删除收货地址，获取收货地址列表。

那么应该有哪些必要的知识储备呢，让我们看一下收货地址这个功能:

添加收货地址中需要对用户id和收货地址实体信息就行校验，那么对于非空的判断，我们如何进行工具的选择呢？传统的判断如下:

```java
/**
 * 添加地址
 * @param uid
 * @param address
 * @return
 */
public Address addAddress(Integer uid,Address address){
    if(null != uid){
        //进行处理..
    }
    return null;
}
```

上边的例子，如果只判断uid为空还好，如果再去判断address这个实体中的某些必要属性是否为空，在字段很多的情况下，这无非是灾难性的。

那我们应该怎么进行这些入参的判断呢，给大家介绍两个知识点:

1. Guava中的Preconditions类实现了很多入参方法的判断
2. jsr 303的validation规范(目前实现比较全的是hibernate实现的hibernate-validator)

如果使用了这两种推荐技术，那么入参的判断会变得简单很多。推荐大家多使用这些成熟的技术和jar工具包，他可以减少很多不必要的工作量。我们只需要把重心放到业务逻辑上。而不会因为这些入参的判断耽误更多的时间。

## 如何优雅地设计Java异常

### domain介绍

根据项目场景来看，需要两个domain模型，一个是用户实体，一个是地址实体。

Address domain如下:

```java
@Entity
@Data
public class Address {
    @Id
    @GeneratedValue
    private Integer id;
    private String province;//省
    private String city;//市
    private String county;//区
    private Boolean isDefault;//是否是默认地址

    @ManyToOne(cascade={CascadeType.ALL})
    @JoinColumn(name="uid")
    private User user;
}
```

User domain如下:

```java
@Entity
@Data
public class User {
    @Id
    @GeneratedValue
    private Integer id;
    private String name;//姓名

    @OneToMany(cascade= CascadeType.ALL,mappedBy="user",fetch = FetchType.LAZY)
    private Set<Address> addresses;
}
```

ok,上边是一个模型关系，用户-收货地址的关系是1-n的关系。上边的@Data是使用了一个叫做lombok的工具，它自动生成了Setter和Getter等方法，用起来非常方便，感兴趣的读者可以自行了解一下。

### dao介绍

数据连接层，我们使用了spring-data-jpa这个框架，它要求我们只需要继承框架提供的接口，并且按照约定对方法进行取名，就可以完成我们想要的数据库操作。

用户数据库操作如下:

```java
@Repository
public interface IUserDao extends JpaRepository<User,Integer> {

}
```

收货地址操作如下:

```java
@Repository
public interface IAddressDao extends JpaRepository<Address,Integer> {

}
```

正如读者所看到的，我们的DAO只需要继承JpaRepository,它就已经帮我们完成了基本的CURD等操作，如果想了解更多关于spring-data的这个项目，请参考一下spring的官方文档，它比不方案我们对异常的研究。

### Service异常设计

ok，终于到了我们的重点了，我们要完成service一些的部分操作:添加收货地址，删除收货地址，获取收货地址列表。

首先看我的service接口定义:

```java
public interface IAddressService {

    /**
    * 创建收货地址
    * @param uid
    * @param address
    * @return
    */
    Address createAddress(Integer uid,Address address);

    /**
    * 删除收货地址
    * @param uid
    * @param aid
    */
    void deleteAddress(Integer uid,Integer aid);

    /**
    * 查询用户的所有收货地址
    * @param uid
    * @return
    */
    List<Address> listAddresses(Integer uid);
}
```

我们来关注一下实现:

#### 添加收货地址

首先再来看一下之前整理的约束条件:

* 入参:
  * 用户id
  * 收货地址实体信息
* 约束:
  * 用户id不能为空，且此用户确实是存在的
  * 收货地址的必要字段不能为空
  * 如果用户还没有收货地址，当此收货地址创建时设置成默认收货地址

先看以下代码实现:

```java
@Override
public Address createAddress(Integer uid, Address address) {
    //============ 以下为约束条件   ==============
    //1.用户id不能为空，且此用户确实是存在的
    Preconditions.checkNotNull(uid);
    User user = userDao.findOne(uid);
    if(null == user){
        throw new RuntimeException("找不到当前用户!");
    }
    //2.收货地址的必要字段不能为空
    BeanValidators.validateWithException(validator, address);
    //3.如果用户还没有收货地址，当此收货地址创建时设置成默认收货地址
    if(ObjectUtils.isEmpty(user.getAddresses())){
        address.setIsDefault(true);
    }

    //============ 以下为正常执行的业务逻辑   ==============
    address.setUser(user);
    Address result = addressDao.save(address);
    return result;
}
```

其中，已经完成了上述所描述的三点约束条件，当三点约束条件都满足时，才可以进行正常的业务逻辑，否则将抛出异常(一般在此处建议抛出运行时异常-RuntimeException)。

介绍以下以上我所用到的技术:

1. Preconfitions.checkNotNull(T t)这个是使用Guava中的com.google.common.base.Preconditions进行判断的，因为service中用到的验证较多，所以建议将Preconfitions改成静态导入的方式:
        import static com.google.common.base.Preconditions.checkNotNull;

2. BeanValidators.validateWithException(validator, address);

    这个使用了hibernate实现的jsr 303规范来做的，需要传入一个validator和一个需要验证的实体,那么validator是如何获取的呢,如下:

```java
@Configuration
public class BeanConfigs {

@Bean
public javax.validation.Validator getValidator(){
    return new LocalValidatorFactoryBean();
    }
}
```

他将获取一个Validator对象，然后我们在service中进行注入便可以使用了:

```java
@Autowired     
private Validator validator ;
```

那么BeanValidators这个类是如何实现的？其实实现方式很简单，只要去判断jsr 303的标注注解就ok了。

那么jsr 303的注解写在哪里了呢？当然是写在address实体类中了:

```java
@Entity
@Setter
@Getter
public class Address {
    @Id
    @GeneratedValue
    private Integer id;
    @NotNull
    private String province;//省
    @NotNull
    private String city;//市
    @NotNull
    private String county;//区
    private Boolean isDefault = false;//是否是默认地址

    @ManyToOne(cascade={CascadeType.ALL})
    @JoinColumn(name="uid")
    private User user;
}
```

写好你需要的约束条件来进行判断，如果合理的话，才可以进行业务操作，从而对数据库进行操作。

这块的验证是必须的，一个最主要的原因是:这样的验证可以避免脏数据的插入。如果读者有正式上线的经验的话，就可以理解这样的一个事情，任何的代码错误都可以容忍和修改，但是如果出现了脏数据问题，那么它有可能是一个毁灭性的灾难。程序的问题可以修改，但是脏数据的出现有可能无法恢复。所以这就是为什么在service中一定要判断好约束条件，再进行业务逻辑操作的原因了。

此处的判断为业务逻辑判断，是从业务角度来进行筛选判断的，除此之外，有可能在很多场景中都会有不同的业务条件约束，只需要按照要求来做就好。

**对于约束条件的总结如下:**

* 基本判断约束(null值等基本判断)
* 实体属性约束(满足jsr 303等基础判断)
* 业务条件约束(需求提出的不同的业务约束)

**当这个三点都满足时，才可以进行下一步操作**

ok,基本介绍了如何做一个基础的判断，那么再回到异常的设计问题上，上述代码已经很清楚的描述如何在适当的位置合理的判断一个异常了，那么如何合理的抛出异常呢？

只抛出RuntimeException就算是优雅的抛出异常吗？当然不是，对于service中的抛出异常，笔者认为大致有两种抛出的方法:

1. 抛出带状态码RumtimeException异常
2. 抛出指定类型的RuntimeException异常

相对这两种异常的方式进行结束，第一种异常指的是我所有的异常都抛RuntimeException异常，但是需要带一个状态码，调用者可以根据状态码再去查询究竟service抛出了一个什么样的异常。

第二种异常是指在service中抛出什么样的异常就自定义一个指定的异常错误，然后在进行抛出异常。

一般来讲，如果系统没有别的特殊需求的时候，在开发设计中，建议使用第二种方式。但是比如说像基础判断的异常，就可以完全使用guava给我们提供的类库进行操作。jsr 303异常也可以使用自己封装好的异常判断类进行操作，因为这两种异常都是属于基础判断，不需要为它们指定特殊的异常。但是对于第三点义务条件约束判断抛出的异常，就需要抛出指定类型的异常了。
对于

```java
throw new RuntimeException("找不到当前用户!");
```

定义一个特定的异常类来进行这个义务异常的判断:

```java
public class NotFindUserException extends RuntimeException {
    public NotFindUserException() {
        super("找不到此用户");
    }

    public NotFindUserException(String message) {
        super(message);
    }
}
```

然后将此处改为:

```java
throw new NotFindUserException("找不到当前用户!");
```

or

```java
throw new NotFindUserException();
```

ok,通过以上对service层的修改，代码更改如下:

```java
@Override
public Address createAddress(Integer uid, Address address) {
    //============ 以下为约束条件   ==============
    //1.用户id不能为空，且此用户确实是存在的
    checkNotNull(uid);
    User user = userDao.findOne(uid);
    if(null == user){
        throw new NotFindUserException("找不到当前用户!");
    }
    //2.收货地址的必要字段不能为空
    BeanValidators.validateWithException(validator, address);
    //3.如果用户还没有收货地址，当此收货地址创建时设置成默认收货地址
    if(ObjectUtils.isEmpty(user.getAddresses())){
        address.setIsDefault(true);
    }

    //============ 以下为正常执行的业务逻辑   ==============
    address.setUser(user);
    Address result = addressDao.save(address);
    return result;
}
```

这样的service就看起来稳定性和理解性就比较强了。

#### 删除收货地址

* 入参:
  * 用户id
  * 收货地址id
* 约束:
  * 用户id不能为空，且此用户确实是存在的
  * 收货地址不能为空，且此收货地址确实是存在的
  * 判断此收货地址是否是用户的收货地址
  * 判断此收货地址是否为默认收货地址，如果是默认收货地址，那么不能进行删除

它与上述添加收货地址类似，故不再赘述，delete的service设计如下:

```java
@Override
public void deleteAddress(Integer uid, Integer aid) {
    //============ 以下为约束条件   ==============
    //1.用户id不能为空，且此用户确实是存在的
    checkNotNull(uid);
    User user = userDao.findOne(uid);
    if(null == user){
        throw new NotFindUserException();
    }
    //2.收货地址不能为空，且此收货地址确实是存在的
    checkNotNull(aid);
    Address address = addressDao.findOne(aid);
    if(null == address){
        throw new NotFindAddressException();
    }
    //3.判断此收货地址是否是用户的收货地址
    if(!address.getUser().equals(user)){
        throw new NotMatchUserAddressException();
    }
    //4.判断此收货地址是否为默认收货地址，如果是默认收货地址，那么不能进行删除
    if(address.getIsDefault()){
       throw  new DefaultAddressNotDeleteException();
    }

    //============ 以下为正常执行的业务逻辑   ==============
    addressDao.delete(address);
}
```

设计了相关的四个异常类:  
NotFindUserException,  
NotFindAddressException,  
NotMatchUserAddressException,  
DefaultAddressNotDeleteException.  
根据不同的业务需求抛出不同的异常。

#### 获取收货地址列表:

* 入参:
  * 用户id
* 约束:
  * 用户id不能为空，且此用户确实是存在的

代码如下:

```java
@Override
public List<Address> listAddresses(Integer uid) {
    //============ 以下为约束条件   ==============
    //1.用户id不能为空，且此用户确实是存在的
    checkNotNull(uid);
    User user = userDao.findOne(uid);
    if(null == user){
        throw new NotFindUserException();
    }

    //============ 以下为正常执行的业务逻辑   ==============
    User result = userDao.findOne(uid);
    return result.getAddresses();
}
```

#### api异常设计

大致有两种抛出的方法:

1. 抛出带状态码RumtimeException异常
2. 抛出指定类型的RuntimeException异常

这个是在设计service层异常时提到的，通过对service层的介绍，我们在service层抛出异常时选择了第二种抛出的方式，不同的是，在api层抛出异常我们需要使用这两种方式进行抛出:要指定api异常的类型，并且要指定相关的状态码，然后才将异常抛出，这种异常设计的核心是让调用api的使用者更能清楚的了解发生异常的详细信息，除了抛出异常外，我们还需要将状态码对应的异常详细信息以及异常有可能发生的问题制作成一个对应的表展示给用户，方便用户的查询。（如github提供的api文档，微信提供的api文档等）,还有一个好处:如果用户需要自定义提示消息，可以根据返回的状态码进行提示的修改。

##### api验证约束

首先对于api的设计来说，需要存在一个dto对象，这个对象负责和调用者进行数据的沟通和传递，然后dto->domain在传给service进行操作，这一点一定要注意，第二点，除了说道的service需要进行基础判断(null判断)和jsr 303验证以外，同样的，api层也需要进行相关的验证，如果验证不通过的话，直接返回给调用者，告知调用失败，不应该带着不合法的数据再进行对service的访问，那么读者可能会有些迷惑，不是service已经进行验证了，为什么api层还需要进行验证么？这里便设计到了一个概念:编程中的墨菲定律，如果api层的数据验证疏忽了，那么有可能不合法数据就带到了service层，进而讲脏数据保存到了数据库。

**所以缜密编程的核心是:永远不要相信收到的数据是合法的。**

##### api异常设计

设计api层异常时，正如我们上边所说的，需要提供错误码和错误信息，那么可以这样设计，提供一个通用的api超类异常，其他不同的api异常都继承自这个超类:

```java
public class ApiException extends RuntimeException {
    protected Long errorCode ;
    protected Object data ;

    public ApiException(Long errorCode,String message,Object data,Throwable e){
        super(message,e);
        this.errorCode = errorCode ;
        this.data = data ;
    }

    public ApiException(Long errorCode,String message,Object data){
        this(errorCode,message,data,null);
    }

    public ApiException(Long errorCode,String message){
        this(errorCode,message,null,null);
    }

    public ApiException(String message,Throwable e){
        this(null,message,null,e);
    }

    public ApiException(){

    }

    public ApiException(Throwable e){
        super(e);
    }

    public Long getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(Long errorCode) {
        this.errorCode = errorCode;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}
```

然后分别定义api层异常：  
ApiDefaultAddressNotDeleteException,  
ApiNotFindAddressException,  
ApiNotFindUserException,  
ApiNotMatchUserAddressException。  
以默认地址不能删除为例:

```java
public class ApiDefaultAddressNotDeleteException extends ApiException {

    public ApiDefaultAddressNotDeleteException(String message) {
        super(AddressErrorCode.DefaultAddressNotDeleteErrorCode, message, null);
    }
}
```

AddressErrorCode.DefaultAddressNotDeleteErrorCode就是需要提供给调用者的错误码。错误码类如下:

```java
public abstract class AddressErrorCode {
    public static final Long DefaultAddressNotDeleteErrorCode = 10001L;//默认地址不能删除
    public static final Long NotFindAddressErrorCode = 10002L;//找不到此收货地址
    public static final Long NotFindUserErrorCode = 10003L;//找不到此用户
    public static final Long NotMatchUserAddressErrorCode = 10004L;//用户与收货地址不匹配
}
```

ok,那么api层的异常就已经设计完了，在此多说一句，AddressErrorCode错误码类存放了可能出现的错误码，更合理的做法是把他放到配置文件中进行管理。

##### api处理异常

api层会调用service层，然后来处理service中出现的所有异常，首先，需要保证一点，一定要让api层非常轻，基本上做成一个转发的功能就好(接口参数，传递给service参数，返回给调用者数据,这三个基本功能)，然后就要在传递给service参数的那个方法调用上进行异常处理。

此处仅以添加地址为例:

```java
@Autowired
private IAddressService addressService;


/**
 * 添加收货地址
 * @param addressDTO
 * @return
 */
@RequestMapping(method = RequestMethod.POST)
public AddressDTO add(@Valid @RequestBody AddressDTO addressDTO){
    Address address = new Address();
    BeanUtils.copyProperties(addressDTO,address);
    Address result;
    try {
        result = addressService.createAddress(addressDTO.getUid(), address);
    }catch (NotFindUserException e){
        throw new ApiNotFindUserException("找不到该用户");
    }catch (Exception e){//未知错误
        throw new ApiException(e);
    }
    AddressDTO resultDTO = new AddressDTO();
    BeanUtils.copyProperties(result,resultDTO);
    resultDTO.setUid(result.getUser().getId());

    return resultDTO;
}
```

这里的处理方案是调用service时，判断异常的类型，然后将任何service异常都转化成api异常，然后抛出api异常，这是常用的一种异常转化方式。相似删除收货地址和获取收货地址也类似这样处理，在此，不在赘述。

##### api异常转化

已经讲解了如何抛出异常和何如将service异常转化为api异常，那么转化成api异常直接抛出是否就完成了异常处理呢？ 答案是否定的，当抛出api异常后，我们需要把api异常返回的数据(json or xml)让用户看懂，那么需要把api异常转化成dto对象(ErrorDTO),看如下代码:

```java
@ControllerAdvice(annotations = RestController.class)
class ApiExceptionHandlerAdvice {

    /**
    * Handle exceptions thrown by handlers.
    */
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public ResponseEntity<ErrorDTO> exception(Exception exception,HttpServletResponse response) {
        ErrorDTO errorDTO = new ErrorDTO();
        if(exception instanceof ApiException){//api异常
            ApiException apiException = (ApiException)exception;
            errorDTO.setErrorCode(apiException.getErrorCode());
        }else{//未知异常
            errorDTO.setErrorCode(0L);
        }
        errorDTO.setTip(exception.getMessage());
        ResponseEntity<ErrorDTO> responseEntity = new ResponseEntity<>(errorDTO,HttpStatus.valueOf(response.getStatus()));
        return responseEntity;
    }

    @Setter
    @Getter
    class ErrorDTO{
        private Long errorCode;
        private String tip;
    }
}
```

ok,这样就完成了api异常转化成用户可以读懂的DTO对象了，代码中用到了@ControllerAdvice，这是spring MVC提供的一个特殊的切面处理。

当调用api接口发生异常时，用户也可以收到正常的数据格式了,比如当没有用户(uid为2)时，却为这个用户添加收货地址,postman(Google plugin 用于模拟http请求)之后的数据:

```java
{
  "errorCode": 10003,
  "tip": "找不到该用户"
}
```

## 总结

本文只从如何设计异常作为重点来讲解，涉及到的api传输和service的处理，还有待优化，比如api接口访问需要使用https进行加密，api接口需要OAuth2.0授权或api接口需要签名认证等问题，文中都未曾提到，本文的重心在于异常如何处理，所以读者只需关注涉及到异常相关的问题和处理方式就可以了。希望本篇文章对你理解异常有所帮助。

## 参考

* [如何优雅的设计java异常](https://lrwinx.github.io/2016/04/28/%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E8%AE%BE%E8%AE%A1java%E5%BC%82%E5%B8%B8/)
