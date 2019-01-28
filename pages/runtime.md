### **RunTime**
### 简介
- RunTime即运行时。Objective-C就是运行时机制，也就是在运行时的一些机制，其中最重要的是消息机制。
- 对于Objective-C的函数，属于动态调用过程，在编译的时候并不能决定真正调用哪个函数，只有在真正运行的时候才会根据函数的名称找到对应的函数调用。
- 在编译阶段，Objective-C可以调用任何函数，即使这个函数并未实现，只要声明过就不会报错；但C语言调用未实现的函数就会报错。

### 作用
- **1.发送消息**
>1. 方法调用的本质就是让对象发送消息。
>2. objc_msgSend,只要对象才能发送消息，因此以objc开头。（#import<objc/message.h>）
>3. 消息机制原理：对象根据方法编号SEL去映射表查找对应的方法实现。

![](/images/runtime_03.jpeg)
```
// 创建User对象，对象方法
        User * user = [[User alloc] init];
        [user eat];
        // 本质是让对象发送消息
        objc_msgSend(user,@selector(eat));
        // 类方法
        [User drink];
        [[User class] drink];
        // 本质是让类对象发送消息
        objc_msgSend([User class], @selector(drink));
```

**warning：**

报错：`Too many arguments to function call, expected 0, have 2`
![](/images/runtime_02.jpeg)

解决方法：
![](/images/runtime_01.jpeg)
- **2.交换方法**
>1. 开发场景：系统自带的方法不够，给系统自带的方法扩展一些功能，并保持原有的功能。
>2. 一：继承系统的类，重写方法；二：使用runtime交换方法。

`需求：`给imageNamed方法提供功能，每次加载图片就判断图片是否加载成功。

第一步：先定义一个分类，定义一个能加载图片并且能打印的方法。 `+ (instancetype)imageWithName:(NSString *)name;`

第二步：交换imageNamed和imageWithName的实现，就能调用imageWithName间接调用imageWithName的实现。
```
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //
    UIImage * image = [UIImage imageNamed:@"sss"];

}
// UIImage分类实现

#import "UIImage+Image.h"
#import <objc/runtime.h>
@implementation UIImage (Image)
// 加载分类到内存的时候调用
+ (void)load{
    // 获取方法地址
    Method imageWithName = class_getClassMethod(self, @selector(imageWithName:));
    Method imageName = class_getClassMethod(self, @selector(imageNamed:));
    // 交换方法地址，相当于交换实现方式
    method_exchangeImplementations(imageWithName, imageName);
}
// 不能再分类中重写系统方法imageNamed,因为会把系统的功能给覆盖掉，而且分类中不能调用super。
+ (instancetype)imageWithName:(NSString *)name{
    // 调用imageWithName，相当于调用imageName
    UIImage * image = [self imageWithName:name];
    if (image == nil) {
        NSLog(@"加载空的图片");
    }
    return image;
}
@end
```
- **3.动态添加方法**
>1. 开发使用场景：如果一个类方法非常多，加载到内存的时候也比较耗资源，需要给每个方法生成映射表，可以使用动态给某个类添加方法解决。
```
// 默认user没有实现run方法，可以通过performSelector调用，但会报错
// 动态添加方法就不会报错
// 动态添加的方法是无法直接调用的，必须用performSeletor:调用。因为performSelector是运行时系统负责去找方法的，在编译时候不做任何校验，如果直接调用是会自动校验。
User * user = [[User alloc] init];
[user performSelector:@selector(run)];
[user performSelector:@selector(fly)];
[user performSelector:@selector(drive)];
[user performSelector:@selector(drived)];
```

方案一: `+(BOOL)resolveInstanceMethod:(SEL)sel`
```
// User.m
// 当调用方法时，系统会调用resolveInstanceMethod这个方法。
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(run)) {
        IMP imp = class_getMethodImplementation(self, @selector(run));
        /*
         参数一：Class——指的是当前类
         参数二：SEL——方法，即将动态添加的方法
         参数三：IMP——IMP就是Implementation的缩写，它是指向一个方法的指针，每一个方法都有一个对应的IMP
         参数四：const char * _Nullable types
         ”v@:”意思就是这已是一个void类型的方法，没有参数传入。
         “i@:”就是说这是一个int类型的方法，没有参数传入。
         ”i@:@”就是说这是一个int类型的方法，有一个对象参数传入。
         v表示返回值为void，@表示self，:表示_cmd。
         */
        class_addMethod([self class], sel, imp, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
// User.m
void run(id self,SEL sel){
    NSLog(@"I'm running");
}
```
方案二: `(id)forwardingTargetForSelector:(SEL)aSelector`
```
// User.m
// 消息转发重定向——决定由谁来执行(转发)
- (id)forwardingTargetForSelector:(SEL)aSelector{
    NSString * selStr = NSStringFromSelector(aSelector);
    if ([selStr isEqualToString:@"fly"]) {
        NSLog(@"forwardingTargetForSelector: %@",selStr);
        Plane * plane = [Plane new];
        return plane;
    }
    // 继续向下面转发
    return [super forwardingTargetForSelector:aSelector];
}
// Plane.m
- (void)fly{
    NSLog(@"%s  I'm flying",__func__);
}
```
方案三：`methodSignatureForSelector:(SEL)aSelector`、`forwardInvocation:(NSInvocation *)anInvocation`
```
// User.m
/*
    经过前两步，依然没法处理消息；下面做最后的尝试，消息处理越靠后，就表示处理消息的成本越大，性能的开销就越大。
 */
// methodSignatureForSelector用来生成方法签名，这个签名就是给forwardInvocation中参数NSInvocation调用。
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    NSString * selStr = NSStringFromSelector(aSelector);
    if ([selStr isEqualToString:@"drive"]) {
        NSMethodSignature * signature = [NSMethodSignature signatureWithObjCTypes:"v@:@"];
        return signature;
    }
    return [super methodSignatureForSelector:aSelector];
}
- (void)forwardInvocation:(NSInvocation *)anInvocation{
    SEL sel = @selector(flyToWhere:);
    NSMethodSignature * signature = [NSMethodSignature signatureWithObjCTypes:"v@:@"];
    anInvocation = [NSInvocation invocationWithMethodSignature:signature];
    Plane * plane = [Plane new];
    [anInvocation setTarget:plane];
    [anInvocation setSelector:sel];
    NSString * where = @"上海";
    // 消息的第一个参数是self，第二个参数是选择子，所以“上海”是第三个参数
    [anInvocation setArgument:&where atIndex:2];
    if ([plane respondsToSelector:sel]) {
        [anInvocation invokeWithTarget:plane];
    }else{
        [super forwardInvocation:anInvocation];
    }
}
// Plane.m
- (void)flyToWhere:(NSString *)place{
    NSLog(@"%s  I'm flying to %@",__func__,place);
}
```
最后一步： `doesNotRecognizeSelector:(SEL)aSelector`
```
// 最后抛出异常
- (void)doesNotRecognizeSelector:(SEL)aSelector{
    NSLog(@"方法 ---%@---- 存在",NSStringFromSelector(aSelector));
}
```
如下消息转发机制：
![](/images/runtime_04.jpeg)
- **4.给分类添加属性**
>1. 原理：给一个类声明属性，其实本质就是这个类添加关联，而不是直接在这个值的内存空间添加内存空间。

没有实现对应的方法，会报错：`Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIImage setNamed:]: unrecognized selector sent to instance 0x6000000ad380'`
```
// UIImage+Image.h
#import <UIKit/UIKit.h>

@interface UIImage (Image)
@property (nonatomic, copy) NSString * named;
@end
// UIImage+Image.m
#import "UIImage+Image.h"
#import <objc/runtime.h>
static const char *key = "sex";
@implementation UIImage (Image)

- (NSString *)named{
    return objc_getAssociatedObject(self, key);
}
- (void)setNamed:(NSString *)named{
    objc_setAssociatedObject(self, key, named, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```
- **5.字典转模型**
