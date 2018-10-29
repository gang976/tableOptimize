# iOS编码规范
## 一.核心原则
### 原则一：代码应该简洁易懂，逻辑清晰
因为软件是需要人来维护的。这个人在未来很可能不是你。所以首先是为人编写程序，其次才是计算机：

* 不要过分追求技巧，降低程序的可读性。
* 简洁的代码可以让bug无处藏身。要写出明显没有bug的代码，而不是没有明显bug的代码。

### 原则二：面向变化编程，而不是面向需求编程。
>需求是暂时的，只有变化才是永恒的。

本次迭代不能仅仅为了当前的需求，写出扩展性强，易修改的程序才是负责任的做法，对自己负责，对公司负责。

### 原则三：先保证程序的正确性，防止过度工程
过度工程（over-engineering）：在正确可用的代码写出之前就过度地考虑扩展，重用的问题，使得工程过度复杂。

引用《王垠：编程的智慧》里的话：
>
先把眼前的问题解决掉，解决好，再考虑将来的扩展问题。
>
先写出可用的代码，反复推敲，再考虑是否需要重用的问题。
>
先写出可用，简单，明显没有bug的代码，再考虑测试的问题。

## 二.通用规范

### 关于大括号

* 控制语句(if,for,while,switch)中，大括号开始与行尾
* 函数中，大括号要开始于行首

	如下示例：
		
		//控制语句
		while(someCondition) {
 
		}
 
		//函数
		void function(param1,param2)
		{
 
		}
		

### 运算符
#### 1. 一元运算符与变量之间没有空格：
	!bValue
	~iValue
	++iCount
	*strSource
	&fSum
	
#### 2. 二元运算符与变量之间必须有空格
	fWidth = 5 + 5;
	fLength = fWidth * 2;
	fHeight = fWidth + fLength;
	
	for(int i = 0; i < 10; i++) {
		//do something
	}
	
### 变量

1. 一个变量有且只有一个功能，尽量不要把一个变量用作多种用途；

2. 变量在使用前应初始化，防止未初始化的变量被引用；

3. 局部变量应该尽量接近使用它的地方；

推荐这样写：

	void func someFunction() 
	{
  		int index = ...;
  		//Do something With index
 
  		...
  		...
 
  		int count = ...;
  		//Do something With count
	}

不推荐这样写：

	void func someFunction() 
	{
  		int index = ...;
  		int count = ...;
  		//Do something With index
 
  		...
  		...
 
  		//Do something With count
	}
	
### if语句
#### 1. 不要使用过多的分支，要善于使用return来提前返回错误的情况

推荐这样写：

	- (void)someMethod 
	{ 
  		if (!goodCondition) {
    		return;
  		}
  		//Do something
	}

不推荐这样写：

	- (void)someMethod 
	{ 
  		if (goodCondition) {
    		//Do something
  		}
	}

比较典型的例子：

	-(id)initWithDictionary:(NSDictionary*)dict error:(NSError)err
	{
   		//方法1. 参数为nil
   		if (!dict) {
     		if (err) *err = [JSONModelError errorInputIsNil];
     		return nil;
    	}
 
    	//方法2. 参数不是nil，但也不是字典
    	if (![dict isKindOfClass:[NSDictionary class]]) {
        	if (err) *err = [JSONModelError errorInvalidDataWithMessage:@"Attempt to initialize JSONModel object using initWithDictionary:error: but the dictionary parameter was not an 'NSDictionary'."];
        	return nil;
    	}
 
    	//方法3. 初始化
    	self = [self init];
    	if (!self) {
        	//初始化失败
        	if (err) *err = [JSONModelError errorModelIsInvalid];
        	return nil;
    	}
 
    	//方法4. 检查用户定义的模型里的属性集合是否大于传入的字典里的key集合（如果大于，则返回NO）
    	if (![self __doesDictionary:dict matchModelWithKeyMapper:self.__keyMapper error:err]) {
        	return nil;
    	}
 
    	//方法5. 核心方法：字典的key与模型的属性的映射
    	if (![self __importDictionary:dict withKeyMapper:self.__keyMapper validation:YES error:err]) {
      		return nil;
    	}
 
    	//方法6. 可以重写[self validate:err]方法并返回NO，让用户自定义错误并阻拦model的返回
    	if (![self validate:err]) {
        	return nil;
    	}
 
    	//方法7. 终于通过了！成功返回model
    	return self;
	}
	
#### 2. 条件语句的判断应该是变量在左，常量在右

推荐这样写：

	if ( count == 6) {
	}

或者

	if ( object == nil) {
	}

或者

	if ( !object ) {
	}

不推荐这样写：

	if ( 6 == count) {
	}

或者

	if ( nil == object ) {
	}
	
#### 3. 每个分支的实现代码都必须被大括号包围
推荐这样写：

	if (!error) {
  		return success;
	}

不推荐这样写：

	if (!error)
    	return success;
    	
或者

	if (!error) return success;
	
#### 4. 条件过多，过长的时候应该换行

推荐这样写：

	if (condition1() && 
    	condition2() && 
    	condition3() && 
    	condition4()) {
  			// Do something
	}
	
不推荐这样写：

	if (condition1() && condition2() && condition3() && condition4()) {
  		// Do something
	}
	
### 函数
#### 1. 一个函数只做一件事（单一原则）

每个函数的职责都应该划分的很明确（就像类一样）。

推荐这样写：

	dataConfiguration();
	viewConfiguration();

不推荐这样写：

	void dataConfiguration()
	{   
   		...
   		viewConfiguration();
	}

#### 2. 对输入参数的正确性和有效性进行检查，参数错误立即返回

推荐这样写：

	void function(param1,param2)
	{
      	if(param1 is unavailable){
           return;
      	}
 
      	if(param2 is unavailable){
           return;
      	}
 
     	//Do some right thing
	}
	
#### 3. 如果在不同的函数内部有相同的功能，应该把相同的功能抽取出来单独作为另一个函数

原来的调用：

	void logic() 
	{
  		a();
  		b();
  		if (logic1 condition) {
    		c();
  		} else {
    		d();
  		}
	}
将a，b函数抽取出来作为单独的函数

	void basicConfig() 
	{
  		a();
  		b();
	}
 
	void logic1() 
	{
 		basicConfig();
  		c();
	}
 
	void logic2() 
	{
 		basicConfig();
  		d();
	}

## iOS代码规范
### 注释

优秀的代码大部分是可以自描述的，我们完全可以用程代码本身来表达它到底在干什么，而不需要注释的辅助。

但并不是说一定不能写注释，有以下三种情况比较适合写注释：

1. 公共接口（注释要告诉阅读代码的人，当前类能实现什么功能）。

2. 涉及到比较深层专业知识的代码（注释要体现出实现原理和思想）。

3. 容易产生歧义的代码（但是严格来说，容易让人产生歧义的代码是不允许存在的）。

除了上述这三种情况，如果别人只能依靠注释才能读懂你的代码的时候，就要反思代码出现了什么问题。

最后，对于注释的内容，相对于“做了什么”，更应该说明“为什么这么做”。

###常量

#### 1. 常量以相关类名作为前缀

推荐这样写：

	static const NSTimeInterval ViewControllerFadeOutAnimationDuration = 0.4f;
	
不推荐这样写：

	static const NSTimeInterval fadeOutTime = 0.4;

#### 2. 对外公开某个常量：

如果我们需要发送通知，那么就需要在不同的地方拿到通知的“频道”字符串（通知的名称），那么显然这个字符串是不能被轻易更改，而且可以在不同的地方获取。这个时候就需要定义一个外界可见的字符串常量。

推荐这样写：

	//.h
	extern NSString *const CacheControllerDidClearCacheNotification;

	//.m
	static NSString * const CacheControllerDidClearCacheNotification = @"CacheControllerDidClearCacheNotification";

不推荐这样写：

	#define CompanyName @"Apple Inc." 
	#define magicNumber 42
	
###宏

#### 1. 宏、常量名都要使用大写字母，用下划线‘_’分割单词。

	#define URL_GAIN_QUOTE_LIST @"/v1/quote/list"
	#define URL_UPDATE_QUOTE_LIST @"/v1/quote/update"
	#define URL_LOGIN  @"/v1/user/login”
		
#### 2. 宏定义中如果包含表达式或变量，表达式和变量必须用小括号括起来。

	#define MY_MIN(A, B)  ((A)>(B)?(B):(A))

### CGRect函数

其实iOS内部已经提供了相应的获取CGRect各个部分的函数了，它们的可读性比较高，而且简短，推荐使用：

推荐这样写：

	CGRect frame = self.view.frame; 
	CGFloat x = CGRectGetMinX(frame); 
	CGFloat y = CGRectGetMinY(frame); 
	CGFloat width = CGRectGetWidth(frame); 
	CGFloat height = CGRectGetHeight(frame); 

而不是

	CGRect frame = self.view.frame;  
	CGFloat x = frame.origin.x;  
	CGFloat y = frame.origin.y;  
	CGFloat width = frame.size.width;  
	CGFloat height = frame.size.height;  

### 范型

建议在定义NSArray和NSDictionary时使用泛型，可以保证程序的安全性：

	NSArray<NSString *> *testArr = @[@"Hello", @"world"];
	NSDictionary<NSString *, NSNumber *> *dic = @{@"key":@(1), @"age":@(10)};
	
###属性

#### 1. 属性的关键字推荐按照 原子性，读写，内存管理的顺序排列

推荐这样写：

	@property (nonatomic, readwrite, copy) NSString *name;
	@property (nonatomic, readonly, copy) NSString *gender;
	@property (nonatomic, readwrite, strong) UIView *headerView;

#### 2. Block属性应该使用copy关键字

推荐这样写：

	typedef void (^ErrorCodeBlock) (id errorCode,NSString *message);
	
	//将block拷贝到堆中
	@property (nonatomic, copy) ErrorCodeBlock errorBlock;

#### 3. 形容词性的BOOL属性的getter应该加上is前缀

推荐这样写：

	@property (assign, getter=isEditable) BOOL editable;

#### 4. 使用getter方法做懒加载

实例化一个对象是需要耗费资源的，如果这个对象里的某个属性的实例化要调用很多配置和计算，就需要懒加载它，在使用它的前一刻对它进行实例化：

	- (NSDateFormatter *)dateFormatter 
	{
    	if (!_dateFormatter) {
           	_dateFormatter = [[NSDateFormatter alloc] init];
           	[_dateFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
    	} 
    	return _dateFormatter;
	}
	
但是也有对这种做法的争议：getter方法可能会产生某些副作用，例如如果它修改了全局变量，可能会产生难以排查的错误。

#### 5. 不要滥用点语法，要区分好方法调用和属性访问

推荐这样写：

	view.backgroundColor = [UIColor orangeColor]; 
	[UIApplication sharedApplication].delegate;
	
不推荐这样写：

	[view setBackgroundColor:[UIColor orangeColor]]; 
	UIApplication.sharedApplication.delegate;

### 方法

#### 1. 方法名中不应使用and，而且签名要与对应的参数名保持高度一致

推荐这样写：

	- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height;

不推荐这样写：

	- (instancetype)initWithWidth:(CGFloat)width andHeight:(CGFloat)height;
	- (instancetype)initWith:(int)width and:(int)height;

#### 2. 方法实现时，如果参数过长，则令每个参数占用一行，以冒号对齐。

	- (void)doSomethingWith:(NSString *)theFoo
                   	   rect:(CGRect)theRect
                   interval:(CGFloat)theInterval
	{
   		//Implementation
	}
	
#### 3. 私有方法应该在实现文件中申明。
	
	//.m
	@interface ViewController ()
	- (void)basicConfiguration;
	@end
 
	@implementation ViewController
	- (void)basicConfiguration
	{
   		//Do some basic configuration
	}
	@end

#### 4. 方法名用小写字母开头的单词组合而成

	- (NSString *)descriptionWithLocale:(id)locale;

#### 5. 方法名前缀

1. 刷新视图的方法名要以refresh为首。

2. 更新数据的方法名要以update为首。

推荐这样写：

	- (void)refreshHeaderViewWithCount:(NSUInteger)count;
	- (void)updateDataSourceWithViewModel:(ViewModel*)viewModel;

### 类
#### 1. 所有返回类对象和实例对象的方法都应该使用instancetype

将instancetype关键字作为返回值的时候，可以让编译器进行类型检查，同时适用于子类的检查，这样就保证了返回类型的正确性（一定为当前的类对象或实例对象）

推荐这样写：

	@interface ZOCPerson
	+ (instancetype)personWithName:(NSString *)name; 
	@end
不推荐这样写：

	@interface ZOCPerson
	+ (id)personWithName:(NSString *)name; 
	@end

#### 2. 在类的头文件中尽量少引用其他头文件

有时，类A需要将类B的实例变量作为它公共API的属性。这个时候，我们不应该引入类B的头文件，而应该使用向前声明（forward declaring）使用class关键字，并且在A的实现文件引用B的头文件。

	// EOCPerson.h
	#import
 
	@class EOCEmployer;
 
	@interface EOCPerson : NSObject
 
	@property (nonatomic, copy) NSString *firstName;
	@property (nonatomic, copy) NSString *lastName;
	@property (nonatomic, strong) EOCEmployer *employer;
 
	@end
 
	// EOCPerson.m
	#import "EOCEmployer.h"
这样做有什么优点呢：

1. 不在A的头文件中引入B的头文件，就不会一并引入B的全部内容，这样就减少了编译时间。

2. 可以避免循环引用：因为如果两个类在自己的头文件中都引入了对方的头文件，那么就会导致其中一个类无法被正确编译。

但是个别的时候，必须在头文件中引入其他类的头文件:

1. 该类继承于某个类，则应该引入父类的头文件。

2. 该类遵从某个协议，则应该引入该协议的头文件。而且最好将协议单独放在一个头文件中。

#### 3. 类的布局

	#pragma mark - Life Cycle Methods
	- (instancetype)init
	- (void)dealloc
 
	- (void)viewWillAppear:(BOOL)animated
	- (void)viewDidAppear:(BOOL)animated
	- (void)viewWillDisappear:(BOOL)animated
	- (void)viewDidDisappear:(BOOL)animated
 
	#pragma mark - Override Methods
 
	#pragma mark - Intial Methods
 
	#pragma mark - Network Methods
 
	#pragma mark - Target Methods
 
	#pragma mark - Public Methods
 
	#pragma mark - Private Methods
 
	#pragma mark - UITableViewDataSource  
	#pragma mark - UITableViewDelegate  
 
	#pragma mark - Lazy Loads
 
	#pragma mark - NSCopying  
 
	#pragma mark - NSObject  Methods
	
### 其他
#### 1. Xcode工程文件的物理路径要和逻辑路径保持一致。

#### 2. 忽略没有使用变量的编译警告

对于某些暂时不用，以后可能用到的临时变量，为了避免警告，我们可以使用如下方法将这个警告消除：

	- (NSInteger)giveMeFive 
	{ 
 		NSString *foo; 
 		#pragma unused (foo) 
 		return 5; 
	}

#### 3. 手动标明警告和错误

手动明确一个错误：

	- (NSInteger)divide:(NSInteger)dividend by:(NSInteger)divisor 
	{ 
 		#error Whoa, buddy, you need to check for zero here! 
 		return (dividend / divisor); 
	}

手动明确一个警告：

	- (float)divide:(float)dividend by:(float)divisor 
	{ 
 		#warning Dude, don't compare floating point numbers like this! 
     	if (divisor != 0.0) { 
        	return (dividend / divisor); 
     	} else {  
        	return NAN; 
 		} 
	}