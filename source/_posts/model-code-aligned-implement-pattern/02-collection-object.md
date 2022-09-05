
---
title: 模型代码一致的实现模式（二）—— 集合对象（上）
date: 2022-09-05 17:17
---

## 集合值得单独抽取对象

大部分的现代语言，都提供了集合的类型，list、vector、set、tree、map等，并围绕这些集合的类型，有一整套方便的api工具库，这导致大家不爱对集合进行抽象。

比如一个用户（ User ）有多个订单（ Order ）之类。那么实现的时候我们会怎么样呢？往往是一个User有一个叫orders的属性，类型是个List，当然不同语言有不同的列表类型，比如C++可能用的就是vector。代码类似：

```java
class User {
    private List<Order> orders;
}
```

其实仔细想想，我们给Order要定个Order类，不会用Map或者Json这种类型，为啥集合就用List而不是单独抽象一个Orders呢？比如类似这样：

```java
class User {
    private Orders order;
}
//当然，User也可以有自己的集合对象Users。
```

实际上，我们在领域概念当中，某个类型的集合其实是一种隐藏概念，其实经常会出现在业务人员或者领域专家的口中，很多需求都是围绕着这种集合概念展开的，我们需要把这种隐藏的概念显示的表达出来以达成模型与代码的一致。否则以List来承载集合概念这样的实现模式，对于简单系统还是能胜任的，但是对于复杂系统，通常是不能胜任的。这么做具体有什么好处呢？我们后面展开聊聊。

注：为了表达方便，我下面将像Order这种表达单个个体的类称之为单体类；将Orders这种表达个体集合的类，称之为集合类。

## 集合对象的好处

## 承载集合逻辑

首先它可以承载围绕着集合进行的扩展需求，不至于把这部分需求散到系统的各个地方。举个例子，假设我们有一个需求场景，里面有两种指令，一种是上层指令，我们就叫它Command，一种是下层指令，我们就叫它Action，一个上层指令需要被转换为多个下层指令来执行，也就是一个Command需要根据条件转换为多个Action，这个转换逻辑非常复杂，受各种自身状态和外部环境影响，所以有很多的分支。

那么在这种场景下，我们转换出的Action，到底是一个的List，比如如下代码：

```java
class Action {
    public void exec(ExecContext context){
        //...
    }
}
//标准库承载集合
class Command_1 {
    public List<Action> transform(TransformContext context){
         //...
    }
}
```

还是抽象成一个集合对象Actions呢? 比如如下代码：

```java
//集合对象承载集合
class Command_2 {
    public Actions transform(TransformContext context){
         //...
    }
}

class Actions {
    private List<Action> actions;

}
```


那么我们可以分析一下，如果集合对象Actions不被建模，当出现了对整组产生影响的新需求，比如对整组的action进行统筹优化，或者一组action执行完要发个通知这种新需求。这个新需求的代码放在哪实现呢？要么放在持有List的地方，可能就是执行的地方：

```java
//标准库承载集合情况下，执行代码：
exec(Command_1 command){
    List<Action> actions = command.transform(transformContext);
    for(Action action : actions){
        action.exec(execContext);//在这里扩展
    }
}
```

如果放在持有List的地方，那么那里的职责就不单一了，本来只是一个简单调度的职责，现在还要考虑每一组Action的特定逻辑。

再要么入侵到每个Action中去：

```java
class Action {
    public void exec(ExecContext context){
        //...
        //在这里扩展
    }
}
```

如果入侵到每个Action中去，那么Action就会变得越来越臃肿，这也会让他变得越来越不稳定，理论上应该是越下层的越稳定，越上层的越频繁修改，这么一搞，作为下层的Action变成了最不稳定的，反而上层或者中层更稳定了，这个是不合理的。

那么抽象了集合对象，就可以直接在Actions里面扩展：

```java
//集合对象承载集合情况下，执行代码：
exec(Command_1 command){
    Actions actions = command.transform(transformContext);
    actions.exec(execContext);
}


class Actions {
    private List<Action> actions;

    void exec(ExecContext execContext){
        for(Action action : actions){
            action.exec(execContext);//在这里扩展
        }
    }
}
```

我们可以看到执行部分的代码因为逻辑没有变化，所以不需要改动代码，Action自身的逻辑没有什么变化，也不需要改动代码。而集合相关的逻辑因为需求发生了变化，在集合相关的类里进行扩展，这是非常合理的事情。我们常说，要建立统一语言，让业务人员与技术人员的认知模型保持一致，从而提高协同效率，那么在提取集合对象这个点上，如果我们做了这件事，那么一定程度上，我们就提升了它们的一致性。

## 屏蔽物理边界

其次，它可以屏蔽掉内存与物理存储的边界。当单体类之间有一对多关系的时候，如果我们使用语言自带的集合类库来定义类型，如之前的代码里：

```java
class User {
    private List<Order> orders;
}
```


这是非常常见的操作，但是这样的话，这个list里边的内容必须都存在于内存当中。实际中一个用户有多少订单呢？被经年累月使用的电商系统，哪怕是个人的订单，通常订单数量都大的可怕，你总不能都加载进来吧。类似的情况可以说在企业级的应用里非常常见。

因为这个非常非常现实的问题，很多orm提供了一些延迟加载的机制，你用的还是List，只是数据没有加载，直到List真的被访问的时候再加载数据，但是吧，延迟加载最终要么是全加载，要么是一条条加载，后者还因为性能有缺陷产生了专门的问题名词：n+1问题。如果我只关心部分呢？因为现实中数据大都是分页的，或者根据某个条件重新排序、过滤、分组然后再分页，这都很常见。所以这种情况，我们就只能专门为这个数据类型搞个dao或repository之类，像什么OrderRepository这种。代码就会变成下面这个样子：

```java
class User {
    private String id;
}

class Order {
    private String userId;
}

interface OrderRepository {
    List<Order> findAllByUserId(String userId);
    
    List<Order> findByUserId(String userId, int size, int offset);

    //....
}
```

但是这么一搞的话，所有的这种领域内的数据类就都有了自己的Repository，凡是数据大一点的，都会变成这个样子，一切都变得很碎片化，面向对象范式的价值被削弱了。

另外，类之间的关系就变得不清楚了，起码不能自动识别。这样在全局层面分析出的实体关系与实际的代码无法自动映射。知识传递过程中就出现了大量的隐性知识，在这里就是这个 String 类型的userId是User这个类的id这件事，这种隐性知识使得想要判断模型与代码是否一致就变得困难了，自然会增大维护成本，。


这种情况下，我们让user持有的不是list，而是orders，这个orders可能只是一个接口，我们可以给它加上各种具体的查询方法。

```java
class User {
    private Orders order;
}

interface Orders {
    List<Order> findAll();
    
    List<Order> findAll(int size, int offset);

    //....
}
```


于是不再是user持有order，而是user持有orders， orders 持有order。我们让这种集合对象承载了实体间的关系。这也就是所谓的associate object模式。

不过这个做法在技术实现上会比较的难一点，用MyBatis会比较容易一点，用JPA就会比较麻烦。集合对象的API的设计也有点反直觉，我们后续再写文章详细讲讲这个写法怎么落地。


## 可以根据状态切分上下文

第三，它可以按照状态进行归类，从而划分上下文。当我们抽象集合类的时候，不会傻乎乎的一个单体类就只抽象一个集合类， 比如order就抽象一个orders类。这样的话其实完全没有必要抽象，因为类本身就代表了全集这个概念，配上list等语言原生的集合类库自然就能搞定一切，绕了一圈又绕回去了。而我们需要做的是要把这个全集进行再切分，具体的手法往往是给集合类加上前缀。比如前面提到的user的orders，它的查询范围一定是这个用户的order，而不是所有的order，所以我们可以抽象一个UserOrders，于是前面的代码就变成了：


```java
class User {
    private UserOrders order;
}

interface UserOrders {
    List<Order> findAll();
    
    List<Order> findAll(int size, int offset);

    //....
}

class UserOrdersImpl implements UserOrders {

    private String userId;

    //.....

}
```

这个UserOrders查询order的时候会自然的加上用户id这个过滤条件，就实现了查询范围一定是这个用户的order这个能力，而且用起来也很方便：user.getOrders().findxxx。这只是简单应用，更有价值的的是我们可以用状态来切割漫长的业务逻辑。

比如有一个电商系统，用户需要下订单，订单就是我们的单体类。那么围绕订单往往是有一个很长的业务流程，在这个漫长的业务流程当中，订单有很多的状态。比如订单下达后，付款前，叫做待付款订单，付完款之后变成了待发货订单，发货之后变成了待收货订单。在每一个状态下，订单又有很多的分支业务逻辑。比如待付款订单可能有无数种付款方式，付完款之后相应的也可能有无数种确认方式。更不要说后面的待发货，待收货这些状态。每一种状态必然伴随着一大堆的业务逻辑代码，那么我们如何建模呢？一种方式就是建子类，待付款订单、待发货订单，待收货订单各是order的一个子类。这种方式不是不行，抛掉继承本身一些不太好的副作用问题不谈，这种建模的问题是只考虑了单体逻辑。也就是单个订单的逻辑。在业务当中还有一种情况，就是集合逻辑，尤其是这个集合被其他的单体类持有的时候，这种逻辑就完全没地方放。只能随意的对别人get出来，然后在用的地方自己编写，逻辑就是这么泄露的。这个时候呢，我们可以抽出他的集合类。比如叫PendingOrders。一个用户有很多order的时候，比如简单说有四种：全部order（UserOrders），待付款order(UserUnpaidOrders)，待发货order(UserUnsendOrders)，待收货order(UserUndeliveredOrder)，待评价order(UserUncommentedOrders)，各自建一组集合类。然后把围绕着他们的集合概念全都放到这些类里面去。你就会发现很多找不到地方放的逻辑突然有地方放了。

这个情况很常见，不只适用于电商领域，漫长的业务逻辑，与之伴生的状态，多种多样的分支，这很明显就会产生复杂的代码，这代码很容易腐化。

当我们按照状态为集合对象建模以后，对于建立统一语言也很有帮助，如果你经常去跟需求方去聊需求。你会发现需求方嘴中的类型和集合是不分的，他可能跟你说待付款订单，这个时候指的不是订单的子类，而是处于待付款这种状态的订单集合，很多人都会苦于如何在这种情况下跟需求方建立统一语言。如果你的建模工具箱里有封装集合对象这个工具，这一切都会就变得容易处理了。

## 收尾的话

最后，我们必须告诉大家，在面对复杂逻辑的时候，集合对象的抽取会带来很多好处，当然不是说语言自带的集合库就不能直接使用，抽取领域特定的集合类这种建模方式必须是在处理复杂逻辑、面对大量变化点的时候才会有好处，在简单逻辑的场景下就是自己人为的增加复杂度。如何判断是复杂逻辑的场景，则需要我们深入领域，充分理解领域知识，结合设计者丰富的经验，才能做出正确的判断。

参考资料


[1] 《OO Parterns》/（美）Peter Coad
[2] 《如何落地业务建模》/ 徐昊