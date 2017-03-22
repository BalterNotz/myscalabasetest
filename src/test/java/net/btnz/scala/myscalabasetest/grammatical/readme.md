**Scala 语法规范**

#词法

Scala程序使用Unicode的基本多文种平面字符集；目前不支持Unicode中的增补的字符。本章定义的Scala词法的两种模式：Scala模式与XML模式。默认使用Scala模式，常量字符'c'指ASCII段\u0000-\u007F.

在Scala中，十六进制Unicode转义字符会被对应的Unicode字符替换。

UnicodeEscape ::= \{\\}u{u} hexDigit hexDigit hexDigit hexDigit

hexDigit      ::= '0'|...|'9'|'A'|...|'F'||'a'|...|'f'|

##标识符

有三种方式可以构造一个标识符：

1. 字母 + 任意字母和数字 或'_'
2. 算符字符 +　任意算符字符
3. 由反引号'`'括起来的任意字符

按惯例，标识符符合最长匹配原则。

保留字：

    abstract case catch class def
    do else extends false final
    finally for forSome if implicit
    import lazy match new null
    object  override package private protected
    requires return sealed super this
    throw trait try true type
    val var while with yield
    _ : = => <- <: <% >: # @

Unicode 算符\u21D2 '=>' 和 \u2190 '<-'以及它们的ASCII对应`=>`也是保留字

示例：Thread.`yield`()，反引号括起来的字符是那些Scala中的保留字的Java中标识符的一个方法，如果直接写Thread.yield()是非法的，因为yield是保留字，但加上反引号是允许的。

##换行字符

semi ::= ';' | nl{nl}

Scala是一个基于行的语言。分号和换行均可作为语句的结束。如果换行满足以下三个条件则会被认为是一个特殊符号'nl':

1. 换行之前的符号是一个语句的结束
2. 换行之后的符号是一个语句的开始
3. 符号处在一个允许多语句的区域中

可以作为语句开始的符号是除了以下分隔符及保留字之外的所有Scala符号：

    catch else extends finally forSome match requires
    with yield , . ; : _ = => <- <: <% >: # [ ) ] }

符号case只有在class或者object符号之前才可以作为语句的开始。

多行语句许可的条件：。。。。。

##字面值

字面值包括整数，浮点数，字符，布尔值，记号，字符串。这些字面值的语法均和Java中的字面值一致。

##字白与注释

符号可由空白字符或注释分隔开。注释有两种格式：

单行注释是由//开始直到行尾的字符序列

多行注释是在/*和*/之间的字符序列。多行注释可以嵌套，但是必须合理的嵌套。

##XML模式

为了允许包含XML片段字面值。当遇到左尖括号'<'时，在以下情况下词法分析就会从Scala模式切换到XML模式：'<'前必须是空白，左括号或者左大括号，'<'后必须跟一个XML命名。

语法:

    (whitespace | '(' | '{' } '<' (XNameStart | '!' | '?')
    XNameStart ::= '_' | BaseChar | Ideographic (和W3C XML一样，但没有':')

扫描器遇到以下条件之一时将会从XML模式切换到Scala模式：

由'<'开始的XML表达式或模式已被成功解析。

解析器遇到一个内嵌的Scala表达式或模式强制扫描器返回正常模式，直到Scala表达式或模式被成功解析。在这种情况下，由于代码和XML片段可以嵌套，解析器将会用一个堆栈来储存嵌套的XML和Scala表达式。

注意在XML模式中不会生成Scala的符号，注释会解析为文本。

示例：

    val b = <book>
                <title>The Scala Language Specification</title>
                <version>{scalaBook.version}</version>
                <authors>{scalaBook.authors.mkList("",",","")}</authors>
            <book>

#标识符，命名和域

在Scala中，命名用来表示类型，值，方法以及类，这些统称为实体。命名在局部定义与声明，继承，import子句，package子句中存在，这些可以统称为绑定。

绑定有优先级，定义（局部或继承）有最高的优先级，然后是显示import，然后是通配符import，然后是包成员，是最低的优先级。

有两种不同的命名空间，一个是类型，一个是术语。同样的命名可以表示类型或术语，这要看命名应用所在的上下文。

绑定有一个域，在此域中用单一命名定义的实体可以用一个简单名称来访问。域可以嵌套。内部域中的绑定将会遮盖同一域中低优先级的绑定，或者外部域中低优先级或同优先级的绑定。注意遮盖只是偏序关系。

#类型

语法：

    Type    ::= InfixType '=>' Type
            | '('['=>' Type] ')' '=>' Type
            | InfixType [ExistentialClause]
    ExistentialClause   ::= 'forSome' '{' ExistentialDC {semi ExistentialDcl} '}'
    ExistentialDcl  ::= 'type' TypeDcl
                    | 'val' ValDcl
    InfixType   ::= CompoundType {id [nl] CompoundType}
    CompoundType    ::= AnnotType {'with' AnnotType}[Refinement]
                    | Refinement
    AnnotType   ::= SimpleType {Annotation}
    SimpleType  ::= SimpleType TypeArgs
                | SimpleType '#' id
                | StableId
                | Path '.' 'type'
                | '(' Types [','] ')'
    TypeArgs    ::= '[' Types ']'
    Types   ::= Type {',' Type}

一价类型和类型构造器（用类型的参数构造类型）是有区别的。一阶类型的一个了集是值类型，表示（一阶）值的集合。值类型可以是具体的或者抽象的。

非值类型描述了那些不是值的标识符的属性。例如，一个类型构造器并不指明值的类型。然而，当一个类型构造器应用到正确的类型参数上时，就会产生一个可能是值类型的一阶类型。

在Scala中，非值类型被间接表述。例：写下一个方法签名来描述一个方法类型，虽然通过它可以得到对应的函数类型，但它本身并不是一个真正的类型。类型构造器是另外一个例子，比如我们可以写type Swap[m[_,_],a,b] = m[b,a]，但是并没有定义直接给出对应的匿名类型函数的语法。

##路径

语法：

    Path            ::= StableId
                    : [id '.'] this
    StableId        ::= id
                    | Path '.' id
                    | [id '.'] 'supper' [ClassQualifier] '.' id
    ClassQualifier  ::= '[' id ']'

路径不是类型本身，但是它们可以是命名类型的一部分，这个功能是Scala类型系统的一个核心角色。

一个路径是下面定义中的一个：

1. 空路径 （不能在程序中显式地写出来）
2. C.this， C是一个类的引用。当在C引用范围内时，路径this是C.this的简写。
3. p.x，p是一个路径，x是p的一个稳定成员。稳定成员是由对象定义或者稳定类型的值定义引入的包或者成员
4. C.super.x或C.super[M].x，C是一个类的引用，x是C的超类或指定的父类M的稳定成员的引用。前缀super是C.super的简写，C是包含引用范围的类的名称。

一个稳定的标识符是由标识符结束的一个路径。

##值类型

Scala中的每一个值类型都有一个以下格式的类型。

###单例类型

语法：

    SimpleType  ::= Path '.' type

单例类型具有p.type的形式，p是一个路径，指向一个期望与scala.AnyRef一致的值。类型指一组为null的或由p表示的值。

一个稳定类型指要么是一个单例类型，要么是特征scala.Singleton的子类型

###类型映射

语法：

    SimpleType  ::= SimpleType '#' id

类型映射T#x指类型T的类型成员x。如果x指向抽象类型成员，那么T必须是一个稳定类型。

###类型指示

语法：

    SimpleType  ::= StableId

类型指示指一个命名值类型。它可以是简单的或限定的。所有的这些类型指示都是类型映射的简写。

绑定在某类，对象或包C上的非限定的类型名t是C.this.type#t的简写，除非类型t是类型模式的一部分。后者中的t是C#t的简写。如果t没有被绑定在某类，对象或包上，那么t就是e.type#t的简写。

一个限定的类型只是具有p.t的形式，p是一个路径，t是一个类型名。这个类型指示等价于类型映射p.type#t。

示例：以下是一些类型指示以及扩展。我们假定一个本地类型参数t，一个值maintable具有一个类型成员Node，以及一个标准类scala.Int

    t           e.type#t
    Int         scala.type#Int
    scala.Int   scala.type#Int
    data.maintable.Node data.maintable.type#Node

###参数化类型

语法：

    SimpleType  ::= SimpleType TypeArgs
    TypeArgs    ::= '[' Types ']'

参数化类型T[U1,...,Un]包括类型指示T以及类型参数U1,...,Un，n >= 1.T必须指向一个具有个参数类型a1,...,an的参数构造方法。

类型参数具有下界L1,...,Ln和上界U1,...,U2。参数化类型必须保证每个参数与其边界一致。

示例：

    class TreeMap[A <: Comparable[A], B] {...}
    class List[A] {...}
    class I extends Comparable [I] {...}

以下是正确的参数化类型：

    TreeMap[I, String]
    List[I]
    List[List[Boolean]]

###元组类型

语法:

    SimpleType  ::= '(' Types [','] ')'

元组类型(T1,...,Tn)是类scala.Tuplen[T1,...,Tn] (n >= 2)的别名形式。此类型可以在结尾处有个额外的逗号，例：(T1,...,Tn,)。

元组类是case类，其字段可以用选择器_1,...,_n来访问。在对应的Product特征中有他们的抽象函数。这些元组类以及Product特征都是标准Scala类库的一部分，其形式如下：

    case class Tuplen[+T1,...,+Tn] (_1:T1,...,_n:Tn) extends Productn[T1,...,Tn]{}
    trait Productn[+T1,...,+Tn]{
        override def arity = n
        def _1 : T1
        ...
        def _n : Tn
    }

###标注类型

语法：

    AnnotType   ::= SimpleType {Annotation}

标注类型T a1,...,an就是给类型T加上类型标注a1,...,an.

###复合类型

语法：

    CompoundType    ::= AnnotType {'with' AnnotType} [Refinement]
                    | Refinement
    Refinement      ::= [nl] '{' RefineStat {semi RefineStat} '}'
    RefineStat      ::= Dcl
                    | 'type' TypeDef
                    |

复合类型T1 with ... with Tn {R}指一个拥有T1,...,Tn类型中的成员以及修饰{R}的对象。如果对象中有声明或定义覆盖了成分类型T1,...,Tn中的声明或定义，就会应用通常的覆盖规则；否则这个声明或定义就将是所谓的“结构化的”。

标例：以下是如何声明以及使用参数类型包含结构化声明修饰的函数。

    case class Bird (val name : String) extends Object {
        def fly(height: Int) = ...
        ...
    }
    case class Plane (val callsign: String) extends Object {
        def fly(height: Int) = ...
        ...
    }
    def takeoff(
            runway: Int,
            r: {val callsign: String; def fly(height: Int)}) = {
        tower.print(r.callsign + " requests take-off on runway " + runway)
        tower.read(r.callsign + " is clear for take-off")
        r.fly(1000)
    }
    val bird = new Bird("Polly the parrot") {val callsign = name}
    val a380 = new Plant("TZ-987")
    takeoff(42, bird)
    takeoff(89, a380)

虽然Bird和Plane没有除了Object之外的任何父类，用结构化声明修饰的函数takeoff的参数r却可以接受任何声明了值callsign以及函数fly的对象。

###中缀类型

语法：

    InfixType   ::= CompoundType {id [nl] CompoundType}

中缀类型T1 op T2由一个中缀算符op应用到两个操作数T1和T2上得来。这个类型等价于类型应用op[T1,T2]。中缀算符可以是除*之外的任意的标识符。

###函数类型

语法：

    Type    ::= InfixType '=>' Type
            | '(' ['=>' Type] ')' '=>' Type

类型(T1,...,Tn) => U表示那些参数类型为T1,...,Tn，并产生一个类型为U的结果的函数。

函数类型是定义了apply函数的类类型的简写。比如n型函数类型(T1,...,Tn) => U就是类Funtionn[T1,...,Tn,U]的简写。Scala库中定义了n为0至9的这些类类型，如下所示：

    package scala
    trait Functionn[-T1,...,-Tn,+R] {
        def apply(x1,T1,...,xn:Tn): R
        override def toString = "<function>"
    }

因此，函数类型与结果类型是协变的，与参数类型是逆变的。

传名函数类型(=>T)=>U是类类型ByNameFunction[T,U]的简写形式，定义如下：

    package scala
    trait ByNameFunction[-T, +R] {
        def apply(x: => T): R
        override def toString = "<function>"
    }

###3.2.10.即存类型

语法：

    Type    ::= InfixType ExistentialClauses
    ExistentialClauses  ::= 'forSome' '{' ExistentialDcl {semi ExistentialDcl} '}'
    ExistentialDcl  ::= 'type' TypeDcl
                    | 'val' ValDcl

即存类型具有T forSome {Q}的形式，Q是一个类型声明的序列。

T forSome {Q}的类的实例就是类oT， o是t1,...,tn上的迭代，对于每一个i都有oLi<:oti<:oUi。既存类型T forSome {Q}的值的集合就是所有其类型实例值的集合的合集。

T forSome {Q}的斯科伦化是一个类实例oT,o是[t'1/t1,...,t'n/tn]上的迭代，每个t'i是介于oLi和oUi间的新的抽象类型。

简化规则：既存类型遵循以下四个等效原则：

1. 即存类型中的多个for子句可以合并。例 T forSome {Q} forSome {Q'}等价于T forSome {Q; Q'}
2. 未使用的限定可以去掉。例：T forSome {Q;Q'}，但是Q'中定义的类型没有被T或Q引用，那么该表达式可以等价为T forSome {Q}
3. 空的限定可以丢弃。例：T forSome {} 等价于 T
4. 一个既类型T forSome {Q}，Q中包含一个子句type t[tps] >: L <: U等价于类型T' forSome {Q}，T'是将T中所有t的协变量替换为U并且将T中所有的t的逆变量替换为L的结果。

在值上的既存量化

既存类型的占位符语法

语法：

    WildcardType    ::= '_' TypeBounds

###3.2.11.Predef中定义的原始类型

每一个Scala程序都默认import一个Predef对象，该对象定义了一些原始类型做为类型类型的别名。数值类型和布尔型有标准的Scala类。String类型与宿主系统的String类型一致。在Java环境下，Predef包括以下类型绑定：

    type byte       = scala.Byte
    type short      = scala.Short
    type char       = scala.Char
    type int        = scala.Int
    type long       = scala.Long
    type float      = scala.Float
    type double     = scala.Double
    type Boolean    = scala.Boolean
    type String     = java.lang.String

##3.3.非值类型

以下类型并不表示值的集合，也并不显式地出现在程序中。它们只以已定义标识符的内部类型而引入。

###3.3.1.方法类型

方法类型在内部表示为(Ts)U, (Ts)是一个类型序列(T1,...,Tn) n >= 0, U是一个（值或者方法）类型。这个类型表示一个命名的方法，其参数的类型是T1,...,Tn，返回结果的类型是U。

方法类型是右结合的，(Ts1)(Ts2)U被处理的方式是(Ts1)((Ts2)U)。

示例：

    def a: Int
    def b (x: Int): Boolean
    def c (x: Int) (y: String, z: String): String

###3.3.2.多态方法类型

多态方法类型在内部表示为[tps]T, [tps]是类型参数部分[a1 >: L1 <: U1,...,an >: Ln <: Un], n >= 0, T是一个（值或方法）类型。该类型表示一个以S1,...,Sn为类型参量并产生类型为T的结果的命名方法，参数类型S1,...,Sn与下界L1,...,Ln和上界U1,...,Un一致。

示例：

    def empty[A]: List[A]
    def union[A <: Comparable[A]] (x: Set[A], xs: Set[A]): Set[A]
    产生如下类型:
    empty: [A >: Nothing <: Any] List[A]
    union: [A >: Nothing <: Comparable[A]] (x: Set[A], xs: Set[A]) Set[A]

###3.3.3.类型构造器

类型构造器在内部的表示方法类似于多态方法类型。[+/- a1 >: L1 <: U1,...,+/-an >: Ln <: Un] T表示一个期望是类型构造器参数或有对应类型参数子句的抽象类型构造器绑定的类型。

示例:以下是类Iterable[+X]的片段：

    trait Iterable[+X] {
        def flatMap[newType[+X]<:Iterabe[X], S](f: X => newType[S]): newType[S]
    }
    从概念上来讲，类型构造器Iterable是匿名类型[+X] Iterable[X]的名称，在flatMap中传递给newType类型构造器参数。

##3.4.基本类型和成员定义

类成员的类型取决于成员被引用的方式。主要有三个概念：

1. 类型 T 的基本类型集合
2. 从前缀类型 S 中可见的类 C 中的类型 T
3. 类型 T 的成员绑定集合

##3.5.类型间的关系

1. 类型恒等   T 恒等 U T和U可以在任何情况下互相替换
2. 一致      T <: U 类型T与类型U一致

#4.基本声明与定义

##4.3.类型声明与类型别名

##4.4.类型参数

##4.5.差异标注

##4.6.函数声明与定义

###4.6.1.叫名参数

语法：

    ParamType   ::= '=>' Type

值参数类型可以有前缀 =>，例如： x: => T。这样一个参数的类型就是无参方法类型 => T。这表明对应的参数并没有在函数应用处求值，而是在函数中每次使用时才求值。也就是该参数以叫名的方式求值。

###4.6.2.重复参数

语法：

    ParamType   ::= Type '*'

###4.6.3.过程

语法：

    FunDcl  ::= FunSig
    FunDef  ::= FunSig[nl] '{'Block'}'

过程有特殊语法，例如，返回Unit值{}的函数。过程声明只是返回类型被忽略的函数声明。返回类型自动定义为Unit类型。例如def f(ps)等价于def f(ps):Unit。

###4.6.4.方法返回类型推断

##4.7.Import子句

#5.类与对象

语法：

    TmplDef ::= ['case'] 'class' ClassDef
            | ['case'] 'object' ObjectDef
            | 'trait' TraitDef

##5.1.模板

    ClassTemplate   ::= [EarlyDefs] ClassParents [TemplateBody]
    TraitTemplate   ::= [EarlyDefs] TraitParents [TemplateBody]
    ClassParents    ::= Constr {'with' AnnotType}
    TraitParents    ::= AnnotType {'with' AnnotType}
    TemplateBody    ::= [nl] '{' [SelfType] TemplateStat {semi TemplateStat} '}'
    SelfType        ::= id [':' Type] '=>'
                    | this ':' Type '=>'
###5.1.1.构造器调用

###5.1.2.类的线性化

###5.1.3.类成员

###5.1.4.覆盖

###5.1.5继承闭包

###5.1.6前置定义

##5.2.修饰符

##5.3.类定义

###5.3.2.Case类

语法：

    TmplDef ::= 'case' 'class' ClassDef

如果一个类定义有case前缀，那么该类就被称为case类。

case类中第一个参数段中的正式参数称为元素，对它们将作特殊处理。首先，该参数的值可以扩展为构造器模式的一个字段。其次，该参数默认添加 val 前缀，除非该参数已经有 var或val 修饰符。然后会针对该参数生成一个访问定义。

case类定义c[tps](ps1)...(psn)有类参数tps和值参数ps，会自动生成一个扩展对象，定义如下：

    object c {
        def apply[tps](ps1)...(psn): c[tps] = new c[Ts](xs1)...(xsn)
        def unapply[tps](x: c[tps]) = scala.Some(x.xs11,...,x.xs1k)
    }

###5.3.3.特征

语法：

    TmplDef             ::= 'trait' TraitDef
    TraitDef            ::= id [TypeParamClause] TraitTemplateOpt
    TraitTemplateOpt    ::= 'extend' TraitTemplate
                        | [['extends'] TemplateBody]

特征是那些要以混入的形式加入到其他类中的类。与通常的类不同，特征不能有构造器参数。且也不能有构造器参数传递给其父类。这些都是没必要的，因为特征在父类初始化后才进行初始化。

##5.4.对象定义

语法：

    ObjectDef       ::= id ClassTemplate

#6.表达式

##6.1.表达式类型化

##6.2.字面值

##6.3.Null值

##6.4.指示器

##6.5.This和Super

##6.6.函数应用

##6.7.方法值

##6.8.类型应用

##6.9.元组

##6.10.实例创建表达式

##6.11.代码块

##6.12.前缀，中缀及后缀运算

##6.13.类型化的表达式

##6.14.标注表达式

##6.15.赋值

##6.16.条件表达式

##6.17.While循环表达式

##6.18.Do循环表达式

##6.19.For语句段

##6.20.Return表达式

##6.21.Throw表达式

##6.22.Try表达式

##6.23.匿名函数

##6.24.语句

##6.25.隐式转换

#7.隐含参数和视图

##7.1.implicit修饰符

##7.2.隐含参数

##7.3.视图

##7.4.视图边界

#8.模式匹配

##8.1.模式

##8.2.类型模式

##8.3.模式中的类型参数推断

##8.4.模式匹配表达式

##8.5.模式匹配匿名函数

#9.顶级定义

##9.1.编译单元

##9.2.打包

##9.3.包引用

##9.4.程序

#10.XML表达式与模式

##10.1.XML表达式

##10.2.XML模式

#11.用户定义的标注

#12.Scala标准库

##12.1.根类

##12.2.值类

##12.3.标准引用类

##12.4.Node类

##12.5.Predef对象

##12.6.