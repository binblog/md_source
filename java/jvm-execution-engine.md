每一个运行的线程都有如下组件：
+ Program Counter (PC)
当前指令（或操作码）的地址，除非它是本地的。  所有CPU都有一个PC，通常PC在每个指令之后递增，因此它保存了要执行的下一条指令的地址。 JVM使用PC来跟踪其执行指令的位置，PC实际上将指向方法区域中的存储器地址。

Java虚拟机栈
一个new frame 会被创建并添加到Java虚拟机栈顶。当一个方法开始执行。而执行完成或抛出异常后将removed 。
每一个frame包含了：
+ 	局部变量表
+ 方法返回地址
+ 操作数栈
+ 指向运行时常量池中该方法定义


局部变量表
栈帧中包含了一组本地变量，包含了在方法执行时使用的所有的变量。包括`this`，所有的方法参数和方法内定义的局部变量。在Java程序编译为Class文件时，就在方法的Code属性的max_locals数据项中确定了该方法所需要分配的局部变量表的最大容量。

单个局部变量空间可以保存boolean，byte，char，short，int，float，reference或returnAddress类型的值。而long类型或double类型的值将占用两个连续的局部变量空间。

局部变量通过索引来寻址。第一个局部变量的索引为零。实例方法中，第0个局部变量是`this`



操作数栈
当执行byte code时需要不断操作数栈，类似于本地cpu使用通用寄存器。
最大深度也在编译的时候写入到Code属性的max_stacks数据项中。
操作数栈是一个后入先出栈。大多数JVM字节代码通过push, pop, duplicate, swap，执行产生或消耗值的操作来操作操作数。因此，在字节代码中非常频繁地在局部变量数组和操作数堆栈之间移动值。 例如，简单的变量初始化导致与操作数栈交互的两个字节代码。  
`int i = 0;`  
将会被编译为如下byte code：
```
 0:	iconst_0	// Push 0 to top of the operand stack
 1:	istore_1	// Pop value from top of operand stack and store as local variable 1
```

```
    public int add(int a, int b) {
        return a + b;
    }
```
转化为字节码
```
  public int add(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1		// 加载下标为1的局部变量到操作数栈顶
         1: iload_2	// 加载下标为2的局部变量到操作数栈顶
         2: iadd	// 加法操作
         3: ireturn	// 返回结果
      LineNumberTable:
        line 30: 0

```

方法返回地址

方法退出后，都需要回到方法被调用的位置，程序才能继续执行。方法返回时可能需要在栈帧中保存一些信息，用于帮助恢复它的上层方法的执行状态。一般，方法正常退出，调用者的PC计数器的值可以作为返回地址，栈帧中很可能保存这种值。而方法异常退出时，返回地址是通过异常处理器确定的，栈帧一般不会保存这部分信息。

方法退出的过程实际是等于把当前栈帧出栈，因些退出时可能执行的操作有：恢复上层方法的局部变量表和操作数栈，把返回值（如果有）压入调用者栈帧的操作数栈中，调整PC计数器的值以指向方法调用指令后面的一条指令等。

动态连接

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接。

C / C ++代码通常编译为object file，然后将多个object file链接在一起，产生可用的程序，如可执行文件或dll。 在链接阶段期间，每个目标文件中的符号引用被相对于最终可执行文件的实际存储器地址替换。 但在Java中，这个链接阶段在运行时动态完成。  

当编译Java类时，变量和方法引用都作为符号引用存储在类的常量池中。符号引用是逻辑引用，而不是指向实际物理内存位置的直接引用。JVM实现可以选择何时解析符号引用，它可以发生在类加载后的类文件验证期间（静态解析），或符号引用第一次使用时（延迟解析）。
除了invokedynamic指令外，虚拟机会对符号引用第一次解析结果进行缓存，以后的引用可以直接使用缓存的结果 。




Java虚拟机中提供了5条方法调用字节码指令：

invokestatic：调用静态方法

invokespecial：调用实例构造器<init>方法，私有方法和父类方法

invokevirtual：调用所有的虚方法

invokeinterface：调用接口方法，会在运行时再确定一个实现该接口的对象。  

invokedynamic: 先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法。


解析  


在类加载的解析阶段，会将一部分符号引用转化为直接引用，这种解析能成立的前提是：方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期不可改变。也就是，调用目标在程序代码写好，编译器进行编译时必须确定下来。主要包括静态方法和私有方法两大类。静态方法与类型直接关联，私有方法在外部不可访问。它们不可能通过继承或重写其他版本。   
只要能被invokestatic和invokespecial指令调用的方法，都可以在解析阶段确定唯一的调用版本，包括静态方法，私有方法，实例构造器，父类方法，它们在类加载时间就会把符号引用解析为该方法的直接引用。这些方法称为非虚方法。其他方法则为虚方法。而final方法虽然使用invokevirtual，但它无法被覆盖，也是一种非虚方法。



重载就是静态解析的结果，通过方法参数匹配一个合适的重载方法。
而多态则是动态解析的结果，需要在第一次执行时才能决定解析的结果。


查看如下简单代码:
```
public class Dispatch {
    public void printTime(java.util.Date date) {
        System.out.println("date : " + date.toString());
    }

    public  void printTime(java.sql.Time time) {
        System.out.println("time : " + time.toString());
    }

    public static void main(String[] args) {
        Dispatch dispatch = new Dispatch();
        java.util.Date date = new java.util.Date();
        java.sql.Time time = new java.sql.Time(date.getTime());
        dispatch.printTime(date);
        dispatch.printTime(time);
    }
}
```
输入结果
```
date : Mon Jan 09 17:54:32 GMT+08:00 2017
time : 17:54:32
```
我们都知道，java.sql.Time time继承了java.util.Date。
上述代码中调用了重载方法printTime。
`java.util.Date time = new java.sql.Time(date.getTime());`，java.util.Date为变量的静态类型或外观类型，而java.sql.Time为变量的实际类型。变量本身的静态类型不会被改变，并且最终的静态类型是在编译期可知。而实际类型变化结果在运行期才可确定，编译器在编译程序的时候并不知道一个对象的实际类型是什么。虚拟机（准确来说是编译器）在重载时通过参数的静态类型而不是实际类型作为判定依据的。并且静态类型是编译期可知的。  
因此，在编译阶段，Javac编译器会根据参数的静态类型决定使用哪个重载版本，
上述代码中
```
        java.util.Date date = new java.util.Date();
        java.util.Date time = new java.sql.Time(date.getTime());
```
静态类型都是java.util.Date，因此
```
        dispatch.printTime(date);
        dispatch.printTime(time);
```
都调用到了`printTime(java.util.Date date)`方法。

通过javap -v Dispatch.class，可以看到如下byte code：
上面的byte code对应代码
```
        dispatch.printTime(date);
        dispatch.printTime(time);
```
```
28: aload_1		
29: aload_2
30: invokevirtual #19                 // Method printTime:(Ljava/util/Da
te;)V
33: aload_1
34: aload_3
35: invokevirtual #19                 // Method printTime:(Ljava/util/Da
te;)V
38: return
```
可以看到两次方法调用都将指向了`Method printTime:(Ljava/util/Da`

重写（多态）
根据变量的实际类型作为判定依据的。

只有运行时才确定实际类型。
```
    public void printTime(java.util.Date date) {
        System.out.println("date : " + date.toString());
    }
```
将调用参数的实际类型中实现的toString方法。

`printTime(java.util.Date date) `中`date.toString()`方法的对应的byte code：
```
15: aload_1
16: invokevirtual #7                  // Method java/util/Date.toString:()Ljava/lang/String;
```
invokevirtual指令执行第一步就是在运行期间确定接收者的实际类型，所以两次调用invokevirtual指令把常量池方法符号引用解析到了不同的直接引用。
动态分派。



