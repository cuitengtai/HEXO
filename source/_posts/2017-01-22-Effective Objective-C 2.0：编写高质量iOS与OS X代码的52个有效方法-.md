---
layout: post
title: "《Effective Objective-C 2.0：编写高质量iOS与OS X代码的52个有效方法》笔记"
date: 2017-01-22
comments: true
categories: iOS
tags: [读书笔记]
keywords: 编写高质量iOS与OS X代码的52个有效方法
publish: true
description: 编写高质量iOS与OS X代码的52个有效方法
---
写出优雅高效的代码是每个程序员的最高追求，最近静下心里细细的品味一下一些提高代码质量的技巧，与君共勉之。

## 熟悉Objective-C

### 了解Objective-C语言的起源

1、区别于C++, Java等面向对象的语言，Objective-C在 运行中是“消息转发”而非“函数调用”，运行时所执行的代码由运行环境决定，而非编译器决定。
2、NSString *someString = @"The string"; someString指针变量被分配在“栈帧”中，@“The string”对象则被分配在“堆空间”中，分配在堆中的内存必须直接管理，而分配在栈上用于保存变量的内存则会在其“栈帧”自动弹出时清理。在Objective-C中，内存管理指的是堆上的这部分内存。

### 在类的头文件中尽量少引用其他头文件

1、不要为了省事，直接在头文件中引用其他类的头文件，将头文件引入的时间尽量延后，可以用"@class"来声明类，减少编译器的负担。
2、如果两个类互相引入对方的头文件会造成“循环引用”，无法正常编译。经测试：只互相引用头文件可正常编译，但是如果在类里面使用对方头文件变量时，就无法编译通过。
3、某个类如果要实现某个协议，尽量把协议的声明写在“类扩展”，即".m"文件的"@interface xxx<XXXDelegate> @end",如果实在不行，那就最好吧协议单独放在一个头文件中，然后将其引入。
	
### 多用字面量语法，少用与之等价的方法

好处:
1、书写简洁，便于阅读。
2、使用字面量数组时可以更快的抛出异常，更加安全。
局限性:
1、自定义创建子类的实例，需要采取“非字面量语法”。
### 多用类型常量，少用#define预处理指令

<del>#define ANIMATION_DURATION 0.3</del>

```
static const NSTimeInterval kAnimationDuration = 0.3
```

1、如果不打算公开变量，可在.m文件中，同时使用static和const来定义常量，如果试图修改该变量编译器就会报错。这种变量只在编译单元内可见，由于不在”全局符号表“中所以无需增加前缀。
2、如果打算公开某个常量，则需要这样来实现
```
// EOCAnimatedView.h
extern const NSTimerInterval EOCAnimatedViewAnimationDuration;
// EOCAnimatedView.m
const NSTimerInterval EOCAnimatedViewAnimationDuration = 0.3;
```
	
在头文件中声明，在实现文件中定义，由于在”全局符号表“中，为避免名称冲突，最好用与之相关的类名作为前缀。这样定义常量是不可被更改的，而使用#define预处理指令定义的常量很可能遭到别人的无意更改，这是非常危险的事情。
	
### 多用类型常量，少用#define预处理指令
1、使用NS_ENUM与NS_OPTIONS宏来定义枚举类型并指明其数据结构。
2、应该用枚举来表示状态机的状态，状态码等值。
3、如果枚举的多个选项可以同时组合，将这些项定义为2的幂，以便可以通过按位或操作将其组合起来。
例如：AnimationEasyIn | AnimationLeft
```
typedef NS_ENUM(NSUInteger, AnimationBehavior) {
AnimationEasyIn = 1 << 0,
AnimationLeft = 1 << 1
};
```
4、处理枚举类型的switch语句中不要实现default分支，这样的话有新枚举加入，编译器就会提示开发者，switch语句中并未处理所有的枚举。
## 对象、消息、运行期
### 理解“属性”这一概念
1、尽量不要直接定义实例变量，推荐使用属性"@proproty"来合成存取方法。使用点语法调用来访问变量。
2、在设置属性所对应的实例变量时，一定要遵循该属性所声明的语义。
即：
```
@property (nonatomic, copy)NSString *name;
	
- (void)setName:(NSString *)name
{
    _name = [name copy];
}
疑问：
- (void)setName:(NSString *)name
{
   // _name = [name copy];
   _name = name;
}
```
经过测试，使用_name = name，和上面的效果是一样的，内存地址发生了变化。
### 在对象内部尽量直接访问实例变量
强烈建议在读取实例变量时候采用直接访问的形式，而在设置实例变量的时候通过属性来做。

理由：
1、直接访问属性，而不经过存取方法，直接访问那块内存，速度当然比较快，但是可以忽略不计。
2、直接访问属性就绕过了如“strong”、“copy”,等“管理语义”，如果在ARC下直接访问一个声明为copy的对象，并不会拷贝改属性，只会保留新值并释放旧值。
3、直接访问实例变量，不会触发“KVO”，
合理的方案：
在对象内部读取数据时，直接通过实例变量来读，在写入数据时，通过属性来写。
注意：
1、在初始化方法和dealloc方法中，应该直接访问实例变量，防止子类覆盖设置方法。
2、懒加载中，通过获取方法来访问属性，否则实例变量永远不会被初始化。
```
- (Person *)person {
	if (!_person) {
		_person = [Person alloc] init];
	}
	return _person;
}
```

### 理解“对象等同性”这一概念
1、重写isEqual和hash方法，判断对象的等同性。
2、利用标识符，类似于数据库的主键，来判断对象的等同性。
3、容器中放入可变对象时，慎重改变其哈希码，如：NSMutableSet中加入NSMutableArray。
4、不盲目地检查每条属性，按照具体需求来制定检测对象。
5、相同的对象必须具有相同的哈希码，但是哈希码相同的对象未必相同。
6、编写hash方法时，应该使用计算速度快而且哈希码碰撞几率低的算法。

[哈希介绍](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E8%A1%A8)
[Equality](http://nshipster.cn/equality/)

### 以“类族模式”隐藏实现细节
1、使用类方法创建实例，如：+ (UIButton *)buttonWithType:
2、从类族的公共抽象基类中继承子类时要当心，子类应当覆写父类中指明需要覆写的方法
### 在既有类中使用关联对象存放自定义数据
“关联对象"(Associated Object),可以在外部给类添加属性，常用在Category中添加。
### 理解objc_msgSend的作用
1、“静态绑定”编译器在编译阶段就能生成调用函数指令，“动态绑定”运行期才可以生成函数调用指令。
2、objc_msgSend函数会在接收者所属类中的“方法列表”中寻找符合的方法，如果找到就执行实现代码，如果找不到，就沿着继承体系，向上继续寻找，如果最终还是找不到，就执行“消息转发操作”，此处可进行手动拦截。
3、尾调用优化对于递归具有重要意义，大大减少调用栈的调用记录，防止栈溢出。
	
[尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)
### 理解消息转发机制
1、运行时系统没有找到消息的接收者，会发起消息转发，给接收者最后一次机会，所有的细节封装在NSInvocation对象中，在此可以进行拦截处理。
2、对象无法收到解读的消息后，首先调用其所属类的类方法：
第一步：
+ (BOOL)resolveInstanceMethod:(SEL)selector 此处可以使用class_addMethod()添加新方法。
第二步：为消息寻找一个备用的接收者，但无法操作这一步的消息。
- (id)forwardingTargetForSelector:(SEL)selector 
第三步：若发现某调用操作不应该由本类处理，则需要调用超类的同名方法，这样，继承体系的每个子类都有机会处理此调用请求，直至NSObject,如果最后调用了NSObject的方法，会抛出异常，表明消息未得到处理。
- (void)forwardingInvocation:(NSInvocation *)invocation

### 用“方法调配技术”测试“黑盒方法”
在运行期，通过runtime向类中添加或者交换方法实现。
### 理解“类对象”的用意
1、如果类型无法在编译器确定，那么就应该使用类型信息查询方法来探知。isMemberOfClass:判断对象是否为某个类的实例，isKindOfClass:判断对象是否为某个类或者其子类的实例。
2、尽量使用类型信息确定对象类型，不要直接比较类对象，因为某些对象可能实现了消息转发功能。
如：	
<del>if（[object class] == [Person class]）{
}<del>
```
if（[object isMemberOfClass:[Person class]] {
}
```

## 接口与API设计
### 用前缀避免命名空间冲突
1、选择与你公司，应用程序或者与之相关的名称作为类名前缀，并在所有代码中统一所有前缀。
2、在Category中给变量，方法添加前缀。
3、若自己所开发的程序中用到了第三方库，则应为其中的名称加上前缀。
4、所选的前缀应为三个字母（苹果规定的）。
### 提供全能的初始化方法
1、在类中提供一个全能的初始化方法，供其他初始化方法调用。
2、若全能初始化方法与超类的不同，则需要覆写超类中对应的方法。
3、如果超类的初始化方法不适用于子类，那么应该覆写这个超类方法，并在其中抛出异常。
```
-(id)initWithWidth:(float)width andHeight:(float)height
{
   @throw [NSException exceptionWithName:...];
}
```

### 实现description方法
1、实现description方法返回一串有意义的字符串，来描述该对象，使用NSLog()打印对象时调用。
2、实现debugDescription方法返回一串有意义的字符串，来描述该对象，在开发者断点调试通过LLDB命令po对象时调用。
### 尽量使用不可变对象
1、尽量把对外部公布出去的属性设置为只读(readonly)
2、如果想修改封装在对象内部的数据，但是又不想别人在外部修改，这时需要在.m文件中重新把属性声明为readwrite，但是这样会产生“竞态条件”，需要将对象的所有数据进行同步，参见41条。
3、通过KVC仍可以修改只读属性的值，这时不符合API规范的。
4、不要把可变的集合对象作为属性公开，而是提供相应的方法修改集合对象。
### 使用清晰而协调的命名方式
1、尽量使用长方法名，清晰的说明方法的意思，但是也要尽量言简意赅，可参考UIKit命名规范。

<del>-(id)initWithSize:(float)width :(float)height<del>
```
-(id)initWithWidth:(float)width andHeight:(float)height
```

### 为私有方法名加前缀
1、给私有方法的名称加上前缀，便于跟公共方法区分。
2、不能用单一下划线做前缀，这是苹果私有API的前缀，会冲突。
### 理解Objective-C错误模型
1、“-fojc-arc-exception”编译标识可以在执行异常代码时不抛出异常
2、异常用于处理致命问题，无需考虑恢复问题，应用程序直接退出，非致命问题使用NSError对象。
3、通过“输出参数”方式把NSError对象回传给调用者
``` 
NSError *error = nil;
BOOL ret = [object doSomething:&error];
if (ret) {
   // There was an error
}
- (Bool)doSomething:(NSError **)error {
   if (// there was an error) {
       if (error) {
           *error = [NSError errorWithDomain:...];
       }
       return NO;
   } else {
       return YES;
   }
}
```
    
在error要指向一个新的对象时（解引用），必须先保证error参数不是nil,因为空指针解引用会导致“段错误”，并使应用程序崩溃。调用者在不关心具体错误时，会给error参数传入nil,因此必须判断这种情况。
### 理解NSCopying协议
1、想让自定义对象具有拷贝功能，需要实现NSCopying协议。    
2、对象拷贝分为深拷贝和浅拷贝，一般情况下尽量执行浅拷贝。
3、如果对象需要深拷贝，可以考虑增加一个专门执行深拷贝的方法。
4、如果自定义对象分为可变和不可变版本，需要同时实现NSCopying和NSMutableCopying协议。

## 协议与分类
### 通过委托与数据源协议进行对象间通信
1、委托模式为对象提供一套接口，通过接口可将相关事件告知其他对象。
2、将委托对象应该支持的接口定义成协议，在协议中将可能需要处理的事件定义为方法。
3、当使用委托模式传递数据时，该模式也叫“数据源协议”，如UITableViewDataSource
4、使用“位域”（含有位段的结构体），将委托对象是否响应相关协议的信息缓存其中。
```
struct {
   unsigned int didReceiveData : 1;
   unsigned int didFailWithError : 1;
   unsigned int didUpdateProgressTo : 1;
} _delegateFlags;
    
- (void)setDelegate:(id<...>)delegate {
   _delegateFlags.didReceiveData = [delegate respondsToSelector:@selctor(...)];
}
    
if (_delegateFlags.didReceiveData) {
   [_delegate ....];
}
```

### 将类的实现代码分散到便于管理的数个分类之中
1、使用分类机制把类的实现代码划分成易于管理的小块。
2、将应该私有的方法归入名叫Private的分类中，以隐藏实现细节。
### 总是为第三方类的分类名称加前缀
1、如果多个分类重载“主实现”的相关方法，以最后一个分类为主，即：分类在项目中编译的顺序。
2、以命名空间来区分分类中所定义的方法，三个小写字母加下划线:abc_xxx
### 请勿在分类中声明属性
1、分类不可以定义属性，如果非要这么做，可以使用“关联对象”。
2、把封装数据所用的全部属性都定义在主类中。
3、可以定义存取方法但是尽量不要定义属性。
### 使用“class-continuation分类”隐藏实现细节
“class-continuation分类”也叫“类扩展”。
1、通过“class-continuation分类”，向类中增加实例变量。
2、私有方法声明在“class-continuation分类”中。
3、私有的协议也可以声明在“class-continuation分类”中。
### 通过协议提供匿名对象
1、协议可以在某种程度上提供匿名类型，具体对象类型可以淡化为遵从某一协议的id类型，协议里规定了对象所应该实现的方法。
2、使用匿名对象id<xxx>来隐藏类型名称。
3、如果具体类型不重要，重要的是对象能够响应协议里的特定方法，那么可以用匿名对象来表示。也就是说，只暴露使用者关心的，隐藏其不需要关注的。
## 内存管理
### 理解引用计数器
1、非ARC通过引用计数器增减来管理内存，对象创建好之后，引用计数器至少为1，若引用计数为正，对    象继续存活，当保留计数降为0时，对象就被销毁了。
2、在对象生命周期中，其他对象通过引用来保留或者释放此对象，保留与释放操作分别会使引用计数递增和递减。
3、不要操作“悬挂指针”（野指针），会引起崩溃，最好将其变为“空指针”。
4、set方法 strong retain新值，release旧值，然后赋值。
5、autoreleasepool能延长对象生命期，在下次运行循环或者更早一些release对象。
6、使用“弱引用”可以防止循环引用。
### 以ARC简化引用计数
1、ARC会自动补上引用计数代码，但是并没有使用Objective-C的消息转发，而是直接通过C代码，性能会更好。
2、ARC只管理Objective-C对象的内存，CoreFoundation对象不归ARC管理，开发者必须适时调用CFRetain/CFRelease。
3、ARC对引用计数进行了特殊优化，如果发现对象在运行期进行多次保留和释放操作，会成对的移除这些操作，只保留有用操作，从而提高效率。
4、ARC对存取方法进行了优化，不需要再去关注先保留新值，再释放旧值，直接赋值即可。
5、ARC借助Objective-C++的特性清理对象，回收Objective-C对象调用多有C++对象的“析构函数"，如发现某个对象里面有C++对象，会生成.cxx_destruct的方法，并在该方法中生成清理内存所需代码。如果有非Objective-C对象（CoreFoundation对象或者malloc()分配的堆内存），不会调用父类的dealloc方法，会生成.cxx_destruct的方法，并在该方法中生成清理内存所需代码，在生成的代码中自动调用父类的dealloc方法。因此可以这样写：
```
-(void)dealloc {
   CFRelease(...);
   free(...);
}
```

### 在dealloc方法中只释放引用并解除监听
1、可以在dealloc方法中注销通知观察者等。
2、比如文件，套接字，大块内存等开销较大或者系统内稀缺资源不可以在dealloc方法中释放，需要用完即释放。
3、系统并不能保证每个创建的对象的dealloc方法都会调用，个别对象在应用程序终止时仍处于存活状态，由于应用程序终止，将内存返还给操作系统，所以实际上这些对象也算是销毁了。所以在系统回收之前我们可以手动清理这些对象，在UIApplicationDelegate/NSApplicationDelegate的委托方法中处理比较合适。
4、不要再dealloc方法中随便调用别的方法，存取方法也不要调用，因为这样可能会触发观察者，还是使用下划线比较稳妥。
### 编写”异常安全代码“时留意内存管理问题
1、捕获异常时，一定要将try块内的对象清理干净。
2、在默认情况下，ARC不生成安全处理异常所需的清理代码，开启”-fobjc-arc-exceptions“标识后，可生成这种代码，不过会导致应用程序变大，而且会降低运行效率。
3、如果发现大量异常捕获操作时，考虑使用NSError错误传递来重构。
### 以弱引用避免保留环
1、某些引用设置为weak可以避免出现”保留环“（循环引用）。
2、weak与unsafe_unretained的区别：
相同：都是弱引用，用来表示”非拥有关系“。
不同：weak 系统如果把属性回收，属性自动设置为nil；unsafe_unretained 系统如果把属性回收，属性仍然指向那个已经回收的实例，也就是”野指针“，继续操作属性会使程序崩溃，
总的来说，weak更加安全一些，但是性能比unsafe_unretained低些。
3、一般来说，如果不拥有某个对象就不要去保留它。这条规则对集合类对象例外，因为集合类对象并不直接拥有其内容，而是通过自己内部的元素来保留这个对象。使用weak尽量也不要去使用一个已经释放的弱引用对象，尽管不会崩溃，但是这依然是一个bug。
### 以”自动释放池块“降低内存峰值
1、自动释放池排布在栈中，对象收到autorelease消息后，系统将其放入最顶端的池里。
2、合理利用自动释放池可以降低内存峰值，如：在for循环内部使用自动释放池包裹代码。
3、@autoreleasepool这种写法可以创建更轻便的自动释放池。
### 用”僵尸对象“调试内存管理问题
1、系统再回收对象时，可以不将其真的回收，而是把它转化成僵尸对象，通过环境变量NSZombieEnabled可以开启此功能。
2、系统会修改对象的isa指针，另其指向特殊的僵尸类，从而使该对象变为僵尸对象，僵尸类能够响应所有的选择子，响应方式为：打印一条包含信息内容及接收者的消息，然后终止程序。
### 不要使用retainCount
1、在ARC中retainCount方法已经被废弃，因为它只能返回特定时间点的引用计数器，而未考虑系统稍后会吧自动释放池清空的情况。或者是retainCount永远不返回0，因为系统可能会优化对象的释放行为，在引用计数是1的时候就回收对象，引用计数可能永远不会是0，所以引用计数是不准确的，就算能够正常返回时凭运气。
2、单例对象的引用计数是不会改变的，其保留和释放都是空操作，retainCount已经被废弃了，任何对象的retainCount都是不确定的，所以retainCount绝对不要用。
## 块与大中枢派发
### 理解“块”这一概念
1、默认情况，block所捕获的变量不能被修改，声明变量时加上__block就可以修改。但是类的实例变量不需要加__block即可被修改。
2、使用block小心因捕获了self而造成循环引用的情况。如果访问了类的实例变量，self也会被捕获。
3、block可以分配在栈或者堆上，也可以是全局的。分配在栈上的block可以拷贝到堆上，这样的话，就和标准的Objective-C对象一样，具备引用计数了。
4、全局的block无法捕获任何状态（变量），运行时也无需状态参与，所使用的内存区域在编译时就确定了。
```
void (^block)() = ^{
   NSLog("This is a block!");
}；
```

### 为常用的块类型创建typedef
1、以typedef重新定义block类型，可令块block变量用起来更加简单，可读性更好。
2、定义新类型时遵循现有命名习惯，不要发生命名冲突。
3、不妨为同一个block定义多个类型别名，如果要重构的代码使用了block类型的某个别名，那么只需修改相应typedef中的签名即可，无需改动其他typedef。
### 用handler块降低代码分散程度
1、使用block来取代delegate可以使代码更加紧凑，便于阅读。
2、在网络请求的API设计中，尽量用一个handlerl来处理数据及错误，更加灵活。
推荐：
```
[fetcher startWithCompletionHander:^(NSData *data, NSError *error){
   if (error) {
       // failed
   } else {
       // success
   }
}];
```
  
不推荐：
```
[fetcher startWithCompletionHandler:^(NSData *data){
   // success
} failureHandler:^(NSError *error){
   // failed
}];
```
    
3、设计API是如果用到handler块，可以增加一个参数，使调用者可以通过此参数来决定应该把块安排在哪个队列执行。
### 使用块引用其所属对象时不要出现保留环
1、如果块所捕获的对象直接或者间接保留了块本身，那么就会造成保留环问题。
2、一定要找一个适当的时机解除保留环，而不能把责任推给API的调用者。
以下是两种会出现保留环的写法：
```
1、_fetcher捕获了block block捕获了_fetcher
_fetcher = [[xxx alloc] init];// 属性
[_fetcher startWithCompletionHander:^(NSData *data, NSError *error){
       NSLog(@"%@", fetcher.url);
       _fetchData = data;
}];
2、fetcher捕获了block block捕获了fetcher.url
xxx *fetcher = [[xxx alloc] init];// 临时变量
[fetcher startWithCompletionHander:^(NSData *data, NSError *error){
       NSLog(@"%@", fetcher.url);
       _fetchData = data;
}];
```

解决方案：
1、weakSelf和strongSelf。
2、在block调用后置为nil。
### 多用派发队列，少用同步锁
1、使用锁可以用来实现同步机制，但会带来性能甚至死锁的问题。
2、将同步与异步派发结合起来，可以实现与普通锁一样的同步行为，而这么做也不会阻塞执行异步派发的线程。
3、使用同步队列及栅栏块（dispatch_barrier_xxx），可以令同步行为更加高效。
### 多用GCD，少用performSelector系列方法
1、使用[object performSelector:@selector(selectorName)]方法可能会造成内存泄漏，ARC不会添加相关的释放操作，因此有可能会造成内存泄漏。
2、可以使用GCD来代替。
### 掌握GCD及操作队列的使用时机
1、GCD并不是最优的办法，在某些情况下使用操作队列（NSOperation）可能会更好一些。
2、NSOperation的优势：取消某个操作，制定操作的依赖关系，通过KVO监控NSOperation对象属性，指定操作优先级。
3、GCD基于任务的队列，NSOperation基于操作的队列。
### 通过Dispatch Group机制，根据系统资源状况来执行任务
1、一系列任务可放在一个dispatch group中，开发者可以在这组任务执行完获得通知。
2、通过dispatch group，可以并发执行多项任务，GCD会根据系统资源来调度执行任务。
### 使用dispatch_once来执行只需运行一次的线程安全代码
1、使用dispatch_once实现单例可以保证线程安全。
2、标记声明为static或者global作用域中，这样可以保证执行dispatch_once函数时，传入的标记也是相同的。
### 不要使用dispatch_get_current_queue
1、dispatch_get_current_queue已经废弃，最好不要使用，可能会产生死锁问题。
## 系统框架
### 熟悉系统框架
请记住：用纯C写成的框架与用Objective-C写成的一样重要，若想成为优秀的Objective-C开发者，应掌握C语言的核心概念。
### 多用块枚举，少用for循环
1、遍历集合有四种方式，最基本的办法是for循环，其次是NSEnumerator遍历法及快速遍历法，最新，最先进的方式是“块枚举法”，也就是block。
2.“块枚举法”通过GCD并发执行遍历操作，效率较高，推荐使用。
3、若提前知道待遍历的collection含有何种对象，则应修改块签名，指出对象的类型。
### 对自定义其内存管理语义的collection使用无缝桥接
1、__bridge本身意思是：ARC仍然具备这个Objective-C对象的所有权。__bridge_retained则与之相反，意味着ARC交出所有权，后面需要加上CFRelease()以释放对象。
2、通过无缝桥接技术，可以在Foundation框架中的Objective-C对象与CoreFoundation框架中的C语言数据结构之间来回转换。
2、在CoreFoundation层面创建collection时，可以指定许多回调函数，这些函数表示此collection应该如何处理元素，然后，可以运用无缝桥接技术，将其转换成具备特殊内存管理语义的Objective-C collection。
### 构建缓存时选用NSCache而非NSDictionary
1、NSCache胜过NSDictionary之处在于，当系统资源将要耗尽时，他可以自动删减缓存。NSCache并不会“拷贝”键，而是会”保留“它，而且是线程安全的。
2、可以给NSCache对象设置上限，用以限制缓存中的对象总个数及“总成本”，而这些尺度则定义了缓存删减其中对象的时机。但是绝对不要把这些尺度当成可靠的“硬限制”，他们仅对NSCache起到指导作用。
3、将NSPurgeableData与NSCache搭配使用，可实现自动清除数据的功能，也就是说当NSPurgeableData对象所占内存为系统所丢弃时，该对象自身也将从缓存中清除。
4、如果缓存使用得当，那么应用程序的响应速度就能提高。只有那种“重新计算起来很费事的”数据，才值得放入缓存，比如那些需要从网络获取或者从磁盘中读取的数据。
### 精简initialize与load的实现代码
1、+ (void)load 当包含类或者分类的程序载入系统时，就会执行此方法，通常是指程序启动的时候，此时运行时是混乱的状态，如果使用了其他的类，其他的类并不一定是初始化好了的。
2、load方法的实现尽量精简，它不遵循那套继承规则，很难确定执行顺序，且load方式执行时应用会阻塞，如果在该方法里面执行了大量的耗时操作，那么程序就会无响应，所以一般不要使用此方法。
3、+(void)initialize 会在程序首次使用该类之前调用，且只调用一次。与load方法的区别：首先它是“惰性”调用，只有程序在用到该类才会调用。此时运行时是正常状态，且执行环境是线程安全的。
4、在编写load方法与initialize要保证代码实现尽量简单，除了初始化全局属性外，最好也不要调用自己的其他的方法。如：无法再编译期设定的全局常量，可以放在initialize方法里面初始化。

### 别忘了NSTimer会保留其目标对象
1、NSTimer对象会保留其目标对象，直到计时器本身失效为止，调用invalidate方法可令计时器失效，另外，一次性的计时器在触发完任务之后也会失效。
2、反复执行任务的计时器很容易引入保留环，如果计时器的目标对象又保留了计时器本身，那么肯定会导致保留环，这种保留环关系，可能是由于直接或者间接发生的。
3、可以扩充NSTimer的功能，用block来打破保留环。
   // 把block作为userInfo的参数传入，一定要先拷贝到堆上，不然一会执行block时可能就被释放了。
```
+ (void)_yy_ExecBlock:(NSTimer *)timer {
  if ([timer userInfo]) {
      void (^block)(NSTimer *timer) = (void (^)(NSTimer *timer))[timer userInfo];
      block(timer);
  }
}
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)seconds 
                                     block:(void (^)(NSTimer *timer))block 
                                   repeats:(BOOL)repeats {
                                 
  return [NSTimer scheduledTimerWithTimeInterval:seconds 
                                          target:self
                                        selector:@selector(_yy_ExecBlock:)
                                        userInfo:[block copy]
                                         repeats:repeats];
}
```

    

  


   




