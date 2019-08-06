# 领域模型
```md
领域模型并不完成业务，每个domain object都是完成属于自己应有的行为（single responsibility），
就如同人跑这个动作，person.run是一个与业务无关的行为。

但这个时候manger或者service在调用 some person.run 的时候可能完成的100米比赛这个业务，也可能是完成是跑去送外卖这个业务。
```

## 领域模型:失血模型、贫血模型、充血模型、胀血模型 - Martin Fowler
```md
“血”指的是domain object的model层内容。
```
* 失血模型
```md
基于数据库的领域设计方式其实就是典型的失血模型。
以Java为例，POJO 只有简单的基于field的setter，getter方法，POJO之间的关系隐藏在对象的某些ID里，
由外面的manager解释，比如son.fatherId，Son并不知道他跟Father有关系，但manager会通过son.fatherId得到一个Father。
```
```md
失血模型中，domain object只有属性的get set方法的纯数据类，所有的业务逻辑完全由Service层来完成的，
由于没有dao，Service直接操作数据库，进行数据持久化。
service:  肿胀的服务逻辑
model: 只包含get set方法
```
* 贫血模型
```md
贫血模型中，domain object包含了不依赖于持久化的原子领域逻辑，而组合逻辑在Service层。
service ：组合服务，也叫事务服务
model：除包含get set方法，还包含原子服务
dao：数据持久化
```
```md
儿子不知道自己的父亲是谁是不对的，不能每次都通过中间机构（Manager）验DNA(son.fatherId)来找爸爸，领域模型可以更丰富一点。
```
```java
public class Son{
	private Father father;
	public Father getFather(){return this.father;}
}
public class Father{
	private Son son;
	private Son getSon(){return this.son;}
}
```
```md
通常一个object是通过一个repository（数据库查询），或者factory（内存新建）得到的
为了构建完整的son对象，sonRepo里需要一个fatherRepo来构建一个father去赋值son.father。
而fatherRepo在构建一个完整father的时候又需要sonRepo去构建一个son来赋值father.son。
这形成了一个无向有环圈，这个循环调用问题是可以解决的，但为了解决这个问题，领域模型会变得有些恶心和将就。
有向无环才是我们的设计目标，为了防止这个循环调用。
我们是否可以在father和son这两个类里省略掉一个引用？
```
```java
public class Father{
	//private Son son; 删除这个引用
	private SonRepository sonRepo;//添加一个Son的repo
	private getSon(){return sonRepo.getByFatherId(this.id);}
}
```
```md
这样在构造Father的时候就不会再构造一个Son了，但代价是我们在Father这个类里引入了一个SonRepository, 
也就是我们在一个domain对象里引用了一个持久化操作，这就是我们说的充血模型。 
```
* 充血模型
```md
充血模型中，绝大多业务逻辑都应该被放在domain object里面，包括持久化逻辑，
而Service层是很薄的一层，仅仅封装事务和少量逻辑，不和DAO层打交道。

service ：组合服务 也叫事务服务
model：除包含get set方法，还包含原子服务和数据持久化的逻辑
```
```md
充血模型和第二种模型差不多，所不同的就是如何划分业务逻辑，即认为，绝大多业务逻辑都应该被放在domain object里面(包括持久化逻辑)

而Service层应该是很薄的一层，仅仅封装事务和少量逻辑，不和DAO层打交道。 
Service(事务封装) ---> domain object <---> DAO 
这种模型就是把第二种模型的 domain object和 business object合二为一了。

所以ItemManager就不需要了，在这种模型下面，只有三个类:

Item：包含了实体类信息，也包含了所有的业务逻辑 
ItemDao：持久化DAO接口类 
ItemDaoHibernateImpl：DAO接口的实现类 

在这种模型中，所有的业务逻辑全部都在Item中，事务管理也在Item中实现。
```
* * 优点
```md
更加符合OO的原则 
Service层很薄，只充当Facade的角色，不和DAO打交道。 
```
* * 缺点： 
```md
DAO和domain object形成了双向依赖，复杂的双向依赖会导致很多潜在的问题。 
如何划分Service层逻辑和domain层逻辑是非常含混的，在实际项目中，由于设计和开发人员的水平差异，可能导致整个结构的混乱无序。 
考虑到Service层的事务封装特性，Service层必须对所有的domain object的逻辑提供相应的事务封装方法，其结果就是Service完全重定义
充血模型的存在让domain object失去了血统的纯正性，他不再是一个纯的内存对象，这个对象里埋藏了一个对数据库的操作，这对测试是不友好的。
```
* 胀血模型 
```md
胀血模型取消了Service层，只剩下domain object和DAO两层，在domain object的 domain logic上面封装事务。
```
```md
基于充血模型的第三个缺点，有同学提出，干脆取消Service层，只剩下domain object和DAO两层，
在domain object的domain logic上面封装事务。 
domain object(事务封装，业务逻辑) <---> DAO 
似乎ruby on rails就是这种模型，他甚至把domain object和DAO都合并了。 
```
```md
该模型优点： 
1、简化了分层 
2、也算符合OO 
```
```md
1、很多不是domain logic的service逻辑也被强行放入domain object ，引起了domain object模型的不稳定 
2、domain object暴露给web层过多的信息，可能引起意想不到的副作用。
```
* 总结
```md
在这四种模型当中，失血模型和胀血模型应该是不被提倡的。而贫血模型和充血模型从技术上来说，都已经是可行的了。
贫血模型和充血模型的差别在于，领域模型是否要依赖持久层，贫血模型是不依赖的，而充血模型是依赖的。
```
* 参考
[领域模型之失血、贫血、充血、胀血模型](https://jlife.iteye.com/blog/625018)

## 领域模型下的依赖注入
```md
依赖注入在runtime是一个singleton对象，只有在spring扫描范围内的对象（@Component）才能通过annotation（@Autowired）用上依赖注入，
通过new出来的对象是无法通过annotation得到注入的

个人推荐构造器依赖注入，这种情况下测试友好，对象构造完整性好，显式的告诉你必须mock/stub哪个对象
```
## 领域模型：测试友好
```md
失血模型和贫血模型是天然测试友好的（其实失血模型也没啥好测试的）,因为他们都是纯内存对象。
但实际应用中充血模型是存在的，要不就是把domain对象拆散，变得稍微不那么优雅（当然可以，贫血和充血的战争从来就没有断过）。
那么在充血模型下，对象里带上了persisitence特性，这就对数据库有了依赖，mock/stub掉这些依赖是高效单元化测试的基本要求
```
## 领域模型:repository的实现方式
```md
设计了Tunnel这个接口，通过这个接口我们可以实现对domain对象在不同类型数据库的存取。
Repository并没有直接进行持久化工作，而是将domain对象转换成POJO交给Tunnel去做持久化工作，Tunnel具体实现可以在任何包实现，
这样，部署上，domain领域模型（domain objects+repositories）和持久化(Tunnels)完全的分开，domain包成为了单纯的内存对象集。
```

## 领域模型下的部署架构
