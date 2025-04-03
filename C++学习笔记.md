# C++学习笔记

## c++指针

有关指针，那么先讲一下电脑的内存，把内存理解为一条有限长度的街道，街道里有很多房子，房子都是有门牌号的。这条街上住着很多人，但是人分为几种大的类型，不同类型的人住的房子是不一样大的。但因为人的基因不同，长的不一样。那么房子就是内存，内存有门牌号，房子里面住着人，也就是内存是用来存放的就是数据，数据有不同的数据结构，对应几种大的类型的人，不同的数据类型申请的内存大小不一样，也就是不同阶层的人申请住的房子的大小不一样。最后，各色各样的人就是对应各种数据。对于cpu来说，每一个数据可以是是1，2，4个字节，那么这些字节存放在不同的内存里，内存是有编号的。对于不同数据类型的数据，他申请到的内存大小也是不一样的，比如int是4个字节，char是一个字节。那么如何去找到对应的人(数据)呢，就需要门牌号(内存地址),那么指针就是这串门牌号，他仅仅是一个整数，这个证书就是门牌号，当然他也是数据，也是住在房子里的。

接着解释一下下面的代码，首先申请一个8个字节char类型的数组，然后他会返回这个数组头部的内存地址，那么我创建一个char类型的指针去接收这个内存地址，那么我们得到了这个数组的门牌号，接着我们将申请的这块8字节的数组内容全部赋值为0，也就是初始化。接着可以使用*解引用符号（就是访问内存地址中内容的符号），来得到或者改变数组的内容， `*buffer = 'A'`所示。那么什么是二维指针，就是指向一个指针的内存地址的指针，如果说一个指针代表一个数组，那么二维指针指向一个二维数组，如`char** ptr2 = &buffer;`所示，创建了一个二维指针，他指向的是指针buffer的内存地址，也就是这个指针就是指针buffer的门牌号，使用第一次 * 号，我们拿到的是buffer指针，第二次 * 号，拿到的就是之前创建数组的内容。最后使用完这块内存，使用delete[]函数删除掉这块数组。

```c++
#include <iostream>
int main(){
    char* buffer = new char[8];        // 申请一个8个字节char类型的数组
    memset(buffer,0,8);                // 将申请的这块8字节的数组内容全部赋值为0，也就是初始化
    *buffer = 'A';                     // 使用*解引用符号，来得到或者改变数组的内容
    char** ptr2 = &buffer;
    **(ptr2++) = 'B';
    ...
    delete[] buffer;
    std::cin.get();                     // 等待输入回车键
}
```

## C++引用

这个就是给变量创建了一个别名，他没有实际的内存，上述的指针是占用内存的，并且初始化可以为NULL，设置为空指针，但是这个引用创建的时候必须赋值，来声明它是谁的别名。

下面的代码，先声明一个Increment函数，用来将输入的int类型变量+1，那么如果只是单纯的给int，不使用指针（*）或者引用（&），那么该函数会新拷贝一个value，然后为它+1，而不是将输入的value变量+1，那么现在我使用引用，将参数变为输入变量的别名，也就是说这个参数和变量使用一个地址了，那么我再更改这个参数，变量也会跟着变化。有时使用函数接收参数的时候，想要只读某个变量，就在前面加入一个const符号，保证其不可被改写。

```c++
#include <iostream>
void Increment(int& value){
    value++;
}
int main(){
    int a = 5;
    int b = 8;
    Increment(a);
    int& ref = a;          // ref是a的别名
    ref = b;               // 到这里，ref不再是a的别名，而是b的别名
    std::cin.get();
}
```

## C++静态变量

有关静态变量，就是将一个变量限定在一个cpp或者h文件中，在链接器链接的时候，无视这个变量，那么这个变量只能在这个文件中读取和改变，同时这个变量只会初始化一次，多次初始化相同名字的静态变量会报错，如果这个变量是静态全局变量，没有什么大问题；但如果是静态局部变量，比如在一个函数中定义一个静态变量，那么他只会初始化一次，下一次进入的时候就不会被初始化。

## C++类静态变量

故名思意，也就是在类里面的静态变量或者静态函数，就是在前面加上static关键字。正如下面所示，相当于类在定义多个实例的时候，这些实例中有关这个静态变量，内容是共享的，每当有一个实例去改变这个量，那么所有实例有关这个量都改变，正如下面所示，在类中定义`static int x, y;`，之后需要声明一个全局变量，也就是在Entity类这个作用域下的x和y，这样就实现了静态变量，接着下面可以使用Entity::x或者y去改变和读取它。

```c++
#include <iostream>
struct Entity
{
    static int x, y;
    void Print(){         // 如果在void前面加入static，则是下面那种情况
        std::cout << x << "," << y << std:endl;
    }
}
int Entity::x;
int Entity::y;
int main(){
    Entity e;
    Entity::x = 2;             // 访问静态变量或者函数都得用::域解析符
    Entity::y = 3;
    Entity e1;
    Entity::x = 5;
    Entity::y = 8;
    e.Print();
    e1.Print();
    std::cin.get();
}
```

那么接下来的重要一点，如果在Print函数前面加入一个static，**那么成为静态函数，静态函数不能访问非静态变量。**而上面的例子告诉我们非静态函数可以访问静态变量。那为啥是这样呢，因为类在调用类的非静态函数时，会将类实例传递给这个非静态函数，那么他就可以使用这个类中的相关静态和非静态变量。如果函数变成了静态函数，那么相当于这个函数被定义在了类的外面，如果不传入类实例，当然会报错。区别如下面所示。

```c++
#include <iostream>
struct Entity
{
    static int x, y;
}、
// 静态函数等效于定义在了类之外，如果不传入实例，则会报错
void Print(){
std::cout << x << "," << y << std:endl;
}
// 静态函数等效于定义在了类之外，传入实例，则不会报错
void Print(Entity e){
    std::cout << e.x << "," << e.y << std:endl;
}
int main(){
    Entity e;
    std::cin.get();
}
```

## C++实现单例

什么是单例，就是这个类只有一个实例存在，有两种方法实现，后者更加简洁，代码如下

前者首先是创建一个类静态变量指针，也就是实例化当前这个类指针，接着需要在外部声明这个实例是一个类静态变量指针，接着创建一个静态方法，返回的是当前这个类类型的引用，那么这个方法里面是返回解引用的类指针，那么就是返回一个实例化的类，接着在类中创建一个自定义的函数，就可以调用这个方法，在main函数中可以看到是Singleton::Get().Hello();那么前面是::符号的原因是因为访问的是静态函数。**这样就创建了一个单例**，也就是我们使用get方法后再调用类函数，只会使用类静态变量指针实例化的那个类，而这个指针只会实例化一次。

后者更为简洁，具体看注释，原理和上面是一样的，也可以看Cherno的视频复习。

```c++
#include <iostream>
class Singleton
{
private:
    static Singleton* s_Instance;                     // 创建一个当前类的静态变量指针
public:
    static Singleton& Get(){ return *s_Instance;}     // 返回的不是指针了，而是类开头的内存地址
    void Hello(){}                                    // 类方法，get后调用
};
Singleton* Singelton::s_Instance = nullptr;           // 声明静态变量指针
int main(){
    Singleton::Get().Hello();
    std::cin.get();
}
```

```c++
#include <iostream>
class Singleton
{
public:
    static Singleton& Get()    // 返回的是一个实例类的别名，如果去掉这个&，下面可不加static
    {
        // 必须使用static，不使用的话运行到下面的括号后会自动销毁这个实例，释放内存
        // 加入之后，会演唱这个实例的周期至程序死亡
        static Singleton instance;      // 这里创建的是一个局部静态变量，所以不用声明
        return instance;
    }
    void Hello(){}
};
int main(){
    Singleton::Get().Hello();
    std::cin.get();
}
```

## C++枚举

**枚举量的声明**

- ENUM是enumeration的缩写。基本上它就是一个数值集合。不管怎么说，这里面的**数值只能是整数。**
- 定义枚举类型的主要目的：**增加程序的可读性**
- 枚举变的名字一般以**大写字母**开头（非必需）
- 默认情况下，编译器设置第一个 枚举变量值为 0，下一个为 1，以此类推（也可以手动给每个枚举量赋值），且 **未被初始化的枚举值的值默认将比其前面的枚举值大1。** ）
- 枚举量的值可以相同
- 枚举类型所使用的类型默认为int类型，也可指定其他类型 ，如 unsigned char

```cpp
enum example {   //声明example为新的数据类型，称为枚举(enumeration);
 Aa, Bb, Cc   //声明Aa, Bb, Cc等为符号常量，通常称之为枚举量，其值默认分别为0，1，2
};

enum example {
 Aa = 1, Bb, Cc = 1，Dd, Ee //1 2 1 2 3 未被初始化的枚举值的值默认将比其前面的枚举值大1。
};

enum example : unsigned char //将类型指定成unsigned char，枚举变量变成了8位整型，减少内存使用。
{
 Aa, Bb = 10, Cc
};

enum example : float  //ERROR！枚举量必须是一个整数，float不是整数（double也不行）。
{
 Aa, Bb = 10, Cc
};
```

**枚举应用**:

**枚举**和**switch**是最好的搭档：

```cpp
enum enumType{
    Step0, Step1, Step2
}Step=Step0;      //注意这里在声明枚举的时候直接定义了枚举变量Step,并初始化为Step0 
switch (Step)
{
case Step0:{…;break;}
case Step1:{…;break;}
case Step2:{…;break;}
default:break;
}
```

如果在类中定义枚举类型，要使用这个枚举类型，则要加上类的作用域

```cpp
class Log{
public:
    enum Level{
        LevelError = 0,
        LevelWarning,
        LevelInfo
    };
private:
    m_LogLevel = LevelInfo;
public:
    void SetLevel(Level level){
        m_LogLevel = level;
    }
};
int main(){
    Log log;
    log.SetLevel(Log::LevelError);         // 这里不要忘记域名解析符
    std::cin.get();
}
```

## C++构造函数

- 当创建对象的时候，构造函数被调用
- 构造函数最重要的作用就是初始化类，构造函数可以有多个定义，带参数的和不带参数的，根据实例化时候传入参数的类型和多少选择哪一个函数实例化的时候使用哪个构造函数。

```cpp
class Entity {
public:
  int X, Y;
  Entity(){}  //不带参数
  Entity(int x, int y) {X=x;Y=y}  //带参数，用来初始化x和y
  //Entity(int x, int y) : x(x), y(y) {}  上下两行意义相同

  void print()
  {
      Entity e;
      std::cout << e.X << ',' << e.Y << std::endl;
  }
};
```

- 构造函数没有返回类型
- **构造函数的命名必须和类名一样**
- 如果你不指定构造函数，你仍然有一个构造函数，这叫做默认构造函数（default constructor），是默认就有的。但是，我们仍然可以删除该默认构造函数：

```cpp
class Log{
public:
    Log() = delete;  //删除默认构造函数
    ......
}
```

- 构造函数不会在你没有实例化对象的时候运行，所以如果你只是使用类的静态方法，构造函数是不会执行的。
- 当你用new关键字创建对象实例的时候也会调用构造函数。

## C++继承

继承中有三个关键字：public private protected，它们相应地改变了基类成员的访问属性。

- 1.**public 继承：**基类 public 成员，protected 成员，private 成员的访问属性在派生类中分别变成：public, protected, private
- 2.**protected 继承：**基类 public 成员，protected 成员，private 成员的访问属性在派生类中分别变成：protected, protected, private
- 3.**private 继承：**基类 public 成员，protected 成员，private 成员的访问属性在派生类中分别变成：private, private, private

但无论哪种继承方式，下面两点都没有改变：

- <font color="red">1.private 成员只能被本类成员（类内）和友元访问，不能被派生类访问；</font>
- <font color="red">2.protected 成员可以被派生类访问。</font>

- 当你创建了一个子类，它会包含父类的一切。
- 继承给我们提供了这样的一种方式：把一系列类的所有通用的代码（功能）放到基类
- 在定义一个新的类 B 时，如果该类与某个已有的类 A 相似（指的是 B 拥有 A 的全部特点），那么就可以把 A 作为一个基类，而把B作为基类的一个派生类（也称子类）。
- 派生类是通过对基类进行修改和扩充得到的，在派生类中，可以扩充新的成员变量和成员函数。
- 派生类拥有基类的全部成员函数和成员变量，不论是private、protected、public。需要注意的是：在派生类的各个成员函数中，不能访问基类的private成员。

**继承的格式**

```cpp
class 派生类名：public 基类名
{
};
```

例子如下，分析：

- 这个Player类不再仅仅只是Player类型，它也是Entity类型，就是说它**同时是这两种类型**。<font color="red">意思是我们可以在**任何想要用Entity的地方使用Player**</font>
- Player总是Entity的一个超集，它拥有Entity的所有内容。
- 因为对于Player来说，在Entity中任何**不是私有**的（private）成员，Player都可以**访问**到

```cpp
//基类
class Entity
{
public:
    float x,y;
 void move(float xa, float ya)
 {
     x += xa;
     y += ya;
 }
};
//派生类
class Player : public Entity
{
public:
 const char* Name;
//  float x,y;    //因为是派生类，所以这些是重复代码，只保留新代码即可
//  void() move(float xa, float ya)
//  {
//      x += xa;
//     y += ya;
// }
 void printName()   //在派生类中，可以扩充新的成员变量和成员函数
 {
     std::cout << Name << std::endl;
 }
};
```

## C++虚函数

- 虚函数可以让我们在子类中重写方法。
- 格式

```cpp
claee 父类名{
   //virtual + 函数
   virtual void GetName(){
       .....
   }
}
```

- 虚函数的例子，通常有三步。

> 第一步，定义基类，声明基类函数为 `virtual` 的。
> 第二步，定义派生类(继承基类)，派生类实现了定义在基类的 `virtual` 函数。
> 第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类`virtual` 函数。

例子：

```cpp
//基类
class Entity
{
public:
    std::string GetName() {return "Entity";}
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {}  //构造函数
 std::string GetName() {return m_Name;}
};

int main(){
 Entity* e = new Entity();//不用手动删除，因为程序会终止（对象被自动删除）
 std::cout << e->GetName() << std::endl; 
 Player* p = new Player("cherno"); 
 std::cout << p->GetName() << std::endl;
}
---------------------------------
//输出
Entity
cherno
```

上述代码虽能够成功运行，**但是如果使用多态的概念来看，目前为止我们写的这些都是不合格的**

如果我引用这个Player并把它当成Entity类型，就会出现问题，例如：

```cpp
int main(){
 Entity* e = new Entity();//不用手动删除，因为程序会终止（对象被自动删除）
 std::cout << e->GetName() << std::endl; 
 Player* p = new Player("cherno"); 
 std::cout << p->GetName() << std::endl;
 //声明基类指针，并指向派生类
 Entity* entity = p; //p是一个Player类型的指针，它是一个Player，但是我把它指向了一个Entity
 std::cout << entity->GetName() << std::endl; 
}
```

输出：

```cpp
Entity
cherno
Entity //运行代码，你会看见打印出了"Entity"，但是我们希望的是打印Player
```

我们希望的是打印Player,因为虽然我们指向的是个Entity*，但是它实际上是一个Player，它是一个Player类的实例，但实际上运行代码，你会看见打印出了"Entity"，why?

```cpp
Entity* entity = p;
```

举一个更清晰的例子：

```cpp
//基类
class Entity
{
public:
    std::string GetName() {return "Entity";} 
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {}  //构造函数
 std::string GetName() {return m_Name;}
};

void printName(Entity* entity){
 std::cout << entity -> GetName() << std::endl;
}

int main(){
 Entity* e = new Entity();
 printName(e); //我们这儿做的就是调用entity的GetName函数，我们希望这个GetName作用于Entity
 Player* p = new Player("cherno"); 
 printName(p); //printName(Entity* entity)，没有报错是因为Player也是 Entity类型。同样我们希望这个GetName作用于Player
}
```

输出：

```cpp
Entity
Entity
```

两次输出都是Entity，**原因在于如果我们在类中正常声明函数或方法，当调用这个方法的时候，它总是会去调用属于这个类型的方法**，而`void printName(Entity* entity);`参数类型是`Entity*`,意味着它会调用Entity内部的GetName函数，**它只会在Entity的内部寻找和调用GetName**.

但是我们希望C++能意识到,在这里我们传入的其实是一个Player，所以请调用Player的GetName。**此时就需要使用虚函数了。**

- 虚函数引入了一种要**动态分配**的东西，一般通过虚表（vtable）来实现编译。虚表就是一个包含类中所有虚函数映射的列表，通过虚表我们就可以在运行时找到正确的被重写的函数。
- 简单来说，你需要知道的就是**如果你想重写一个函数，你么你必须要把基类中的原函数设置为虚函数**

```cpp
//基类
class Entity
{
public:
    virtual std::string GetName() {return "Entity";} //第一步，定义基类，声明基类函数为 virtual的。
};

//派生类
class Player : public Entity
{
private: 
    std::string m_Name; 
public: 
    Player(const std::string& name):m_Name (name) {} 
    //第二步，定义派生类(继承基类)，派生类实现了定义在基类的 virtual 函数。
    std::string GetName() override {return m_Name;}  //C++11新标准允许给被重写的函数用"override"关键字标记,增强代码可读性。
};

void printName(Entity* entity){
    std::cout << entity -> GetName() << std::endl;
}

int main(){
    Entity* e = new Entity();
    printName(e); 
    //第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类virtual函数。Entity* p = new Player("cherno");也可以
    Player* p = new Player("cherno"); 
    printName(p); 
}
```

## C++接口（纯虚函数）

- **纯虚函数优点**

> 防止派生类忘记实现虚函数，**纯虚函数使得派生类必须实现基类的虚函数**。
> 在某些场景下，创建基类对象是不合理的，含有纯虚拟函数的类称为**抽象类**，它**不能直接生成对象。**

- **声明方法**: 在基类中纯虚函数的方法的后面加 **=0**

```cpp
virtual void funtion()=0;
virtual std::string GetName() = 0;
```

- C++中的纯虚函数本质上与其他语言（bi如Java或C#）中的抽象方法或接口相同。
- 纯虚函数与虚函数的区别在于，纯虚函数的基类中的`virtual`函数，只定义了，但不实现。实现交给派生类来做。
- **只能实例化一个实现了所有纯虚函数的类**。**纯虚函数必须被实现**，然后我们才能创建这个类的实例。
- 纯虚函数允许我们在基类中定义一个没有实现的函数，然后**强制子类**去实现该函数。
- 实际上，其他语言有interface关键字而不是叫class，但C++没有。接口只是C++的类而已。
- 在面向对象程序设计中，创建一个只包含未实现方法然后交由子类去实际实现功能的类是非常普遍的,这通常被称为接口。**接口就是一个只包含未实现的方法并作为一个模板的类**。并且由于此**接口类**实际上不包含方法实现，所以我们**无法实例化**这个类。

例子：

```cpp
//基类
class Entity
{
public:
    virtual std::string GetName() = 0; //声明为纯虚函数。请记住，这仍然被定义为虚函数，但是=0实际上将它变成了一个纯虚函数，这意味着如果你想实例化这个类，那么这个函数必须在子类中实现
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {} 
 std::string GetName() override {return m_Name;} //实现纯虚函数
};

void printName(Entity* entity){
 std::cout << entity -> GetName() << std::endl;
}

int main(){
// Entity* e = new Entity();  //会报错，在这里我们必须给它一个实际上实现了这个函数的子类
 Entity* e = new Player("");  //ok
 printName(e); 

 Player* p = new Player("cherno"); 
 printName(p); 
}
```

> 再看一个更好的例子
> 假设我们想编写一个打印这儿所有类名的函数。

```cpp
class Printable{   //接口。其实就是个类。之所以称它为接口，只是因为它有一个纯虚函数，仅此而已。
public:
    virtual std::string GetClassName()= 0;
};
//基类
class Entity : public Printable
{
public:
    virtual std::string GetName() {return "Entity";}
 std::string GetClassName() override {return "Entity";} //实现接口的纯虚函数
};

//派生类
class Player : public Entity //因为Player已经是Entity（已有接口），所以Player不用去实现Printable接口
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {} 
 std::string GetName() override {return m_Name;} 
 std::string GetClassName() override {return "Player";} //实现接口的纯虚函数
};

void  Print(Printable* obj){  //我们需要某种类型能提供GetClassName函数，这就是接口。所有的打印都来自于这个Print函数，它接受printable对象，它不关心实际是什么类，这里并没有做Printable的实例化，所以并不会报错
    std::cout <<obj ->GetClassName() << std::endl;
}
int main(){
 Entity* e = new Entity();
 Player* p = new Player("cherno"); 
 Print(e);
 Print(p);
}
//输出：
Entity
Player
```

上例中，如果Player不是Entity的话，就要添加接口，如下

```cpp
class Player : public OtherClass,Printable  //加逗号，添加接口Printable
{
 ....
 std::string GetName() override {return m_Name;} 
};
```

## C++数组

- C++数组就是表示一堆的变量组成的集合，一般是一行相同类型的变量。
- 内存访问违规（Memory access violation）：在debug模式下，你会得到一个程序崩溃的错误消息，来帮助你调试那些问题；然而在release模式下，你可能不会得到报错信息。这意味着你已经写入了不属于你的内存。

```cpp
//定义一个含5个整数的数组  
    int a[5];
    //访问
    a[5],a[-1]; //内存访问违规（Memory access violation）
```

- 循环的时候涉及到性能问题，我们一般是小于比较而不是小于等于（少一次等于的判断）。

```cpp
int main()
{
    int example[5];
    int* ptr = example;
    for (int i = 0; i< 5;i++) //5个元素全部设置为2
        example[i] = 2; 

    example[2] = 5; //第三个元素设置为5
    *(ptr + 2) = 6; //第三个元素设置为6。因为它会根据数据类型来计算实际的字节数，所以在这里因为这个指针是整形指针所以会是加上2乘以4，因为每个整形是4字节
    *(int*)((char*)ptr + 8) = 7; //第三个元素设置为5。因为每个char只占一个字节
    std::cin.get();
}
```

### 栈数组和堆数组

- 不能把栈上分配的数组（字符串）作为返回值，**除非**你传入的参数是一个内存地址。
- 如果你想返回的是在函数内新创建的数组，那你就要用new关键字来创建。
- **<font color="red">栈数组`int example[5];` 堆数组`int* another = new int[5];`</font>**

```cpp
int main()
{
 int example[5]; //这个是创建在栈上的，它会在跳出这个作用域时被销毁
 for (int i = 0; i< 5;i++) //5个元素全部设置为2
    example[i] = 2; 
    int* another = new int[5];//这行代码和之前的是同一个意思，但是它们的生存期是不同的.因为这个是创建在堆上的,实际上它会一直存活到直到我们把它销毁或者程序结束。所以你需要用delete关键字来删除它。
 for (int i = 0; i< 5;i++) //5个元素全部设置为2
    another[i] = 2; 
 delete[] another;
 std::cin.get();
}
```

> 上述的两个数组在内存上看都是一样的，元素都是5个2；
> 那为什么要使用new关键字来动态分配，而不是在栈上创建它们呢？**最大的原因是因为生存期**,因为new分配的内存，会一直存在，直到你手动删除它。
> 如果你有个函数要**返回新创建的数组**，那么你**必须要使用new来分配**，**除非**你传入的参数是一个内存地址。

### memory indirection（内存间接寻址）

> 意思是，有个指针，指针指向另一个保存着我们实际数组的内存块（p-> p-> array），这会产生一些内存碎片和缓存丢失。

```cpp
class Entity
{
public:
    int example[5]; //栈数组
    Entity()  //创建一个构造函数，用来初始化所有值为2
    {
        for (int i = 0; i< 5;i++) 
             example[i] = 2;   
    }
};

int main()
{
    Entity e; //实例化一个对象。如果我们查看Entity e的内存地址，可以看到Entity的内存上实际就是一行，包含了数组中所有的2，所有的数据都在这儿

    std::cin.get();
}
```

> 但是如果我在这里使用new关键字在堆上创建，运行相同的代码
> 这时我们查看Entity e的内存地址，可以看到这里的内存根本没有2。我看到另一个内存地址，其实这就是个指针。我可以复制该地址然后粘贴查找（因为endian（字节序）的原因我必须要反转它们），然后就可以看到我真正的数据。这就是**memory indirection（内存间接寻址）**。

```cpp
class Entity
{
public:
    int* example = new int[5];  //堆数组
    Entity()  
    {
        for (int i = 0; i< 5;i++) 
             example[i] = 2;   
    }
};

int main()
{
    Entity e; //这时我们查看Entity e的内存地址，可以看到这里的内存根本没有2。我看到另一个内存地址，其实这就是个指针。我可以复制该地址然后粘贴查找（因为endian（字节序）的原因我必须要反转它们），然后就可以看到我真正的数据。这就是memory indirection（内存间接寻址）

    std::cin.get();
}
```

我们在e地址上实际上得到的另一个地址，这个地址指向我们实际的数组。这意味着如果我们要访问这个数组，我们基本上要在代码里跳来跳去，先找到Entity，然后再找到数组。

如果可以你应该在**栈**上**创建数组来避免这种情况**。因为像这样在内存里跳跃肯定会影响性能

### C++11中的std:array

> 这是一个内置数据结构，定义在C++11的标准库中。很多人喜欢用它来代替这里的原生数组，因为他有很多优点，它有边界检查，有记录数组的大小
> 实际上我们没有办法计算原生数组的大小，但可以通过一些办法知道大小（例如因为当你删除这个数组时，编译器要知道实际上需要释放多少内存）。

**计算原生数组大小**

> 方法一：通过依赖编译器，它有可能会在数组中存储一个负索引（-1），但应该永远都不要在数组内存中访问数组的大小，这很危险。

创建一个**栈数组**,你不知道他的实际大小，因为它是在栈上分配的，也就是说这是**栈上的地址加上偏移量**

```cpp
int a[5]; //栈数组
```

所以如果你写:

```cpp
sizeof(a); //20bytes
```

如果你想知道有多少个元素，可以用sizeof(a)除以数据类型int的大小，得到5

```cpp
int count = sizeof(a) / sizeof(int); //5
```

但是如果你用**堆数组**example做同样的事：

```cpp
int* example = new int[5];  //堆数组
int count = sizeof(example) / sizeof(int); //1
```

你再这里得到的实际上是一个整形指针的大小（int* example），就是4字节，4/4就是1。

所以**只能在栈分配的数组上用这个技巧**，但是你真的**不能相信这个方法**！当你把它放在函数或者它变成了指针，那你完蛋了（因为**“栈上的地址加上偏移量”**）。所以你要做的就是自己维护数组的大小。

如何维护呢？方法有两个

方法一：

```cpp
class Entity
{
public:
 static constexpr const int size = 5;//在栈中为数组申请内存时,它必须是一个编译时就需要知道的常量。constexpr可省略，但类中的常量表达式必须时静态的
 int example[size];  //此时为栈数组,
 Entity()  
 {
     for (int i = 0; i<size;i++) 
          example[i] = 2;   
 }
};
```

方法二：std:array

```cpp
include <array> //添加头文件
class Entity
{
public:
    std::array<int,5> another; //使用std::array
 Entity()  
 {
     for (int i = 0; i< another.size();i++) //调用another.size()
          example[i] = 2;   
 }
};
```

这个方法会安全一些。

## C++字符串

**字符串/字符数组的工作原理**

> 1.字符串实际上是字符数组
> 2.C++中有一种数据类型叫做char，是Character的缩写，这是一个字节的内存。它很有用因为它能把指针转换为char型指针，所以你可以用字节来做指针运算。它对于分配内存缓冲区也很有用，比如分配1024个char即1KB空间。它对字符串和文本也很有用，因为C++对待字符的默认方式是通过Ascii字符进行文本编码。我们在C++中处理字符是一个字符一个字节。Ascii可以扩展比如UTF-8、UTF-16、UTF-32，我们有wide string（宽字符串）等。我们有两个字节的字符、三个字节、四个字节的等等。
> **3.C++中默认的双引号就是一个字符数组const char\*，并且\**末尾会补’\0’ （空终止符）， 而cout会输出直到’\0’就终止。**

```cpp
const char* name = "cherno"; //c风格字符串
char* name = "cherno" //报错，因为C++中默认的双引号就是一个字符数组const char*
char name[3] = {'l','i','u'};//报错，缺少空终止符
char name[3] = {'l','i','u'，0};//正确
char name[3] = {'l','i','u'，'\0'};//正确,因为ascii码'\0'就是null
```

**C++字符串（std::string）的使用**

- C++标准库里有个类叫string，实际上还有一个模板类basic_string。**std::string** 本质上就是这个basic_string的char作为模板参数的**模板类实例**。叫**模板特化**（template specialization），就是把char作为模板类basic string的模板参数，意味着char就是每个字符背后的的数据类型。
- 在C++中使用字符串时你应该使用std::string。
- string有个接受参数为**char指针或者const char指针**的**构造函数**。在C++中用双引号来定义字符串一个或者多个单词时，它其实就是**const char数组**，而不是char数组。
- std::string本质上它就是一个char数组，一个char的数组和一些内置函数

使用：

```cpp
#include <iostream>
#include <string>

  int main()
  {
    std::string name = "Cherno";
    mname.size();
    std::cout << name << std::endl;

    std::cin.get();
  }
```

追加字符串

```cpp
  std::string name = "Cherno" + "hello!";//ERROR!
```

原因是在将两个const char数组相加，因为，**双引号里包含的内容是const char数组，它不是真正的string**；它不是字符串，你不能将两个指针或者两个数组加在一起，它不是这么工作的。

所以如果你想这么做，要么就是把它们分成多行，然后name+="hello！"

```cpp
  std::string name = "Cherno"";
  name += "hello!  //OK
```

这样做是在将一个指针加到了字符串name上了，然后+=这个操作符在string类中被重载了，所以可以支持这么操作。

或者经常做的是**显式地调用string构造函数**将**其中一个传入string构造函数中**,相当于你在创建一个字符串，然后附加这个给他。

```cpp
  std::string name = std::string("Cherno") + "hello!";//OK
  bool contains = name.find("no") != std::string::npos;//用find去判断是否包含字符“no”
```

## C++字符串字面量

- 字符串字面量就是双引号中的内容。
- 字符串字面量是存储在**内存**的**只读部分**的，不可对只读内存进行写操作。
- C++11以后，默认为`const char*`,否则会报错。

```cpp
char* name = "cherno";//Error!
  name[2] = 'a'; //ERROR! 未定义行为；是因为你实际上是在用一个指针指向那个字符串字面量的内存位置，
                 //但字符串字面量是存储在内存的只读部分的，而你正在试图对只读内存进行写操作
  --------------------------------------
  const char* name = "cherno"; //Ok!
  name[2] = 'a'; //ERROR!const不可修改
  //如果你真的想要修改这个字符串，你只需要把类型定义为一个数组而不是指针
  char name[] = "cherno"; //Ok!
  name[2] = 'a'; //ok
```

- 从C++11开始，有些编译器比如Clang，实际上只允许你编译`const char*`, 如果你想从一个字符串字面量编译char,你必须手动将他转换成`char*`

```cpp
char* name = (char*)"cherno"; //Ok!
name[2] = 'a'; //OK
```

- 别的一些字符串

> 基本上，char是一个字节的字符，char16_t是两个字节的16个比特的字符（utf16），char32_t是32比特4字节的字符（utf32），const char就是utf8. 那么wchar_t也是两个字节，和char16_t的区别是什么呢？事实上宽字符的大小，实际上是由编译器决定的，可能是一个字节也可能是两个字节也可能是4个字节，实际应用中通常不是2个就是4个（Windows是2个字节，Linux是4个字节），所以这是一个变动的值。如果要两个字节就用char16_t，它总是16个比特的。

```cpp
   const char* name = "lk";
   const wchar_t* name2 = L"lk";
   const char16_t* name3 = u"lk";
   const char32_t* name4 = U"lk";
   const char* name5 = u8"lk";
```

**string_literals**

```cpp
#include <iostream>
#include <string>

int main()
{
    using namespace std::string_literals;

    std::string name0 = "hbh"s + " hello";

    std::cin.get();
}
```

string_literals中定义了很多方便的东西，这里字符串字面量末尾加s，可以看到实际上是一个操作符函数，它返回标准字符串对象（std::string）

然后我们就还能方便地这样写等等：

```cpp
std::wstring name0 = L"hbh"s + L" hello";
```

string_literals也可以忽略转义字符

```cpp
#include <iostream>
#include <string>

int main()
{
    using namespace std::string_literals;

    const char* example =R"(line1
    line2
    line3
    line4)"

    std::cin.get();
}
```

## C++中CONST

- **const首先作用于左边的东西；如果左边没东西，就做用于右边的东西**
- const被cherno称为伪关键字，因为它在改变生成代码方面做不了什么。
- const是一个承诺，承诺一些东西是不变的，你是否遵守诺言取决于你自己。我们要保持const是因为这个承诺实际上可以简化很多代码。
- 绕开const的方法：**(但不建议这么做)**

> **强制转换**

```cpp
  const int Age =90;
  int* a = new int;
  *a = 2;
  a = &Age //error!
  a =(int*)&Age //ok
```



### **const指针的用法**

> 适用于指针和指针指向的内容

```cpp
int* a = new int;
*a = 2;
a =(int*)&Age //ok
```

上述代码我们可以做两件事，一可以**改变这个指针的内容**，就是指针指向的内存内容；二可以**改变指针指向的内存地址**。

> **const int*（同int const\*）**
> **可以改变指针指向的地址,不能再去修改指针指向的内容了**

```cpp
const int* a = new int;
*a = 2; //error! 不能再去修改指针指向的内容了。
a =(int*)&Age  //可以改变指针指向的地址
```

> int* const
> **可以改变指针指向的内容,不能再去修改指针指向的地址**

```text
int* const a = new int;
*a = 2; //ok
a =(int*)&Age  //error
```

> const int* const
> **既不可以改变指针指向的内容,也不能再去修改指针指向的地址**

### **在类和方法中的const**

> const的第三种用法，他和变量没有关系，而是用在方法名的后面( **只有类才有这样的写法** )
> 这意味这这个方法不会修改任何实际的类，因此这里你可以看到我们不能修改类的成员变量

```cpp
class Entity
{
private:
    int m_x,m_y;
public:
    int Getx() const  //const的第三种用法，他和变量没有关系，而是用在方法名的后面
    {
        return m_x; //不能修改类的成员变量
        m_x = 2; //ERROR!
    }
    void Setx(int a)
    {
        m_x = a; //ok
    }
};

void PrintEntity(const Entity& e)  //const 类只能调用类中的const函数，不能调用非const函数
{
    std::cout << e.Getx() << std::endl;
}

int main()
{
    Entity e;
}
```

然后有时我们就会写两个Getx版本，一个有const一个没有，然后上面面这个传const Enity&的方法就会调用const的GetX版本。

<font color="red">所以，我们把成员方法标记为const是因为**如果我们真的有一些const Entity对象，我们可以调用const方法**。如果没有const方法，那const Entity&对象就掉用不了该方法。</font>

- 如果实际上没有修改类或者它们不应该修改类，**总是**标记你的方法为const，否则在有常量引用或类似的情况下就用不了你的方法。
- 在**const函数中**， 如果要修改别的变量，可以用**关键字mutable**：

把类成员标记为mutable，意味着类中的const方法可以修改这个成员。

```cpp
class Entity
  {
  private:
      int m_x,m_y;
      mutable int var;         // mutable关键字可以使得这个变量在类中的const函数中被更改 
  public:
      int Getx() const 
      {   
          var = 2; //ok mutable var
          return m_x; //不能修改类的成员变量
          m_x = 2; //ERROR!
      }
  };
```

## C++mutable关键字

mutable有两种不同的用途：

> 1.与const一起用(最主要的用法，见上一篇)
> 2.lambda表达式，或者同时包含这两种情况

```cpp
//引用传递，没什么问题
#include <iostream>
int main()
{
    int x = 8;  
    auto f = [&]()
    {
        x++;  //如果时值传递，则会报错。
        std::cout << y << std::endl;
    };
    f();
}
-----------------------------------------------
//值传递的正确做法
#include <iostream>
int main()
{
    int x = 8;
    auto f = [=]()
    {
        int y = x;  //有些臃肿
        y++;
        std::cout << y << std::endl;
    };
    f();
}
```

添加mutable关键字,会干净许多（但本质是一样的）

```cpp
#include <iostream>
int main()
{
    int x = 8;
    auto f = [=]() mutable
    {
        x++;
        std::cout << x << std::endl;
    };
    f();
}
```

## C++的成员初始化列表

- 注意：在成员初始化列表里需要按成员变量定义的顺序写。这很重要，因为不管你怎么写初始化列表，它都会按照定义类的顺序进行初始化。
- 使用成员初始化列表的原因：

> 代码风格简洁 避免性能浪费。<font color="red">原因是如方法一中的代码，m_name在创建类的时候会被初始化一次，接着在执行构造函数的时候，对其进行了赋值，看上去像是赋值，实际上是第二次初始化，并且舍弃了第一次的初始化的值，那么这样就会造成性能浪费，为了解决这个问题，尽可能的去使用第二种成员初始化列表的方法，因为使用使用成员初始化列表的方法会自动检查类中是否有进行重复的初始化，有的话那么别的地方的声明则不作数，只在这个成员初始化列表的地方生效。</font>

有两种方法可以**在构造函数中初始化类成员**

方法一：

```cpp
class Entity
  {
  private:
    std::string m_Name;
  public:
    Entity()   //默认构造函数
    {    
        m_Name = "Unknow";
    }
    Entity(const std::string& name)
    {
        m_Name = name;
    }
  };
```

方法二：初始化成员列表

```cpp
  #include <iostream>
  
  class Entity
  {
  private:
  	std::string m_Name;
  	int m_Score;
  public:
  	Entity() : m_Name("Unknow"), m_Score(0)
  	{
  	}
  	Entity(const std::string& name,int n) :m_Name(name), m_Score(100)
  	{
  	}
  	const std::string& GetName() const { return m_Name; };
  	const int& GetScore() const { return m_Score; };
  };
  int main()
  {
  	Entity e0;
  	Entity e1("lk",50);
  	std::cout << e0.GetName() <<e0.GetScore() << std::endl;
  	std::cout << e1.GetName() <<e1.GetScore()<<std::endl;
  }
```

## C++三元运算符

格式:

```cpp
条件表达式 ? 表达式1 : 表达式2;
语义：如果“条件表达式”为true，则整个表达式的值就是表达式1，忽略表达式2；如果“条件表达式”为false，则整个表达式的值就是表达式2,等价于if/else语句。
```

> 实际上只是if的语法糖。

作用: - 代码更简洁 - 速度更快一点

- <font color="red">尽量不对三元操作符进行嵌套</font>

## 创建并初始化C++对象

- 基本上，当我们编写了一个类并且到了我们实际开始使用该类的时候，就需要实例化它(除非它是完全静态的类)
- 实例化类有两种选择，这两种选择的区别是内存来自哪里，我们的对象实际上会创建在哪里。
- 应用程序会把内存分为两个主要部分：堆和栈。还有其他部分，比如源代码部分，此时它是机器码。

### 栈分配

格式:

```cpp
// 栈中创建
Entity entity;
Entity entity("lk");
```

- 什么时候栈分配？几乎任何时候，因为在C++中这是初始化对象最快的方式和最受管控的方式。
- 什么时候不栈分配？ 如果创建的**对象太大**，或是需要显示地控制对象的**生存期**，那就需要堆上创建 。

### 堆分配

格式:

```cpp
// 堆中创建
Entity* entity = new Entity("lk");
delete entity； //清除
```

- 当我们调用new Entity时，实际发生的就是我们在堆上分配了内存，我们调用了构造函数，然后这个new Entity实际上会返回一个Entity指针，它返回了这个entity在堆上被分配的内存地址，这就是为什么我们要声明成Entity*类型。
- 如果你使用了new关键字，那你就要用delete来进行清理。

## C++ new关键字

- new的主要目的是分配内存，具体来说就是在堆上分配内存。
- 如果你用new和**[]**来分配数组，那么也用**delete[]**。
- new主要就是找到一个满足我们需求的足够大的内存块，然后**返回一个指向那个内存地址的指针**。

```cpp
int* a = new int; //这就是一个在堆上分配的4字节的整数,这个a存储的就是他的内存地址.
  int* b = new int[50];//在堆上需要200字节的内存。
  delete a;
  delete[] b;
  //在堆上分配Entity类
  Entity* e = new Entity();
  Entity* e = new Entity;//或者这我们不需要使用括号，因为他有默认构造函数。
  Entity* e0 = new Entity[50]; //如果我们想要一个Entity数组，我们可以这样加上方括号,在这个数组里，你会在内存中得到50个连续的Entity
  delete e;
  delete[] e0;
```

- 在new类时，该关键字做了两件事

> 分配内存 调用构造函数

```cpp
Entity* e = new Entity();//1.分配内存 2.调用构造函数
Entity* e = (Entity*)malloc(sizeof(Entity);//仅仅只是分配内存**然后给我们一个指向那个内存的指针
//这两行代码之间仅有的区别就是第一行代码new调用了Entity的构造函数
delete e;//new了，必须要手动清除
```

- new 是一个**操作符**，就像加、减、等于一样。它是一个操作符，这意味着你可以重载这个操作符，并改变它的行为。
- 通常调用new会调用隐藏在里面的C函数malloc，但是**malloc仅仅只是分配内存**然后给我们一个指向那个内存的指针，而**new不但分配内存，还会调用构造函数**。同样，**delete则会调用destructor析构函数。**
- new支持一种叫**placement new**的用法，这决定了他的内存来自哪里, 所以你并没有真正的分配内存。在这种情况下，你只需要调用构造函数，并在一个特定的内存地址中初始化你的Entity，可以通过些new()然后指定内存地址，例如：

```cpp
int* b = new int[50]; 
Entity* entity = new(b) Entity();
```

## C++隐式转换与explicit关键字

**隐式转换**

> 隐式转换**只能进行一次**。

```cpp
#include <iostream>

class Entity
{
private:
    std::string m_Name;
    int m_Age;
public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    Entity(int age)
        : m_Name("Unknown"), m_Age(age) {}
};

int main()
{
    Entity test1("lk");
    Entity test2(23); 
    Entity test3 = "lk"; //error!只能进行一次隐式转换,这里后面是const char*，不是std::string，不能进行两次转换（一次到string，一次到类的构造函数）
    Entity test4 = std::string("lk");
    Entity test5 = 23; //发生隐式转换


    std::cin.get();
}
```

如上，在test5中，int型的23就被隐式转换为一个Entity对象，这是**因为Entity类中有一个Entity(int age)构造函数，因此可以调用这个构造函数，然后把23作为他的唯一参数，就可以创建一个Entity对象。**

同时我们也能看到，对于语句`Entity test3 = "lk";`会报错，原因是**只能进行一次隐式转换**，<font color='red'>`"lk"`是`const char`数组，这里需要先转换为`std::string`，再从string转换为Entity变量，两次隐式转换是不行的，所以会报错。但是写为`Entity test4 = std::string("lk");`就可以进行隐式转换。</font>

最好不写`Entity test5 = 23;`这样的函数，应尽量避免隐式转换。因为`Entity test2(23);`更清晰。



**explicit 关键字**

- explicit是用来当你想要显示地调用构造函数，而不是让C++编译器隐式地把任何整形转换成Entity
- 我有时会在数学运算库的地方用到explicit，因为我不想把数字和向量来比较。一般explicit很少用到。
- 如果你在构造函数前面加上explicit，这意味着这个构造函数不会进行隐式转换
- 如果你想用一个整数构造一个Entity对象，那你就必须显示的调用这个构造函数，**explicit会禁用隐式转换**，explicit关键字放在构造函数前面

```cpp
#include <iostream>
  class Entity
  {
  private:
    std::string m_Name;
    int m_Age;
  public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    explicit Entity(int age)  //声明为explicit
        : m_Name("Unknown"), m_Age(age) {}
  };

  int main()
  {
    Entity test1("lk");
    Entity test2(23); 
      Entity test3 = "lk"; 
    Entity test4 = std::string("lk");
    Entity test5 = 23; //error！禁用隐式转换


    std::cin.get();
  }
```

加了explicit后还想隐式转换，则可以：

```cpp
Entity test5 = (Entity)23; //ok
```

## C++运算符(操作符)及其重载

- **操作符就是函数。**
- 运算符是给我们使用的一种符号，通常代替一个函数来执行一些事情。比如加减乘除、dereference运算符、箭头运算符、+=运算符、&运算符、左移运算符、new和delete、逗号、圆括号、方括号等等 。
- 运算符重载允许你在程序中定义或者更改一个操作符的行为。
- 应该相当少地使用操作符重载，只在他非常有意义的时候使用。

### “+”和“*”操作符重载

重载和无重载的区别：

> **无重载**（例如JAVA里就没有操作符重载）

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
    Vector2(float x,float y) 
        :x(x),y(y){}
    Vector2 Add(const Vector2& other) const
    {
        return Vector2(x + other.x, y + other.y);
    }
    Vector2 Multiply(const Vector2& other) const
    {
        return Vector2(x * other.x, y * other.y);
    }

};

int main()
{
    Vector2 position(4.0f, 4.0f);
    Vector2 speed(0.5f, 1.5f);
    Vector2 powerup(1.1f, 1.1f); //改变speed
    Vector2 result1 = position.Add(speed.Multiply(powerup)); //无重载方式
    std::cin.get();
}
```

> **使用重载** 需要要【**定义操作符**】比如上述代码中的`“+”`和`*`要定义重载操作

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
    Vector2(float x,float y) 
        :x(x),y(y){}
    Vector2 Add(const Vector2& other) const
    {
        return Vector2(x + other.x, y + other.y);
    }

    Vector2 operator+(const Vector2& other) const  //定义+操作符
    {
        return Add(other);
    }

    Vector2 Multiply(const Vector2& other) const
    {
        return Vector2(x * other.x, y * other.y);
    }
    Vector2 operator*(const Vector2& other) const  //定义*操作符
    {
        return Multiply(other);
    }

};

int main()
{
    Vector2 position(4.0f, 4.0f);
    Vector2 speed(0.5f, 1.5f);
    Vector2 powerup(1.1f, 1.1f); //改变speed
    Vector2 result1 = position.Add(speed.Multiply(powerup)); //无重载方式
    Vector2 result2 = position + speed * powerup; //重载方式

    std::cin.get();
}
```

result2的写法比rusult1看起来好的多。

### 左移操作符的重载

左移操作符的重载

> 如上，现在我们有了这个Vector2，然后我们想要把它打印到控制台

```cpp
Vector2 result2 = position + speed * powerup; //重载方式
std::cout << result2 << std::endl; //C++ 没有与这些操作数匹配的运算符 操作数类型为:
// std::ostream << Vector2
```

> 报错的原因在于**"<<"操作符还没有被重载**，他接受两个参数，一个是输出流，也就是cout，然后另一个就是Vector2 （**操作数类型为: std::ostream << Vector2** ）
> 我们可以**在Vector2类外面**对它进行重载，因为她其实和Vector2其实没什么关系

```cpp
#include <iostream>
struct Vector2
{
      ......

};
//定义左移操作符的重载
std::ostream& operator<<(std::ostream& stream, const Vector2& other)
{
    stream << other.x << "," << other.y;  //这里的stream已经知道要如何打印浮点数.所以我们不需要再对浮点数进行重载
    return stream;
}

int main()
{
    .....
    Vector2 result2 = position + speed * powerup; 
    std::cout << result2 << std::endl; //需要重载<<
    std::cin.get();
}
```

### bool操作符的重载

> == 和 !=

```cpp
#include <iostream>
struct Vector2
{
    ....
    bool operator==(const Vector2& other) const  //定义操作符的重载,如果!=，这里做相应修改即可
    {
        return x == other.x && y == other.y;
    }

    bool operator!=(const Vector2& other) const  //如果!=，这里做相应修改即可
    {
        return !(*this == other);
    }
};
    ....
int main()
{
    ....
    Vector2 result1 = position.Add(speed.Multiply(powerup));
    Vector2 result2 = position + speed * powerup; 

    if (result1 == result2)   //需要对==进行重载操作 （！=同理）
    {
      ....
    }
    std::cin.get();
}
```

## C++this关键字

- C++中有this关键字，通过他我们可以访问成员函数，成员函数就是属于某个类的函数或方法。
- this在一个**const函数**中，this是一个`const Entity* const`或者是`const Entity*`,在一个**非const函数**中，那么它就是一个`Entity*`类型的
- 在函数内部，我们可以引用this，**this是指向这个函数所属的当前对象实例的指针**

> 所以我们可以在C++中写一个**非静态**的方法，为了去调用这个方法，我们需要先**实例化**一个对象，然后再去调用这个方法，所以这个方法必须由一个**有效对象**来调用，而**this关键字**就是指向那个对象的**指针**

```cpp
  class Entity
  {
    public:
      int x,y;
      Entity(int x, int y)
      {
          
      }
  };
```

对于上面Entity类中的代码，构造函数中如果过要对**成员变量x,y**进行赋值我可以用成员初始化的方式来进行赋值。

但是假如我不想这么做，我想要**在函数内部**做这个操作，那可能就会遇到点问题了，因为**成员变量**的名字和**传入参数**的名字**相同**，所以，如果：

```cpp
Entity(int x, int y)
 {
        x = x;     
 }
```

这其实只是在用它自己给这个x参数进行赋值操作，这相当于啥都没干。

我其实真正要做的是引用属于**这个类的x和y**，这个类的成员。而**this关键字**可以让我们做到这点，因为**this是指向当前对象的指针**。

```cpp
Entity(int x, int y)
 {
     Entity* e = this;
        e->x = x;     
 }
```

更清晰一点

```cpp
Entity(int x, int y)
 {
        this->x = x;  //同(*this).x = x;   
        this->y = y; 
 }
```

如果我们要写一个返回其中一个变量的函数的话，比如:

```cpp
class Entity
{
public:
 int x,y;
 Entity(int x, int y)
 {
        this->x = x;   
        this->y = y; 
 }
 int GetX() const  //在函数后面加上const是很常见的，因为他不会修改这个类
 {
 `  Entity* e = this;//ERROR!
        const Entity* e= this;//ok
     `e->x = 5；//ERROR！
 }
};
```

在这个**const函数**(`int GetX() const`)里，我们**不能写**`Entity* e= this`，而应该是`const Entity* e= this`。

因为**函数后面加上const**就意味着我们不能修改这个类，所以**this必须是const**的。

所以也不能写`e->x = 5;`如果没有这个const倒是可以这么写。

```cpp
int GetX() const  
 {
    const Entity* e= this;//ok
    e->x = 5；//ERROR！
        //所以也不能写`e->x = 5;`如果没有这个const倒是可以这么写。
    Entity* e= this;//ERROR！
    e->x = 5；//ok
 }
```

> 另一个用到的场景就是，如果我们想要调用这个Entity类外面的函数，他不是Entity的方法，但是我们想在这个类内部调用一个外部的函数，然后这个函数接受一个Entity类型作为参数，这时候就可以使用**this**

```cpp
class Entity;  //前置声明。
void PrintEntity(Entity* e); //在这里声明
class Entity  
{
  public:
    int x,y;
    Entity(int x, int y)
    {
        // Entity* e = this;
        this->x = x;   
        this->y = y; 
        PrintEntity(this); //我们希望能在这个类里调用PrintEntity,就可以传入this，这就会传入我已经设置了x和y的当前实例
    }
}; 
void PrintEntity(Entity* e) //在这里定义
{
    //print something
}

//如果我想传入一个常量引用，我要做的就是在这里进行解引用this

  void PrintEntity(const Entity& e); //传入常量引用
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
            this->x = x;   
            this->y = y; 
          PrintEntity(*this); // 解引用
      }
  }; 
  void PrintEntity(const Entity& e) 
  {
      //print something
  }
```

在**非const函数**里通过**解引用this**，我们就可**赋值给Entity&**，如果是在**const方法**中，我们会得到一个**const引用**

```cpp
  void PrintEntity(const Entity& e);
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
     		this->x = x;   
     		this->y = y; 
          Entity& e = *this;  //在非const函数里通过解引用this，我们就可赋值给Entity&
          PrintEntity(*this); //解引用this
      }
      int GetX() const  
      {
  		const Entity& e = *this; //在const方法中，我们会得到一个const引用
      }
  }; 
  void PrintEntity(const Entity& e) 
  {
      //print something
  }
```

## C++的对象生存期（栈作用域生存期）

1.基于栈的变量生存周期是什么意思

> 这些分为两部分：一个是你必须要明白对象是如何生存在栈上的，这样你才会写出能正常工作不会崩溃的代码

2.作用域可以是任何东西，比如说函数作用域，还有像if语句作用域，或者for和while循环作用域，或者空作用域、类作用域。

3.每当我们在C++中进入一个作用域，我们是在push栈帧。它不一定就是一个栈帧。

> 当我在push数据时，你可以想象成把一本书放到书堆上，在这个作用域声明的变量一就是你再这本书里写的内容，一旦这个作用域结束，你就把这本书从书堆上拿出来，他就结束了，每个基于栈的变量，就是你再那本书里创建的对象就都结束了

4.**基于栈的变量在我们离开作用域的时候就会被摧毁，内存被释放**。在堆上创建的，当程序结束后才会被系统摧毁。

### 局部作用域创建数组的经典错误

例如：返回一个在作用域内创建的数组

> 如下代码，因为我们没有使用new关键字，所以他不是在堆上分配的，我们只是在栈上分配了这个数组，当我们返回一个指向他的指针时(`return array`)，也就是返回了一个指向栈内存的指针，旦离开这个作用域（CreateArray函数的作用域），这个栈内存就会被回收

```cpp
int CreateArray()
{
    int array[50];  //在栈上创建的
    return array;
}
int main()
{
    int* a = CreateArray(); //不能正常工作
}
```

如果你想要像这样写一个函数，那你一般有**两个选择**

> 1.在堆上分配这个数组，这样他就会一直存在

```cpp
int CreateArray()
{
    int* array = new int[50];  //在堆上创建的
    return array;
}
```

> 2.将创建的数组赋值给一个在这个作用域外的变量

比如说，我在这里创建一个大小为50的数组，然后把这个数组作为一个参数传给这个函数，当然在这个CreateArray函数里就不需要再创建数组了，但是我们可以对传入的数组进行操作，比如，填充数组，因为我们只是闯入了一个指针，所以不会做分配的操作。

```cpp
int CreateArray(int* array)
{
//填充数组
}
int main()
{
   int array[50];
    CreateArray(array); //不能正常工作
}
```

### 基于栈的变量的好处

1.可以帮助我们自动化代码。 比如类的作用域，比如像智能指针smart_ptr，或是unique_ptr，这是一个作用域指针，或者像作用域锁（scoped_lock）。

2.最简单的例子可能是作用域指针，它基本上是一个类，它是一个指针的包装器，在构造时用堆分配指针，然后在析构时删除指针，所以我们可以自动化这个new和delete。

> 创建Entity对象时，我还是想在堆上分配它，但是我想要在跳出作用域时自动删除它，这样能做到吗？我们可以使用标准库中的作用域指针unique_ptr实现。
> 如下，ScopedPtr就是我们写的一个最基本的作用域指针，由于其是在栈上分配的，然后作用域结束的时候，ScopedPtr这个类就被析构，析构中我们又调用delete把堆上的指针删除内存。

```cpp
#include <iostream>

class Entity
{
private:

public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }
    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }

    ~ScopedPtr()
    {
        delete m_Ptr;
    }
};

int main()
{
    {
        ScopedPtr test = new Entity();  //发生隐式转换。虽然这里是new创建的，但是不同的是一旦超出这个作用域，他就会被销毁。因为这个ScopedPtr类的对象是在栈上分配的
        ScopedPtr test(new Entity());  // 这样直接传递指针，避免隐式转换
    }

    std::cin.get();
}
```

上述隐式转换的解释是，`ScopedPtr test = new Entity();` 这里 `new Entity()` 返回的是 `Entity*` 指针，而 `ScopedPtr` 需要一个 `Entity*` 作为构造函数的参数。因此，`new Entity()` 先被隐式转换为 `ScopedPtr`，然后再调用 `ScopedPtr` 的构造函数。这种隐式转换可能会导致意外问题，因此建议使用 `explicit` 关键字避免隐式转换，或者直接使用显式的 `ScopedPtr` 构造。

## C++的智能指针

- 智能指针本质上是原始指针的包装。当你创建一个智能指针，它会调用new并为你分配内存，然后基于你使用的智能指针，这些内存会在某一时刻自动释放。
- 优先使用unique_ptr，其次考虑shared_ptr。

> 尽量使用unique_ptr因为它有一个较低的开销，但如果你需要在对象之间共享，不能使用unique_ptr的时候，就使用shared_ptr

### 作用域指针unique_ptr的使用

- 要访问所有这些智能指针，你首先要做的是包含**memory头文件**
- <font color='red'>它的原理和上一章节自己编写的ScopedPtr类是一样的</font>
- unique_ptr是作用域指针，意味着超出作用域时，它会被销毁然后调用delete。
- unique_ptr是**唯一的**，不可复制，不可分享，因为它的拷贝构造函数被delete了。

> 如果复制一个unique_ptr，会有两个指针，两个unique_ptr指向同一个内存块，如果其中一个死了，它会释放那段内存，也就是说，指向同一块内存的第二个unique_ptr指向了已经被释放的内存。

- <font color='red'>unique_ptr构造函数实际上**是explicit的**，没有构造函数的隐式转换,需要**显式**调用构造函数。</font>
- 最好使用`std::unique_ptr<Entity> entity = std::make_unique<Entity>();` 因为如果构造函数碰巧抛出异常，不会得到一个没有引用的悬空指针从而造成内存泄露,它会稍微安全一些。
- `std::make_unique<>()`是在**C++14**引入的，C++11不支持。

```cpp
#include <iostream>
   #include <memory>

   class Entity
   {
   private:

   public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }

    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
       void Print(){}
   };

   int main()
   {
    {
           //使用unique_ptr的一种方式
        std::unique_ptr<Entity> entity = new Entity(); // error! unique_ptr不能隐式转换
        std::unique_ptr<Entity> entity(new Entity());//ok,可以但不建议
           std::unique_ptr<Entity> entity = std::make_unique<Entity>(); //推荐使用std::make_unique。因为如果构造函数碰巧抛出异常，它会稍微安全一些。std::make_unique<>()是在C++14引入的，C++11不支持。
           entity->Print();  //像一般原始指针使用

    }

    std::cin.get();
   }
```

### 共享指针shared_ptr 的使用

1.shared_ptr的工作方式是**通过引用计数**。

> 引用计数基本上是一种方法，可以跟踪你的指针有多少个引用，一旦引用计数达到零，他就被删除了。
> 例如：我创建了一个共享指针shared_ptr，我又创建了另一个shared_ptr来复制它，我的引用计数是2，第一个和第二个，共2个。当第一个死的时候，我的引用计数器现在减少1，然后当最后一个shared_ptr死了，我的引用计数回到零，内存就被释放。

2.shared_ptr需要分配另一块内存，叫做控制块，用来存储引用计数，如果您首先创建一个new Entity，然后将其传递给shared_ptr构造函数，它必须分配，做2次内存分配。先做一次new Entity的分配，然后是shared_ptr的控制内存块的分配。然而如果你用make_shared你能把它们组合起来，这样更有效率。

```cpp
std::shared_ptr<Entity> sharedEntity = sharedEntity(new Entity());//不推荐！
std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();//ok
```

3.使用格式： `std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();`

```cpp
#include <iostream>
#include <memory>

class Entity
{
private:

public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }
    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
};

int main()
{
    {
        std::shared_ptr<Entity> e0;
        {
        //  std::shared_ptr<Entity> sharedEntity = sharedEntity(new Entity());//不推荐！
            std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();//ok
            e0 = sharedEntity; //可以复制
        } //此时sharedEntity已经“死了”,但没有调用析构，因为e0仍然是活的，并且持有对该Entity的引用，此时计数由2-》1
    } //析构被调用，因为所有的引用都消失了，计数由2-》0，内存被释放

    std::cin.get();
}
```

### 弱指针weak_ptr

1.可以和共享指针shared_ptr一起使用，类似于一个东西存在时，你创建这个弱指针，你会看到两个东西，但是一旦那个实体的东西没了，这个弱指针也没了。

2.weak_ptr可以被复制，但是同时**不会增加额外的控制块来控制计数**，仅仅声明这个指针还活着。

> 当你将一个shared_ptr赋值给另外一个shared_ptr，引用计数++，而若是**把一个shared_ptr赋值给一个weak_ptr时，它不会增加引用计数**。这很好，如果你不想要Entity的所有权，就像你可能在排序一个Entity列表，你不关心它们是否有效，你只需要存储它们的一个引用就可以了。

```cpp
{
    std::weak_ptr<Entity> e0;
    {
        std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();
        e0 = sharedEntity;
    } //此时，此析构被调用，内存被释放
}
```

## C++的拷贝与拷贝构造函数

1.拷贝构造函数的格式：

```cpp
//声明：
T(const T& var);
//定义
T(const T& var){
    //函数体，进行深拷贝 分配空间放副本
}
//不使用拷贝函数，禁止赋值
T(const T& var) = delete;
```

2.每当你编写一个变量被赋值另一个变量的代码时，你**总是**在复制。在指针的情况下，你在复制指针，也就是内存地址，内存地址的数字，就是数字而已，所以不会去新建一块内存，也就是复制指针后，会有两个指针指向同一块内存。

3.“成员树”不包括指针和引用时，浅拷贝和深拷贝没区别。

4.浅拷贝只拷贝基本数据类型（非指针变量）

> 浅拷贝的问题是如果对象中变量带有指针（或引用）,则会发生错误.因为**两个指针指向同一个内存**,**一个对象修改,另一个对象的值也被更改了.** 当在**析构**的时候,会**发生两次free (double free)**同一个内存，造成错误，因为第一次析构的时候，指针指向的内存已经被释放了，使用权部属于它了，你再去释放没有权限。

```cpp
class String
{
private:
    char* m_Buffer;
    unsigned m_Size;
public:
    String(const char* string)
    {
        m_Size = strlen(string); //计算字符串的长度，这样就可以把这个字符串的数据复制到缓冲区中
        m_Buffer = new char[m_Size+1]; //需要一个空终止符，所以+1.(也可以使用strcpy函数（拷贝时，包含了空终止字符）)
        memcpy(m_Buffer,string,m_Size); //是把这个指针复制到我们实际的缓冲区中，这样缓冲区就会被我们的字符填充
        m_Buffer[m_Size] = 0;  //手动在最后添加自己的空终止符。不在上一行代码中写m_Size+1的原因是为了避免：char* string这个字符串不能正常的通过空终止符结束而造成错误。

    }
    ~String()
    {
        delete [] m_Buffer;
    }
    char& operator[](const unsigned index) //[]操作符重载
    {
        return m_Buffer[index];
    }
    friend std::ostream& operator<<(std::ostream& stream, const String& string);//把<<操作符重载函数声明为String类的友元，这样就可以在该重载函数中访问m_Buffer
};
std::ostream& operator<<(std::ostream& stream, const String& string) //<<操作符重载，用来打印我创建的字符串
{
    stream<<string.m_Buffer;
    return stream;
}
```

基于以上的代码，复制一个字符串并输出：

```cpp
int main()
{   
    String string = "Cherno";
    String second = string;
    std::cout << string << std::endl;
    std::cout << second << std::endl;
    std::cin.get();
}
//输出
Cherno
Cherno
------------------------------------------
int main()
{   
    String string = "Cherno";
    String second = string;
    second[2] = 'a';
    std::cout << string << std::endl;
    std::cout << second << std::endl;
    std::cin.get();
}
//输出,两个都被改变了
Charno
Charno
```

正常输出后，运行到`std::cin.get();`此时敲击回车键，**程序崩溃！**

造成该错误的原因时：String类中含有一个指针变量和一个整形变量，复制字符串是，对这两个变量也进行了赋值，但对这个指针只复制了他的内存地址，于时此时由两个指针，**这两个指针指向同一个内存,一个对象修改,另一个对象的值也被更改了.**并且，当在**析构**的时候,会**发生两次free (double free)**同一个内存，造成错误.。

为了解决这个问题，据需要使用**拷贝构造函数**

> 拷贝构造函数是一个构造函数，当你复制第二个字符串时，它会被调用，当你把一个字符串赋值给一个对象时，这个对象也是一个字符串。当你试图创建一个新的变量并给它分配另一个变量时，它（这个变量）和你正在创建的变量有相同的类型。你复制这个变量，也就是所谓的拷贝构造函数

```cpp
class String
{
private:
    char* m_Buffer;
    unsigned m_Size;
public:
    String(const char* string)
    {
        m_Size = strlen(string); 
        m_Buffer = new char[m_Size+1];
        memcpy(m_Buffer,string,m_Size);
        m_Buffer[m_Size] = 0;  

    }
    String(const String& other):m_Size(other.m_Size)  //创建拷贝构造函数
    {
        m_Buffer = new char[m_Size+1];  //分配一个新的缓冲区
        memcpy(m_Buffer,other.m_Buffer,m_Size+1); //知道other的大小了，other字符串已经有了一个空终止字符，因为它是一个字符串，必须有空终止字符。
    } // 此函数为拷贝构造函数，new出一块内存，复制原来的数组
    ~String()
    {
        delete [] m_Buffer;
    }
    char& operator[](const unsigned index) 
    {
        return m_Buffer[index];
    }
    friend std::ostream& operator<<(std::ostream& stream, const String& string);
};
std::ostream& operator<<(std::ostream& stream, const String& string) 
{
    stream<<string.m_Buffer;
    return stream;
}
```

## C++的箭头操作符

**1.特点：**

- 箭头运算符必须是类的成员。
- 一般将箭头运算符定义成了const成员，这是因为与递增和递减运算符不一样，获取一个元素并不会改变类对象的状态。

**2.对箭头运算符返回值的限定**

> 箭头运算符的重载**永远不能丢掉成员访问**这个最基本的含义。当我们重载箭头时，可以改变的是箭头从哪个对象当中获取成员，而箭头获取成员这一事实则永远不变。 对于形如point->mem的表达式来说，point必须是指向类对象的指针或者是一个重载了operator->的类的对象。根据point类型的不同，point->mem分别等价于

```cpp
(*point).mem； //point 是一个内置的指针类型
point.operator()->mem； //point是类的一个对象
```

重载的箭头运算符**必须返回类的指针或者自定义了箭头运算符的某个类的对象。**

**3.三种应用场景**

1)可用于指针调用成员：p->x 等价于 (*p).x

2)重载箭头操作符

```cpp
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() 
    {
        std::cout << "Hello!" << std::endl;
    }
};
class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()  //重载操作符
    {
        return m_Ptr;
    }
};

int main()
{
    {
        ScopedPtr entity = new Entity();
        entity->Print();
    }
    std::cin.get();
}
```

进一步,可以写为const版本的：

```cpp
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() const   //添加const
    {
        std::cout << "hello!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()
    {
        return m_Ptr;
    }
    const Entity* operator->() const //添加const
    {
        return m_Ptr;
    }
};

int main()
{
    {
        const ScopedPtr entity = new Entity(); //如果是const，则上面代码要改为const版本的。
        entity->Print();
    }
    std::cin.get();
}
```

3)可用于计算成员变量的offset：

> 引自B站评论：
> 因为"指针->属性"访问属性的方法实际上是通过把指针的值和属性的偏移量相加，得到属性的内存地址进而实现访问。 而把指针设为nullptr(0)，然后->属性就等于0+属性偏移量。编译器能知道你指定属性的偏移量是因为你把nullptr转换为类指针，而这个类的结构你已经写出来了(float x,y,z)，float4字节，所以它在编译的时候就知道偏移量(0,4,8)，所以无关对象是否创建

```cpp
struct vec2
{
    int x,y;
    float pos,v;
};
int main()
{   
    int offset = (int)&((vec2*)nullptr)->x; // x,y,pos,v的offset分别为0,4,8,12
    std::cout<<offset<<std::endl;
    std::cin.get();
}
```

##  C++的动态数组(std::vector)

1.vector本质上是一个动态数组,是内存连续的数组，<font color='red'>注意它创建的内存在堆区，而array在栈区</font>

2.它的使用需要包含头文件`#include <vector>`

3.使用格式：

类型尽量**使用对象**而非指针。

```cpp
std::vector<T> a；//T是一种模板类型，尽量使用对象而非指针
```

4.添加元素

```cpp
a.push_back(element); // 后面插入
    //定义一个类
    struct Vertex
    {
        float x, y, x;
    }

    std::vector<Vertex> vertices; //定义一个Vertex类型的动态数组
    vertices.push_back({ 1, 2, 3 });//列表初始化（结构体或者类，可以按成员声明的顺序用列表构造）
    vertices.push_back({ 4, 5, 6 });//同：vertices.push_back(Vertex(4, 5, 6)
    vertices.push_back({ 7, 8, 9 });
```

5.初始化

遍历

> for遍历

```cpp
for(int i =0; i < vertices.size();i++)
{
    std::cout << vertices[i] << std::endl;
}
```

> 范围for循环遍历

```cpp
for(Vertex& v : vertices)  //引用，避免复制浪费。
{
    std::cout << v << std::endl;
}
```

6.清除数组列表

```cpp
vertices.clear();
```

7.清除指定元素

> 例如：清除第二个元素

```cpp
vertices.erase(vertices.begin()+1) //参数是迭代器类型，不能直接用下标索引
```

8.参数传递时，如果不对数组进行修改，请使用引用类型传参。

```cpp
void Function(const std::vector<T>& vec){};
```

## C++的std::vector使用优化

vecctor的优化策略：

> **问题1：**当向vector数组中**添加新元素**时，为了扩充容量，**当前的vector的内容会从内存中的旧位置复制到内存中的新位置**(产生一次复制)，然后删除旧位置的内存。 简单说，push_back时，容量不够，会自动调整大小，重新分配内存。这就是将代码拖慢的原因之一。 **解决办法：** vertices.reserve(n) ，直接指定容量大小，避免重复分配产生的复制浪费。
> **问题2：**在非vector内存中创建对象进行初始化时，即push_back() 向容器尾部添加元素时，首先会创建一个临时容器对象（不在已经分配好内存的vector中）并对其追加元素，然后再将这个对象拷贝或者移动到【我们真正想添加元素的容器】中 。这其中，就造成了一次复制浪费。 **解决办法：** **emplace_back**，直接在容器尾部创建元素，即直接在已经分配好内存的那个容器中直接添加元素，不创建临时对象。

简单的说：

> **reserve提前申请内存**，避免动态申请开销 **emplace_back直接在容器尾部创建元素**，省略拷贝或移动过程

```cpp
#include <iostream>
#include <vector>

struct Vertex
{
    float x, y, z;

    Vertex(float x, float y, float z)
        : x(x), y(y), z(z)
    {
    }

    Vertex(const Vertex& vertex)
        : x(vertex.x), y(vertex.y), z(vertex.z)
    {
        std::cout << "Copied!" << std::endl;
    }
};

int main()
{
    std::vector<Vertex> vertices;
    vertices.push_back(Vertex(1, 2, 3 )); //同vertices.push_back({ 1, 2, 3 });
    vertices.push_back(Vertex(4, 5, 6 ));
    vertices.push_back(Vertex(7, 8, 9 ));

    std::cin.get();
}
```

输出：

```cpp
Copied!
Copied!
Copied!
Copied!
Copied!
Copied!
```

**发生六次复制的原因：**

理解一：

> 环境:VS2019，x64，C++17标准，经过我自己的测试，vector扩容因子为1.5，初始的capacity为0.
> 第一次push_back，capacity扩容到1，临时对象拷贝到真正的vertices所占内存中，第一次Copied；第二次push_back，发生扩容，capacity扩容到2，vertices发生内存搬移发生的拷贝为第二次Copied，然后再是临时对象的搬移，为第三次Copied；接着第三次push_back，capacity扩容到3（2*1.5 = 3，3之后是4，4之后是6...），vertices发生内存搬移发生的拷贝为第四和第五个Copied，然后再是临时对象的搬移为第六个Copied；

理解二：

```cpp
std::vector<Entity> e;
    Entity data1 = { 1,2,3 }; 
    e.push_back( data1); // data1->新vector内存
    Entity data2 = { 1,2,3 }; 
    e.push_back( data2 ); //data1->新vector内存   data2->vector新vector内存  删除旧vector内存
    Entity data3 = { 1,2,3 };
    e.push_back(data3);  // data1->新vector内存  data2->vector新vector内存  data3->vector新vector内存  删除旧vector内存
所以他的输出的次数分别是1，3，6
他的复制次数你可以这样理解递增。 1+2+3+4+5+....
```

解决:

```cpp
int main()
{   
    std::vector<Vertex> vertices;
    //ver 1 : copy 6 times
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 2 : copy 3 times
    vertices.reserve(3);
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 3 : copy 0 times
    vertices.reserve(3);
    vertices.emplace_back(1, 2, 3);
    vertices.emplace_back(4, 5, 6);
    vertices.emplace_back(7, 8, 9);

    std::cin.get();
}
```

## C++中如何处理多返回值

### 方法一：通过函数参数传引用或指针的方式

> 把函数定义成`void`，然后通过**参数引用传递**的形式“返回”两个字符串，这个实际上是修改了目标值，而不是返回值，但某种意义上它确实是返回了两个字符串，而且没有复制操作，技术上可以说是很好的。但这样做会使得函数的形参太多了，可读性降低，有利有弊 。

```cpp
#include <iostream>
void GetUserAge(const std::string& user_name,bool& work_status,int& age)
{
    if (user_name.compare("xiaoli") == 0)
    {
        work_status = true;
        age = 18;
    }
    else
    {
        work_status = false;
        age = -1;
    }
}

int main()
{
    bool work_status = false;
    int age = -1;
    GetUserAge("xiaoli", work_status, age);
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age << std::endl;
    getchar();
    return 0;
}
```

### 方法二： 通过函数的返回值是一个`array`（数组）或vector

> 当然，这里也可以返回一个vector，同样可以达成返回多个数据的目的。
> 不同点**是Array是在栈上创建，而vector会把它的底层储存在堆上**，所以从技术上说，返回Array会更快
> 但以上方法都**只适用于相同类型**的多种数据的返回

```cpp
//设置是array的类型是stirng，大小是2
std::array<std::string, 2> ChangeString() {
    std::string a = "1";
    std::string b = "2";

    std::array<std::string, 2> result;
    result[0] = a;
    result[1] = b;
    return result;

    //也可以return std::array<std::string, 2>(a, b);
}
```

### 方法三：使用std::pair返回两个返回值

> 可以**返回两个不同类型**的数据返。
> 使用std::pair这种抽象数据结构，该数据结构可以绑定两个异构成员。这种方式的弊端是**只能返回两个值。**

```cpp
#include <iostream>

std::pair<bool, int> GetUserAge(const std::string& user_name)
{
    std::pair<bool, int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_pair(true, 18);
    }
    else
    {
        result = std::make_pair(false, -1);
    }

    return result;
}

int main()
{
    std::pair<bool, int> result = GetUserAge("xiaolili");
    std::cout << "查询结果：" << result.first << "   " << "年龄：" << result.second << std::endl;
    getchar();
    return 0;
}
```

### 方法四：使用std::tuple返回三个或者三个以上返回值

> std::tuple这种抽象数据结构可以将三个或者三个以上的异构成员绑定在一起，返回std::tuple作为函数返回值理论上可以返回三个或者三个以上的返回值。
> `tuple`相当于一个类，它可以包含x个变量，但他不关心类型，用`tuple`需要包含头文件`#include`

```cpp
#include <iostream>
#include <tuple>

std::tuple<bool, int,int> GetUserAge(const std::string& user_name)
{
    std::tuple<bool, int,int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_tuple(true, 18,0);
    }
    else
    {
        result = std::make_tuple(false, -1,-1);
    }

    return result;
}

int main()
{
    std::tuple<bool, int,int> result = GetUserAge("xiaolili");

    bool work_status;
    int age;
    int user_id;

    std::tie(work_status, age, user_id) = result;
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age <<"   "<<"用户id:"<<user_id <<std::endl;
    getchar();
    return 0;
}
```

### 方法五：返回一个结构体(推荐)

> 结构体是在栈上建立的，所以在技术上速度也是可以接受的
> 而且不像用pair的时候使用只能`temp.first, temp.second`，这样不清楚前后值是什么，可读性不佳。而如果换成`temp.str, temp.val`后可读性极佳，永远不会弄混！

```cpp
#include <iostream>
struct result {
    std::string str;
    int val;
};
result Function () {
    return {"1", 1};//C++新特性，可以直接这样子让函数自动补全成结构体
}
int main() {
    auto temp = Function();
    std::cout << temp.str << ' ' << temp.val << std::endl;
}
--------------------------------------------
#include <iostream>
using namespace std;

struct Result
{
    int add;
    int sub;
};

Result operation(int a, int b)
{
    Result ret;
    ret.add = a + b;
    ret.sub = a - b;
    return ret;
}

int main()
{
    Result res;
    res = operation(5, 3);
    cout << "5+3=" << res.add << endl;
    cout << "5-3=" << res.sub << endl;
}
```

## C++的模板

**模板**：模板允许你定义一个可以根据你的用途进行编译的模板（有意义下）。故所谓模板，就是让编译器基于DIY的规则去为你写代码 。

### 函数的模板（对形参）

> 不使用模板

```cpp
  void Print(int temp) {
      cout << temp;
  }
  void Print(string temp) {
      cout << temp;
  }
  void Print(double temp) {
      cout << temp;
  }
  int main() {
      Print(1);
      Print("hello");
      Print(5.5);
      //如果要用一个函数输出三个类型不同的东西，则要手动定义三个不同重载函数
      //这其实就是一种复制粘贴就可以完成的操作
  }
```

> 使用模板

格式： **`template<typename T>`**

```cpp
template<typename T> void Print(T temp) {
    //把类型改成模板类型的名字如T就可以了
    cout << temp;
}
//干净简洁
int main() {
    Print(1);
    Print("hello");
    Print(5.5);
}
```

> 通过`template`定义，则说明定义的是一个模板，它会在编译期被评估，**所以`template`后面的函数其实不是一个实际的代码**，**只有当我们实际调用时，模板函数才会基于传递的参数来真的创建** 。 只有当真正调用函数的时候才会被实际创建 。

模板参数

```cpp
template<typename T> void Print(T temp) {
    cout << temp;
}
int main() {
    Print(96);//这里其实是隐式的传递信息给模板，可读性不高
    Print<int>(96);//可以显示的定义模板参数，声明函数接受的形参的类型！！！
    Print<char>(96);//输出的可以是数字，也可以是字符！这样的操纵性强了很多！！！
}
```

### 类的模板

> 传递数字给模板，来指定要生成的类

```cpp
//不仅仅是typename!
template<int N> class Array {
private:
    //在栈上分配一个数组，而为了知道它的大小，要用模板传一个数字N过来
    int m_Array[N];
};
int main() {
    Array<5> array;//用尖括号给模板传递构造的规则。
}
```

> 传多个规则给模板，**用逗号隔开就行**

```cpp
//可以传类型，也可以传数字，功能太强大了
//两个模板参数：类型和大小
template<typename T, int size> class Array {
private:
    T m_Array[size];
};
int main() {
    Array<int, 5> array;
}
```

## C++的堆和栈内存的比较

1.当我们的程序开始的时候，程序被分成了一堆不同的内存区域，除了堆和栈以外，还有很多东西，但我们最关心这两个 。

2.**栈**通常是一个预定义大小的内存区域，通常约为2兆字节左右。**堆**也是一个预定义了默认值的区域，**但是它可以**随着应用程序的进行而改变。

3.**栈和堆内存区域的实际位置（物理位置）在ram中完全一样**（并不是一个存在CPU缓存而另一个存在其他地方）

> 在程序中，内存是用来实际储存数据的。我们需要一个地方来储存允许程序所需要的数据（比如局部变量or从文件中读取的东西）。而栈和堆，它们就是可以储存数据的地方，但**栈和堆的工作原理非常非常不同**，但本质上它们做的事情是一样的

4.栈和堆的区别

区别一：定义格式不同

```cpp
//在栈上分配
int val = 5; 
//在堆上分配
int *hval = new int;    //区别是，我们需要用new关键词来在堆上分配
*hval = 5;
```

区别二：内存分配方式不同

对栈来说：

> **在栈上**，分配的内存都是**连续**的。添加一个int，则**栈指针（栈顶部的指针）**就移动4个字节，所以连续分配的数据在内存上都是**连续**的。栈分配数据是直接把数据堆在一起（所做的就是移动栈指针），所以栈分配数据会很快 。
> 如果离开作用域，在栈中分配的所有内存都会弹出，内存被释放。

对堆来说

> **在堆上**，分配的内存都是**不连续**的，`new`实际上做的是在内存块的**空闲列表**中找到空闲的内存块，然后把它用一个指针圈起来，然后返回这个指针。（但如果**空闲列表**找不到合适的内存块，则会询问**操作系统**索要更多内存，而这种操作是很麻烦的，潜在成本是巨大的）
> 离开作用域后，堆中的内存仍然存在

建议： **能在栈上分配就在栈上分配**，**不能够在栈上分配时或者有特殊需求时（比如需要生存周期比函数作用域更长，或者需要分配一些大的数据），才在堆上分配**

## C++的宏

1.**预处理阶段** ：当编译C++代码时，首先**预处理器**会过一遍C++所有的**以#符号开头（这是预编译指令符号）的语句，当预编译器将这些代码评估完后给到编译器去进行实际的编译**。

2.**宏和模板的区别**：**发生时间**不同，宏是在**预处理阶段**就被评估了，而模板会被评估的更晚一点。

3.**用宏的目的：**写一些宏将代码中的文本**替换**为其他东西（**纯文本替换**）（不一定是简单的替换，是可以自定义调用宏的方式的）

```cpp
#defind WAIT std::cin.get()
//这里可以不用放分号，如果放分号就会加入宏里面了
int main() {
    WAIT;
    //等效于std::cin.get()，属于纯文本替换
    //但单纯做这种操作是很愚蠢的，除了自己以外别人读代码会特别痛苦
}
```

4.宏的用法之一：**宏是可以发送参数的**

```cpp
#include <iostream>
//宏是可以传递参数的，虽然参数也是复制粘贴替换上去的，并没有像函数那样讲究
#define log(x) std::cout << x << std::endl
int main() {
    log("hello");
    //这样子会输出“hello”
    return 0;
}
```

5.宏可以辅助调试

> 在Debug模式下会有很多日志的输出，但是在Release模式下就不需要日志的输出了。正常的方法可能会删掉好多的输出日志的语句或者函数，**但是用宏可以直接取消掉这些语句**

利用宏中的`#if，#else`，`endif`来实现。如：

```cpp
#include <iostream>

#defind PR_DEBUG 1 //可以在这里切换成0，作为一个开关
#if PR_DEBUG == 1   //如果PR_DEBUG为1
#defind LOG(x) std::cout << x << std::endl  //则执行这个宏
#else   //反之
#defind LOG(x)   //这个宏什么也不定义，即是无意义
#endif    //结束

int main() {
    LOG("hello");
    return 0;
}
```

如果在Debug(PR_DEBUG == 1)模式下，则会打印日志，如果在Release(PR_DEBUG == 0)模式，则在**预处理阶段就会把日志语句给删除掉**。

利用`#if 0`和`#endif`删除一段宏.

```cpp
#include <iostream>

#if 0   //从这里到最后的endif的宏都被无视掉了，某种意义上的删除

#defind PR_DEBUG 1 
#if PR_DEBUG == 1   
#defind LOG(x) std::cout << x << std::endl  
#else   
#defind LOG(x)  
#endif    

#endif  //结束

int main() {
    LOG("hello");
    return 0;
}
```

## C++的auto关键字

**auto的使用场景：**

> 在使用iterator 的时候，如：

```cpp
std::vector<std::string> strings;
strings.push_back("Apple");
strings.push_back("Orange");
for (std::vector<std::string>::iterator it = strings.begin(); //不使用auto
    it != strings.end(); it++)
{
    std::cout << *it << std::endl;
}
for (auto it = strings.begin(); it != strings.end(); it++) //使用auto
{
    std::cout << *it << std::endl;
}
```

> 当类型名过长的时候可以使用auto

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>

class Device{};

class DeviceManager
{
private:
    std::unordered_map<std::string, std::vector<Device *>> m_Devices;
public:
    const std::unordered_map<std::string, std::vector<Device *>> &GetDevices() const
    {
        return m_Devices;
    }
};

int main()
{
    DeviceManager dm;
    const std::unordered_map<std::string, std::vector<Device *>> &devices = dm.GetDevices();//不使用auto
    const auto& devices = dm.GetDevices(); //使用auto

    std::cin.get();
}
```

除此之外类型名过长的时候也可以使用using或typedef方法：

```cpp
using DeviceMap = std::unordered_map<std::string, std::vector<Device*>>;
typedef std::unordered_map<std::string, std::vector<Device*>> DeviceMap;

const DeviceMap& devices = dm.GetDevices();
```

**auto使用建议**：如果不是上面两种应用场景，请尽量不要使用auto！能不用，就不用！

## C++的静态数组（std::array)

1.std::array是一个实际的标准数组类，是C++标准模板库的一部分。

2.**静态**的是指**不增长的数组**，当**创建array**时就要**初始化其大小**，不可再改变。

3.使用格式

```cpp
#include <array>  // 先要包含头文件
int main() {
    std::array<int, 5> data;  //定义，有两个参数，一个指定类型，一个指定大小
    data[0] = 1;
    data[4] = 10;
    return 0;
}
```

4.array和原生数组都是创建在栈上的（vector是在堆上创建底层数据储存的）

5.原生数组越界的时候不会报错，而array会有越界检查，会报错提醒。

6.使用std::array的好处是可以**访问它的大小**（通过s**ize()**函数），它是一个**类**。

```cpp
#include<iostream>
#include<array>

void PrintArray(const std::array<int, 5>& data)  //显式指定了大小
{
    for (int i = 0;i < data.size();i++)  //访问数组大小
    {
        std::cout << data[i] << std::endl;
    }
}
int main()
{
    std::array<int, 5> data;
    data[0] = 0;
    data[1] = 1;
    data[2] = 2;
    data[3] = 3;
    data[4] = 4;
    PrintArray(data);
    std::cin.get();
}
```

如何传入一个**标准数组作为参数**，但**不知道数组的大小**？

方法：使用模板

```cpp
#include <iostream>
#include <array>

template <typename T>
void printarray(const T &data)
{
    for (int i = 0; i < data.size(); i++)
    {
        std::cout << data[i] << std::endl;
    }
}

template <typename T, unsigned long N> // or // template <typename T, size_t N>
void printarray2(const std::array<T, N> &data)
{
    for (int i = 0; i < N; i++)
    {
        std::cout << data[i] << std::endl;
    }
}

int main()
{
    std::array<int, 5> data;
    data[0] = 2;
    data[4] = 1;
    printarray(data);
    printarray2(data);
}
//代码参考：https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno57.cpp
```

## C语言风格的函数指针

1.定义方式：

```cpp
void(*function)() = Print; //很少用，一般用auto关键字
```

2.函数指针的使用

> 无参数的函数指针

```cpp
void Print() {
    std::cout << "hello，world" << std::endl;
}
int main() {
    //void(*function)() = Print； 正常写法，但一般用auto就可以了
    auto function = Print();    //ERROR!，auto无法识别void类型
    auto function = Print;  //OK!，去掉括号就不是在调用这个函数，而是在获取函数指针，得到了这个函数的地址。就像是带了&取地址符号一样"auto function = &Print;""(隐式转换)。
    function();//调用函数
    //这里函数指针其实也用到了解引用（*），这里是发生了隐式的转化，使得代码看起来更加简洁明了！
}
//输出：
hello,world
```

> 对于有参数的函数指针，在使用的时候传上参数即可

```cpp
void Print(int a) {
    std::cout << a << std::endl;
}
int main() {
    auto temp = Print;  //正常应该是 void(*temp)(int) = Print,太过于麻烦，用auto即可
    temp(1);    //在用函数指针的时候也传参数进去就可以正常使用了
}
```

> 也可以用typedef或者using来使用函数指针

```cpp
#include<iostream>

void HelloWorld()
{
    std::cout << "Hello World!" << std::endl;
}
int main()
{
    typedef void(*HelloWorldFunction)(); 

    HelloWorldFunction function = HelloWorld;
    function();

    std::cin.get();
}
```

为什么要首先使用函数指针

> 如果需要**将一个函数作为另一个函数的形参**，那么就要需要函数指针 .

```cpp
void Print(int val) {
    std::cout << val << std::endl;
}

//下面就将一个函数作为形参传入另一个函数里了
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp); //就可以在当前函数里用其他函数了
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, Print); //这里就是传入了一个函数指针进去！！！！
}
```

**优化：lambda**

> **lambda本质上是一个普通的函数**，只是它不像普通函数这样声明，它是我们的代码**在过程中生成的，用完即弃的函数**，不算一个真正的函数，是**匿名函数** 。
> 格式：`[] ({形参表}) {函数内容}`

```cpp
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp);     //正常调用lambda函数
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, [](int val){ std::cout << val << std::endl; });     //如此简单的事就交给lambda来解决就好了
}
```

## C++的lambda

[官方参考网站：https://en.cppreference.com/w/cpp/language/lambda]

1.lambda本质上是一个**匿名函数**。 用这种方式创建函数不需要实际创建一个函数 ，它就像一个**快速的一次性函数** 。 lambda更像是一种变量，在实际编译的代码中作为一个符号存在，而不是像正式的函数那样。

2.使用场景：

> 在我们会设置函数指针指向函数的任何地方，我们都可以将它设置为lambda

3.lambda表达式的写法(使用格式)：`[]( {参数表} ){ 函数体 }`

> 中括号**表示的是**捕获，作用是**如何传递变量** lambda使用**外部（相对）**的变量时，就要**使用捕获**。

如果使用捕获,则：

- 添加头文件： `#include <functional>`
- 修改相应的函数签名 `std::function <void(int)> func`替代 `void(*func)(int)`
- 捕获[]使用方式：

> `[=]`，则是将所有变量**值传递**到lambda中
> `[&]`，则是将所有变量**引用传递**到lambda中
> `[a]`是将变量a通过值传递，如果是`[&a]`就是将变量a引用传递
> 它可以有0个或者多个捕获

```cpp
//If the capture-default is `&`, subsequent simple captures must not begin with `&`.
struct S2 { void f(int i); };
void S2::f(int i)
{
    [&]{};          // OK: by-reference capture default
    [&, i]{};       // OK: by-reference capture, except i is captured by copy
    [&, &i] {};     // Error: by-reference capture when by-reference is the default
    [&, this] {};   // OK, equivalent to [&]
    [&, this, i]{}; // OK, equivalent to [&, i]
}


//If the capture-default is `=`, subsequent simple captures must begin with `&` or be `*this` (since C++17) or `this` (since C++20). 
struct S2 { void f(int i); };
void S2::f(int i)
{
    [=]{};        // OK: by-copy capture default
    [=, &i]{};    // OK: by-copy capture, except i is captured by reference
    [=, *this]{}; // until C++17: Error: invalid syntax
                  // since C++17: OK: captures the enclosing S2 by copy
    [=, this] {}; // until C++20: Error: this when = is the default
                  // since C++20: OK, same as [=]
}

详情参考：https://en.cppreference.com/w/cpp/language/lambda
```

事例：

```cpp
#include <iostream>
#include <vector>
#include <functional>
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp);     //正常调用lambda函数
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //函数指针的地方都可以用auto来简化操作，lambda亦是
    //这样子来定义lambda表达式会更加清晰明了
    auto lambda = [](int val){ std::cout << val << std::endl; }
    ForEach(values, lambda);    
}
-------------------------------------------------
//lambda可以使用外部（相对）的变量，而[]就是表示打算如何传递变量
#include <functional>   //要用捕获就必须要用C++新的函数指针！
//新的函数指针的签名有所不同！
void ForEach(const std::vector<int>& values, const std::function<void(int)>& func) {
    for (int temp : values) {
        func(temp);     
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //注意这里的捕获必须要和C++新带的函数指针关联起来！！！
    int a = 5;  //如果lambda需要外部的a向量
    //则在捕获中写入a就好了
    auto lambda = [a](int val){ std::cout << a << std::endl; }
    ForEach(values, lambda);    
}
```

我们有一个可选的修饰符**mutable**，它**允许lambda函数体**修改通过拷贝传递捕获的参数。若我们在lambda中给a赋值会报错，需要写上mutable 。

```cpp
int a = 5;
auto lambda = [=](int value) mutable { a = 5; std::cout << "Value: " << value << a << std::endl; };
```

另一个使用lambda的场景**`find_if`**

> 我们还可以写一个lambda接受vector的整数元素，遍历这个vector找到比3大的整数，然后返回它的迭代器，也就是满足条件的第一个元素。
> **`find_if`**是一个搜索类的函数，区别于`find`的是：**它可以接受一个函数指针来定义搜索的规则，返回满足这个规则的第一个元素的迭代器**。这个情况就很适合lambda表达式的出场了

```cpp
#include <algorithm>  // 使用lambda必须要包含的类
#include <vector>
#include <iostream>

int main() {
    std::vector<int> values = { 1, 5, 3, 4, 2 };
    //下面就用到了lambda作为函数指针构成了find_it的规则
    auto it = std::find_if(values.begin(), values.end(), [](int value) { return value > 3; });  //返回第一个大于3的元素的迭代器 
    std::cout << *it << std::endl;  //将其输出
}
```

## 为什么不使用using namespace std

1.不容易分辨各类函数的来源

> 比如我在一个自己的库中定义了一个vector，而标准库里又有一个vector，那么如果用了using namespace std 后，所用的vector到底是哪里的vector呢？

```cpp
std::vector<int>vec1;   //good
DiyClass::vector<int>vec2   //good

using namespace std;
using namespace DiyClass    //万一有其他人用了DiyClass的命名空间
vector<int>vec3 //便会有歧义，完全不知道到底是哪里的vector
```

2.一定**不要**在**头文件内**使用`using namespace std`

> 如果别人用了你的头文件，就会把这些命名空间用在了你原本没有打算用的地方，会导致莫名其妙的产生bug，如果有大型项目，追踪起来会很困难。 如果公司有自己的模板库，然后里面有很多重名的类型或者函数，就容易弄混；

3.可以就在一些小作用域里用，**但能不用就不用！养成良好的代码书写习惯！**

## C++的命名空间

1.命名空间是C++独有，C是没有的，故写C时会有命名冲突的风险。

2.**类本身就是名称空间** 。

> 类外使用一个类内的成员需要加`::`

3.命名空间（名称空间）的主要目的是**避免命名冲突**，便于管理各类命名函数。使用名称空间的原因，是因为我们希望**能够在不同的上下文中调用相同的符号**。

```cpp
#include <iostream>
#include <string>
#include <algorithm>
namespace apple {
    void print(const char *text) {
        std::cout << text << std::endl;
    }
}

namespace orange {
    void print(const char *text) {
        std::string temp = text;
        std::reverse(temp);
        std::cout << temp << std::endl;
    }
}
int main() {
    //using namespace apple::print; //单独引出一个print函数
    //using namespace apple;//引出apple名称空间的所有成员

    apple::print("hello");  //输出正常text
    orange::print("world"); //输出反转的text
}
```

拓展:详情请参考原文链接：https://zhuanlan.zhihu.com/p/441602923

大型程序往往会使用多个独立开发的库，这些库会定义大量的全局名字，如类、函数和模板等，不可避免会出现某些名字相互冲突的情况。命名空间`namespace`分割了全局命名空间，其中每个命名空间是一个作用域。

```cpp
namespace foo {
    class Bar { /*...*/ };
}  // 命名空间结束后无需分号
```

**命名空间定义**

**1. 每个命名空间都是一个作用域**

> 同其他作用域类似，命名空间中的每个名字都必须表示该空间内的唯一实体。因为不同命名空间的作用域不同，所以在不同命名空间内可以有相同名字的成员。

**2. 命名空间可以不连续**

> 命名空间的定义可以不连续的特性使得我们可以将几个独立的接口和实现文件组成一个命名空间，定义多个类型不相关的命名空间也应该使用单独的文件分别表示每个类型。

**3. 模板特例化**

> 模板特例化必须定义在原始模板所属的命名空间中，和其他命名空间名字类似，只要我们在命名空间中声明了特例化，就能在命名空间外部定义它了：

```cpp
// 我们必须将模板特例化声明成std的成员
namespace std {
    template <> struct hash<Foo>;
}

// 在std中添加了模板特例化的声明后，我们就可以在命名空间std的外部定义它了
template<> struct std::hash<Foo> {
    size_t operator()(const Foo& f) const {
        return hash<string>()(f.str) ^
            hash<double>()(f.d);
    }
};
```

**4. 全局命名空间**

全局作用域中定义的名字（即在所有类、函数以及命名空间之外定义的名字）也就是定义在全局命名空间`global namespace`中。全局作用域是隐式的，所以它并没有名字，下面的形式表示全局命名空间中一个成员：

```cpp
::member_name
```

**5. 嵌套的命名空间**

```cpp
namespace foo {
    namespace bar {
        class Cat { /*...*/ };
    }
}

// 调用方式
foo::bar::Cat
```

**6. 内联命名空间**

C++11新标准引入了一种新的嵌套命名空间，称为内联命名空间`inline namespace`。内联命名空间可以被外层命名空间直接使用。定义内联命名空间的方式是在关键字`namespace`前添加关键字`inline`：

```cpp
// inline必须出现在命名空间第一次出现的地方
inline namespace FifthEd {
    // ...
}
// 后续再打开命名空间的时候可以写inline也可以不写
namespace FifthEd {  // 隐式内敛
    // ...
}
```

当应用程序的代码在一次发布和另一次发布之间发生改变时，常使用内联命名空间。例如我们把第五版`FifthEd`的所有代码放在一个内联命名空间中，而之前版本的代码都放在一个非内联命名空间中：

```cpp
namespace FourthEd {
    // 第4版用到的其他代码
    class Cat { /*...*/ };
}

// 命名空间cplusplus_primer将同时使用这两个命名空间
namespace foo {
#include "FifthEd.h"
#include "FoutthEd.h"
}
```

因为`FifthEd`是内联的，所以形如`foo::`的代码可以直接获得`FifthEd`的成员，如果我们想用到早期版本的代码，则必须像其他嵌套的命名空间一样加上完整的外层命名空间名字：

```cpp
foo::FourthEd::Cat
```

**7. 未命名的命名空间**

关键字`namespace`后紧跟花括号括起来的一系列声明语句是未命名的命名空间`unnamed namespace`。未命名的命名空间中定义的变量具有静态生命周期：它们在第一次使用前被创建，直到程序结束时才销毁。

> *Tips：每个文件定义自己的未命名的命名空间，如果两个文件都含有未命名的命名空间，则这两个空间互相无关。在这两个未命名的命名空间里面可以定义相同的名字，并且这些定义表示的是不同实体。如果一个头文件定义了未命名的命名空间，则该命名空间中定义的名字将在每个包含了该头文件的文件中对应不同实体。*

和其他命名空间不同，未命名的命名空间仅在特定的文件内部有效，其作用范围不会横跨多个不同的文件。未命名的命名空间中定义的名字的作用域与该命名空间所在的作用域相同，如果未命名的命名空间定义在文件的最外层作用域中，则该命名空间一定要与全局作用域中的名字有所区别：

```cpp
// i的全局声明
int i;
// i在未命名的命名空间中的声明
namespace {
    int i;  
}
// 二义性错误: i的定义既出现在全局作用域中, 又出现在未嵌套的未命名的命名空间中
i = 10;
```

> *未命名的命名空间取代文件中的静态声明：*
> *在标准C++引入命名空间的概念之前，程序需要将名字声明成`static`的以使其对于整个文件有效。在文件中进行静态声明的做法是从C语言继承而来的。在C语言中，声明为`static`的全局实体在其所在的文件外不可见。* ***在文件中进行静态声明的做法已经被C++标准取消了，现在的做法是使用未命名的命名空间。***

## C++的线程

1.使用多线程，首先要添加头文件`#include <thread>`。

2.在**Linux平台**下编译时需要加上**"-lpthread"链接库**

3.创建一个线程对象：**`std::thread objName (一个函数指针以及其他可选的任何参数)`**

4.等待一个线程完成它的工作的方法 :`worker.join()`

> **这里的线程名字是worker，换其他的也可以,自己决定的）** 调用`join`的目的是：**在主线程上等待 工作线程 完成所有的执行之后，再继续执行主线程**

```cpp
//这个代码案例相当无用，只是为了展示多线程的工作而展示的。
#include <iostream>
#include <thread>
void DoWork() {
    std::cout << "hello" << std::endl;
}
int main() {
    //DoWork即是我们想要在另一个执行线程中发生的事情
    std::thread worker(DoWork); //这里传入的是函数指针！！！函数作为形参都是传函数指针！！！
    //一旦写完这段代码，它就会立即启动那个线程，一直运行直到我们等待他退出
    worker.join();  //join函数本质上，是要等待这个线程加入进来（而线程加入又是另一个复杂的话题了）

    //因为cin.get()是join语句的下一行代码，所以它不会运行，直到DoWork函数中的所有内容完成！
    std::cin.get();
}
#include <iostream>
#include <thread>

static bool is_Finished = false;
void DoWork() {
    using namespace std::literals::chrono_literals; //等待时间的操作可以先using一个命名空间，为 1s 提供作用域
    while (is_Finished) {
        std::cout << "hello" << std::endl;
        std::this_thread::sleep_for(1s);    //等待一秒
    }
}

int main() {
    std::thread worker(DoWork); //开启多线程操作

    std::cin.get(); //此时工作线程在疯狂循环打印，而主线程此时被cin.get()阻塞
    is_Finished = true;// 让worker线程终止的条件，如果按下回车，则会修改该值，间接影响到另一个线程的工作。

    worker.join();  //join:等待工作线程结束后，才会执行接下来的操作

    std::cin.get();
}
```

如果是正常情况，`DoWork`应该会一直循环下去，但因为这里是多线程，所以可以在另一个线程中修改工作线程的变量，来停止该线程的循环。 **多线程对于加速程序是十分有用的，线程的主要目的就是优化。**

## C++的计时

https://en.cppreference.com/w/cpp/chrono文档链接

**作用**:

计时的使用很重要。在逐渐开始集成更多复杂的特性时，如果编写性能良好的代码时，需要用到计时来看到差异。

**利用chrono类计时**：

1.包含头文件**`#include`** 2.获取当前时间：

```cpp
std::chrono::time_point<std::chrono::steady_clock> start = std::chrono::high_resolution_clock::now();
//或者，使用auto关键字
auto  start = std::chrono::high_resolution_clock::now();
auto  end = std::chrono::high_resolution_clock::now();
----------------------------------------------------------
//实例
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    //literals：文字
    using namespace std::literals::chrono_literals; //有了这个，才能用下面1s中的's'
    auto start = std::chrono::high_resolution_clock::now(); //记录当前时间
    std::this_thread::sleep_for(1s);    //休眠1s，实际会比1s大。函数本身有开销。
    auto end = std::chrono::high_resolution_clock::now();   //记录当前时间
    std::chrono::duration<float> duration = end - start;    //也可以写成 auto duration = end - start; 
    std::cout << duration.count() << "s" << std::endl;
    return 0;
}
```

3.获得时间差：

```cpp
std::chrono::duration<float> duration = end - start;
//或者
auto duration = end - start;
```

> 注意：在**自定义计时器类的构造函数、析构函数**中，**不要使用auto关键字**，应该在计时器类的构造函数、析构函数**前**定义start、end、duration变量。

```cpp
struct Timer   //写一个计时器类。
{
    std::chrono::time_point<std::chrono::steady_clock> start, end;
    std::chrono::duration<float> duration;

    Timer()
    {
        start = std::chrono::steady_clock::now(); //如果使用auto关键字会出现警告
    }

    ~Timer()
    {
        end = std::chrono::steady_clock::now();
        duration = end - start;

        float ms = duration.count() * 1000;
        std::cout << "Timer took " << ms << " ms" << std::endl;
    }
};
void Function()
{
    Timer timer;
    for (int i = 0; i < 100; i++)
        std::cout << "Hello\n"; //相比于std::endl更快
}

int main()
{
    Function();
}
```

## C++多维数组

数组优化的一个方法：**把二维数组转化成一维数组来进行存储。**

```cpp
//代码参考来源：https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno64.cpp
#include <iostream>
#include <array>

int main()
{

    // 要知道，这样处理数组的数组，会造成内存碎片的问题
    // 我们创建了5个单独的缓冲区，每个缓冲区有5个整数，他们会被分配到内存的随机(空闲)位置
    // 在大量调用时，很可能造成cache miss，损失性能

    int *array = new int[5];
    int **a2d = new int *[5]; // 5个int指针
    for (int i = 0; i < 5; i++)
        a2d[i] = new int[5]; // allocate the memory

    for (int y = 0; y < 5; y++)
    {
        for (int x = 0; x < 5; x++)
        {
            a2d[y][x] = 2;
        }
    }

    // int ***a3d = new int **[5]; // 5个int指针的指针   三维数组
    // for (int i = 0; i < 5; i++)
    // {
    //     a3d[i] = new int *[5];
    //     for (int j = 0; j < 5; j++)
    //     {
    //         // int **ptr = a3d[i];
    //         // ptr[j] = new int[5];
    //         a3d[i][j] = new int[5];
    //     }
    // }

    for (int i = 0; i < 5; i++) // 需要先释放真正的多维数组
        delete[] a2d[i];
    delete[] a2d;
    // 这只会释放5个指针的内存，而后面分配的内存由于丢失掉了这些指针，
    // 也无法释放了，这就造成了内存泄漏

    int *array = new int[6 * 5];  //二维
    // for (int i = 0; i < 6 * 5; i++)
    // {
    //     array[i] = 2;
    // }
    for (int y = 0; y < 5; y++)   //数组优化，将二维数组转化为一维数组
    {
        for (int x = 0; x < 6; x++)
        {
            array[y * 5 + x] = 2;
        }
    }

    std::cin.get();
}
```

> B评论区的一个讨论问题：

```cpp
#include<iostream>
#include<chrono>

struct Timer   //写一个计时器类。
{
    std::chrono::time_point<std::chrono::steady_clock> start, end;
    std::chrono::duration<float> duration;

    Timer()
    {
        start = std::chrono::steady_clock::now(); //如果使用auto关键字会出现警告
    }

    ~Timer()
    {
        end = std::chrono::steady_clock::now();
        duration = end - start;

        float ms = duration.count() * 1000;
        std::cout << "Timer took " << ms << " ms" << std::endl;
    }
};
struct Rgb
{
    int r;
    int g;
    int b;
};

#define M 8000
#define N 5000

void draw()
{
    Timer timer;
    Rgb* a = new Rgb[M * N];
    for (int i = 0; i < M; i++)
    {
        for (int j = 0; j < N; j++)
        {
            a[i + j * M] = { 1,2,3 };
        }
    }
    //delete[] a;
}

//void draw()
//{
//  Timer timer;
//  Rgb* a = new Rgb[M * N];
//  for (int j = 0; j < N; j++)
//  {
//      for (int i = 0; i < M; i++)
//      {
//          a[j + i * N] = { 1,2,3 };
//      }
//  }
//  //delete[] a;
//}

void draw2()
{
    Timer timer;
    Rgb** a = new Rgb * [M];
    for (int i = 0; i < M; i++)
    {
        a[i] = new Rgb[N];
        for (int j = 0; j < N; j++)
        {
            a[i][j] = { 1,2,3 };
        }
        //delete[] a[i];   //这一句很神奇，加上后在release模式下，速度快5倍
    }
    //delete[] a;
}

int main()
{
    draw();
    draw2();
}
```

> 结论与Cherno的完全相反，二维数组比一维在debug与release下，均快1倍，如果在二维数组方式下，加上一句delete【】，再快将近5倍。
> 应该是你draw里面赋值的时候有问题。你这个两层循环内层是j,但j又是列指标，所以相当于本来完全连续的赋值变成每次赋值都要跑隔M的地方才能赋所以会变得很慢。
> 这里分配必然是慢的，因为是间隔分配，了解内存分配都知道越分散性能越差。Cherno说的快应该是读取的时候，读取的时候因为少了间接性（多层指针指向），读取性能要比多维高很多，修改性能应该也高很多。 另这里不应该用 【i + j * M】 而是应该用 【j + i * N】这样性能也会好很多，因为这是连续分配。
> release模式会优化代码，不一定会执行全部。 另外按升序遍历，索引应该是i*N+j，因为j走一遍，i才加1。连续的内存才能容易cache hit
> 我把样本数据扩大到5000*5000 之后 , release 下一维明显更快 , 而 debug 模式下二维更快一点

## C++内置的排序函数

文档链接https://en.cppreference.com/w/cpp/algorithm/sort

1.sort( vec.begin(), vec.end(), 谓语)

谓语可以设置排序的规则，谓语可以是内置函数，也可以是lambda表达式。

2.**默认**是从小到大排序

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};
    std::sort(values.begin(), values.end());  
    for (int value : values)
    std::cout << value << std::endl; // 1 2 3 4 5
    std::cin.get();
}
```

3.使用内置函数，添加头文件functional，使用std::greater函数，则会按照从大到小顺序排列。

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};             
    std::sort(values.begin(), values.end(),std::greater<int>()); 
    for (int value : values)
    std::cout << value << std::endl; // 5 4 3 2 1
    std::cin.get();
}
```

4.使用 lambda 进行灵活排序

```cpp
std::sort(values.begin(), values.end(), [](int a, int b)
    {
            return a < b;
    });
```

> 对于已定的传入参数的顺序`[](int a, int b)`，函数体中如果参数a在前面，则返回true，如果参数a在后面则返回false

```text
a < b //返回true，a排在前面。此时为升序排列（如果a小于b，那么a就排在b的前面）
a > b //返回true, a排在前面，此时为降序排列（如果a大于b，那么a就排在b的前面）
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};          

    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            return a < b;  // 如果a小于b，那么a就排在b的前面。 1 2 3 4 5
    });

    for (int value : values)
    std::cout << value << std::endl;

    std::cin.get();
}
```

5.如果把1排到最后

> 如果a==1，则把它移到后面去，即返回false，不希望它在b前。 如果b==1，我们希望a在前面，要返回true。

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};          
    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            if (a == 1)
                return false;
            if(b == 1)
                return true;
            return a < b;   //2 3 4 5 1
    });
    for (int value : values)
    std::cout << value << std::endl;
    std::cin.get();
}
```

## C++的类型双关(type punning)

1.将同一块内存的东西通过不同type的指针给取出来

> 把一个int型的内存，换成double去解释，当然**这样做很糟糕**，因为添加了四字节不属于原本自己的内存，只是作为演示。 原始方法：**（取地址，换成对应类型的指针，再解引用）**

```cpp
#include <iostream>
int main()
{
    int a = 50;
    double value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
//可以用引用，这样就可以避免拷贝成一个新的变量：（只是演示，这样做很糟糕）
#include <iostream>
int main()
{
    int a = 50;
    double& value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
```

2.把一个结构体转换成数组进行操作（? 还不理解）

```cpp
#include <iostream>
struct Entity
{
    int x, y;
};

int main()
{
    Entity e = {5, 8};
    int *position = (int *)&e;
    std::cout << position[0] << ", " << position[1] << std::endl;

    int y = *(int *)((char *)&e + 4);
    std::cout << y << std::endl;
}
```

## C++的联合体( union )

1.`union { };`，注意结尾有**分号**。

2.通常union是匿名使用的，但是匿名union不能含有成员函数

3.在可以使用**类型双关**的时候，使用union时，可读性更强 。

4.union的特点是**共用内存** 。可以像使用结构体或者类一样使用它们，也可以给它添加静态函数或者普通函数、方法等待。然而**不能使用虚方法**，还有其他一些限制。

```cpp
#include <iostream>
int main() {  
    union {  //匿名使用，不写名字
        float a;
        int b;
    };  
    a = 2.0f;  //共享内存，a被赋值了一个浮点数，整形的b也被复制了一个浮点数
    std::cout << a << '，' << b << std::endl;
    //输出： 2，107165123
    //原因：int b取了组成浮点数的内存，然后把它解释成一个整型（类型双关）
}
```

较实用的一个例子：

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
};

struct Vector4
{
    union // 不写名称，作为匿名使用
    {
        struct //第一个Union成员
        {
            float x, y, z, w;
        };
        struct // 第二个Union成员，与第一个成员共享内存
        {
            Vector2 a, b;//a和x，y的内存共享，b和z，w的内存共享
        };
    };
};

void PrintVector2(const Vector2 &vector)
{
    std::cout << vector.x << ", " << vector.y << std::endl;
}

int main()
{
    Vector4 vector = {1.0f, 2.0f, 3.0f, 4.0f};
    PrintVector2(vector.a);
    PrintVector2(vector.b);
    vector.z = 500;
    std::cout << "-----------------------" << std::endl;
    PrintVector2(vector.a);
    PrintVector2(vector.b);
}
//输出：
1，2
3，4
-----------------------
1，2
500，4
```

> 引自评论： union里的成员会共享内存，分配的大小是按最大成员的sizeof, 视频里有两个成员，也就是那两个结构体，改变其中一个另外一个里面对应的也会改变. 如果是这两个成员是结构体struct{ int a,b} 和 int k , 如果k=2 ; 对应 a也=2 ，b不变； union我觉得在这种情况下很好用，就是用不同的结构表示同样的数据 ，那么你可以按照获取和修改他们的方式来定义你的 union结构 很方便

## C++的虚析构函数

1.如果用基类指针来引用派生类对象，那么基类的析构函数必须是 virtual 的，否则 C++ 只会调用基类的析构函数，不会调用派生类的析构函数。

2.继承时，要养成的一个好习惯就是，**基类析构函数中，加上virtual。**

> 为什么要调用派生类析构函数？
> 若派生类有一个成员int数组在堆上分配东西，在构造函数中分配，在析构函数中删除。运行当前代码发现没有调用那个派生析构函数，但是它调用了派生类的构造函数。我们在构造函数中分配了一些内存，但是永远不会调用派生析构函数delete释放内存，因为析构函数没有被调用，永远不会删除堆分配数组，这就是所谓的内存泄漏。

```cpp
#include <iostream>

class Base
{
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; }
};

class Derived : public Base
{
public:
    Derived()
    {
        m_Array = new int[5];
        std::cout << "Derived Constructor\n";
    }
    ~Derived()
    {
        delete[] m_Array;
        std::cout << "Derived Destructor\n";
    }

private:
    int *m_Array;
};

int main()
{
    Base *base = new Base();
    delete base;
    std::cout << "-----------------" << std::endl;
    Derived *derived = new Derived();
    delete derived;
    std::cout << "-----------------" << std::endl;
    Base *poly = new Derived();
    delete poly; // 基类析构函数中如果不加virtual，则此处会造成内存泄漏
    // Base Constructor
    // Base Destructor
    // -----------------
    // Base Constructor
    // Derived Constructor
    // Derived Destructor
    // Base Destructor
    // -----------------
    // Base Constructor
    // Derived Constructor
    // Derived Destructor //基类析构函数中如果不加virtual，子类的虚构函数不会被调用
    // Base Destructor
}
```

> 引自B站评论区：
> 此处这位外国友人说错了，定义基类的虚析构并不是什么相加，而是：**基类中只要定义了虚析构（且只能在基类中定义虚析构，子类析构才是虚析构，如果在二级子类中定义虚析构，编译器不认，且virtual失效），在编译器角度来讲，那么由此基类派生出的所有子类地析构均为对基类的虚析构的重写，当多态发生时，用父类引用，引用子类实例时，此时的虚指针保存的子类虚表的地址，该函数指针数组中的第一元素永远留给虚析构函数指针。所以当delete 父类引用时，即第一个调用子类虚表中的子类重写的虚析构函数此为第一阶段。然后进入第二阶段：（二阶段纯为内存释放而触发的逐级析构与虚析构就没有半毛钱关系了）而当子类发生析构时，子类内存开始释放，因内存包涵关系，触发父类析构执行，层层向上递进，至到子类所包涵的所有内存释放完成。**

## C++的类型转换

> cast 分为 `static_cast` `dynamic_cast` `reinterpret_cast` `const_cast`

### static_cast

static_cast用于进行比较“自然”和低风险的转换，就是c语言风格的强制转换。如整型和浮点型、字符型之间的互相转换,不能用于指针类型的强制转换

> 任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast。

```cpp
double dPi = 3.1415926;
int num = static_cast<int>(dPi);  //num的值为3
double d = 1.1;
void *p = &d;
double *dp = static_cast<double *>(p);
```

### reinterpret_cast

reinterpret_cast 用于进行各种不同类型的指针之间强制转换。

> 通常为运算对象的位模式提供较低层次上的重新解释。危险，不推荐。

```cpp
int *ip;
char *pc = reinterpret_cast<char *>(ip);
```

### const_cast

const_cast 添加或者移除const性质

> 用于改变运算对象的**底层const**。常用于有函数重载的上下文中。
> **顶层const：**表示**对象是常量**。举例int *const p1 = &i; //指针p1本身是一个常量，不能改变p1的值，p1是顶层const。
> **底层const：**与指针和引用等复合类型部分有关。举例：const int *p2 = &ci; //指针所指的对象是一个常量，允许改变p2的值，但不允许通过p2改变ci的值，p2是底层const

```cpp
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}

//上面函数返回的是常量string引用，当需要返回一个非常量string引用时，可以增加下面这个函数
string &shorterString(string &s1, string &s2) //函数重载
{
    auto &r = shorterString(const_cast<const string &>(s1), 
                            const_cast<const string &>(s2));
    return const_cast<string &>(r);
}
```

### dynamic_cast

dynamic_cast 不检查转换安全性，仅运行时检查，如果不能转换，返回NULL。

> 支持运行时类型识别(run-time type identification,RTTI)。
> 适用于以下情况：我们想使用基类对象的指针或引用执行某个派生类操作并且该操作不是虚函数。一般来说，只要有可能我们应该尽量使用虚函数，使用RTTI运算符有潜在风险，程序员必须清楚知道转换的目标类型并且必须检查类型转换是否被成功执行。

```cpp
//https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno69.cpp
#include <iostream>
class Base
{
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; }
};

class Derived : public Base
{
public:
    Derived()
    {
        m_Array = new int[5];
        std::cout << "Derived Constructor\n";
    }
    ~Derived()
    {
        delete[] m_Array;
        std::cout << "Derived Destructor\n";
    }

private:
    int *m_Array;
};

class AnotherClass : public Base
{
public:
    AnotherClass(){};
    ~AnotherClass(){};
};

int main()
{
    // double value = 5.25;
    // // int a = value;
    // // int a = (int)value;
    // double a = (int)value + 5.3; // 10.3 // C style cast here

    // double s = static_cast<int>(value) + 5.3; // C++ style cast here

    // std::cout << a << std::endl;
    // std::cout << s << std::endl;

    Derived *derived = new Derived();

    Base *base = derived;

    // AnotherClass *ac = static_cast<AnotherClass*>(base);  //NULL
    Derived *ac = dynamic_cast<Derived *>(base);

    delete derived;
}
```

## C++的std::bind

**概述**
std::bind的头文件是 <functional>，它是一个函数适配器，接受一个可调用对象（callable object），生成一个新的可调用对象来“适应”原对象的参数列表。
函数原型
std::bind函数有两种函数原型，定义如下：

```cpp
template< class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );

template< class R, class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );
```

std::bind返回一个基于f的函数对象，其参数被绑定到args上。
f的参数要么被绑定到值，要么被绑定到placeholders（占位符，如_1, _2, ..., _n）。

std::bind将可调用对象与其参数一起进行绑定，绑定后的结果可以使用std::function保存。std::bind主要有以下两个作用：

1. 将可调用对象和其参数绑定成一个仿函数；
2. 只绑定部分参数，减少可调用对象传入的参数。

1 std::bind绑定普通函数

```cpp
double callableFunc (double x, double y) {return x/y;}
auto NewCallable = std::bind (callableFunc, std::placeholders::_1,2);  
std::cout << NewCallable (10) << '\n';                       
```

- bind的第一个参数是函数名，普通函数做实参时，会隐式转换成函数指针。因此std::bind(callableFunc,_1,2)等价于std::bind (&callableFunc,\_1,2)；

- \_1表示占位符，位于<functional>中，std::placeholders::\_1；
- 第一个参数被占位符占用，表示这个参数以调用时传入的参数为准，在这里调用NewCallable时，给它传入了10，其实就想到于调用callableFunc(10,2);

2 std::bind绑定一个成员函数

```cpp
class Base
{
public:
    void display_sum(int a1, int a2)
    {
        std::cout << a1 + a2 << '\n';
    }
    int m_data = 30;
};
int main() 
{
    Base base;
    auto newiFunc = std::bind(&Base::display_sum, &base, 100, std::placeholders::_1);
    newiFunc(20); // should out put 120. 
}
```

- bind绑定类成员函数时，第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址。
- 必须显式地指定&Base::diplay_sum，因为编译器不会将对象的成员函数隐式转换成函数指针，所以必须在Base::display_sum前添加&；
- 使用对象成员函数的指针时，必须要知道该指针属于哪个对象，因此第二个参数为对象的地址 &base；

3 绑定一个引用参数
默认情况下，bind的那些不是占位符的参数被拷贝到bind返回的可调用对象中。但是，与lambda类似，有时对有些绑定的参数希望以引用的方式传递，或是要绑定参数的类型无法拷贝。

```cpp
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>
#include <sstream>
using namespace std::placeholders;
using namespace std;

ostream & printInfo(ostream &os, const string& s, char c)
{
    os << s << c;
    return os;
}

int main()
{
    vector<string> words{"welcome", "to", "C++11"};
    ostringstream os;
    char c = ' ';
    for_each(words.begin(), words.end(), 
                   [&os, c](const string & s){os << s << c;} );
    cout << os.str() << endl;

    ostringstream os1;
    // ostream不能拷贝，若希望传递给bind一个对象，
    // 而不拷贝它，就必须使用标准库提供的ref函数
    for_each(words.begin(), words.end(),
                   bind(printInfo, ref(os1), _1, c));
    cout << os1.str() << endl;
}
```

**1. `vector<string> words{"welcome", "to", "C++11"};`**

定义了一个 `vector<string>`，存储了 `{"welcome", "to", "C++11"}` 这三个字符串。

------

**2. `ostringstream os;`**

`std::ostringstream` 是 C++ 标准库提供的 **字符串流类**，它继承自 `std::ostream`，用于向字符串对象写入数据。可以把它看作是一个可以像 `std::cout` 一样写入数据的字符串缓冲区，数据最终会存储到 `os.str()` 里。

**作用**

它主要用于**将数据格式化存储到字符串**，而不是直接输出到控制台。最终可以用 `os.str()` 获取字符串内容。

------

**3. `for_each(words.begin(), words.end(), [&os, c](const string & s){os << s << c;});`**

- `for_each()` 是 `<algorithm>` 头文件中的算法，它的作用是：
  - **遍历整个 `vector`**，对每个元素执行 `lambda` 表达式中的逻辑。
- `lambda` 表达式 ` [&os, c](const string & s){ os << s << c; }` 解释：
  - `[&os, c]`：表示 `os` 以引用捕获（**可以修改 os**），`c` 以值捕获（**不会修改原始变量**）。
  - `(const string & s)`：遍历 `words` 中的每个 `string`，赋值给 `s`。
  - `{ os << s << c; }`：将 `s` 和 `c`（即空格字符 `' '`）追加到 `os` 里面。

**执行过程**

等价于：

```
cpp复制编辑os << "welcome" << ' ';
os << "to" << ' ';
os << "C++11" << ' ';
```

最终 `os.str()` 里存的是：

```
arduino

复制编辑
"welcome to C++11 "
```

然后 `cout << os.str() << endl;` 输出：

```
css

复制编辑
welcome to C++11 
```

------

**4. `ostringstream os1;`**

同 `os`，用于存储字符串流。

------

**5. `for_each(words.begin(), words.end(), bind(printInfo, ref(os1), _1, c));`**

**作用**

- `bind(printInfo, ref(os1), _1, c)` **等价于** 一个函数 `printInfo(os1, s, c)`，其中：
  - `printInfo` 是 `ostream& printInfo(ostream &os, const string& s, char c)`，负责格式化输出。
  - `ref(os1)`：`std::ref` **包装 os1**，确保 `bind` 传递**引用**（因为 `ostream` 不能拷贝）。
  - `_1`：占位符，代表 `for_each` 迭代的 `vector` 元素。
  - `c`：空格字符 `' '`，作为 `printInfo` 的第三个参数。

**执行过程**

等价于：

```
cpp复制编辑printInfo(os1, "welcome", ' '); // os1 << "welcome" << ' ';
printInfo(os1, "to", ' ');      // os1 << "to" << ' ';
printInfo(os1, "C++11", ' ');   // os1 << "C++11" << ' ';
```

最终 `os1.str()` 里存的是：

```
arduino


复制编辑
"welcome to C++11 "
```

然后 `cout << os1.str() << endl;` 输出：

```
css


复制编辑
welcome to C++11 
```

------

**总结**

**`std::ostringstream` 是什么？**

`std::ostringstream` 是一个**字符串流类**，用于**像 `std::cout` 一样向字符串写入数据**，最终可以用 `.str()` 取出字符串。

**两个 `for_each()` 在做什么？**

1. **第一个 `for_each`**：使用 **lambda 表达式** 遍历 `words`，逐个将字符串追加到 `os` 中。
2. **第二个 `for_each`**：使用 **`std::bind`** 绑定 `printInfo` 函数，实现同样的操作，但更加通用（`printInfo` 也可以复用）。

最终，两种方法的输出效果一致：

```
welcome to C++11 
welcome to C++11 
```

但 `bind` 方式可以让 `printInfo` 作为通用函数复用，而 `lambda` 方式更直观，适用于简单场景。