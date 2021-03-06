类从加载到虚拟机内存开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading），验证（Verification），准备（Preparation），解析（Resolution）， 初始化（Initiallization），使用（Using）和卸载（Unloading），其中验证，准备，解析3个部分统称为连接（Linking）

## 加载
在加载阶段，虚拟机需要完成如下操作：
>1. 通过一个类的全限定名来获取此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

虚拟机不仅可以从Class文件获取类的二进制流， 还可以从ZIP包中读取（jar，ear，war格式），从网络中获取（Applet应用），运行时计算生成（动态代理技术），或由其他文件生成（如Jsp）。  

加载阶段可以使用系统提供的引导类加载器来完成，也可以由用户自定义的类加载器去完成，开发人员可以通过自定义类加载器去控制字节流的获取方式（即重写一个类加载器的loadClass()方法）

但数组类本身不通过类加载器创建，而是由Java虚拟机直接创建，不过数组类的元素类型（ElementType，指数组去掉所有维度的类型）最终还是需要类加载器创建。

加载阶段完成后，二进制字节流就按照虚拟机所需的格式存储在方法区中，方法区中的数据存储格式由虚拟机实现自定义。然后在内存中实例化一个java.lang.Class类的对象。这个对象将作为程序访问方法区这些类型数据的外部接口。

## 验证
验证是连接阶段的第一步，这个阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

class文件并不一定要求用Java源码编译而来，可以通过任何途径产生，所以虚拟机必须检查输入的字节流，否则可能因为载入了有害的字节流而导致系统崩溃。

验证阶段大致完成下面4个阶段的检验动作：文件格式验证，元数验证，字节码验证，符号引用验证  
1. 文件格式验证  
验证字节是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理。  
2. 元数据验证  
对字节码描述进行语义分析，以保证其描述的信息符合Java语言规范的要求。
这阶段的主要目的是对类的元数据进行语义校验，保证不存在不符合Java语言规范的元数据信息。  
3. 字节码验证  
通过数据流和控制流分析， 确定程序语义是合法的，符合逻辑的。在第二阶段对元数据信息中的数据类型做完校验后，这个阶段将对类的方法体进行校验分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的事件。    
4. 符号引用验证  
该阶段的校验发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段——解析阶段中发生。符号引用验证可以看做对类自身外（常量池中的各种符号引用）的信息进行匹配性校验。  
符号引用验证的目的是确保解析动作能正常执行，如果无法通过符号引用验证，就会抛出一个java.lang.IncompatibleClassChangeError异常的子类，如java.lang.IllegalAccessError，java.lang.NoSuchMethodError,java.lang.NoSuchFieldError等。

## 准备
准备阶段是正式为类变量分配内存并设置类变量初始值的阶段， 这些变量所使用的内存都将在方法区中进行分配。  

这时进行内存分配的仅包括类变量（被static修改的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中。其次，初始值通常指数据类型的零值。如类变量`public static int value = 123`，准备阶段后value初始值为0而不是123， 因为这时尚未开始执行任何Java方法，而把value赋值为123的putstatic指令是程序被编译后，存放在类构造器<clinit>()方法中，所以把value赋值为123的动作将在初始化阶段才会执行。

## 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。  

符号引用：以一组符号来描述所引用的目标，符号引用可以是任何形式的字面量，但使用前必须先无歧义地定位到目标。符号引用与虚拟机实现的内存布局无关，引用的目标并一定已经加载到内存中。在Class文件中以CONSTANT_Class_info，CONSTANT_Fieldref_info等常量形式出现。  
直接引用：可以是直接指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。直接引用是和虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不相同。如果有了直接引用，那引用的目标必定已经在内存中存在。  

解析动作主要针对类或接口，字段，类方法，接口方法，方法类型，方法句柄和调用点限定符7类符号引用进行，对应常量池中的CONSTANT_Class_info，CONSTANT_Fieldref_info，CONSTANT_Methodref_info，CONSTANT_InterfaceMethodref_info，CONSTANT_MethodType_info，CONSTANT_MethodHandle_info和CONSTANT_InvokeDynamic_info 7种常量类型。

虚拟机规范并未规定解析阶段发生的具体时间，只要求在执行anewarray, checkcase, getfield, getstatic, instanceof, invokeddynamic, invokeinterface, invokespecial, invokestatic, invokevirtual, ldc, ldc_w, multianewarray, new, putfield和putstatic这16个用于操作符引用的字节码之前，先对它们所使用的符号引用进行解析。所以虚拟机实现可以根据需要来判断是在类被加载器加载时就对常量池的符号引用进行解析，还是等到一个符号引用将要使用前才去解析它。  

除invokedynamic指令外，虚拟机可以对第一次解析结果进行缓存（在运行时常量池中记录直接引用，并把常量标识为已解析状态）从而避免解析动作重复进行。


## 初始化
在前面的类加载过程，除了在加载阶段用户应用程序可以通过自定义加载器参与外， 其他动作完成由虚拟机主导和控制。到了初始化阶段，才真正开始执行类定义中的Java程序代码。   
在准备阶段，类变量已经赋过一次初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源，或者说：执行类构造器`<clinit>()`方法的过程。

`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序决定。  
`<clinit>()`方法与类的构造函数（实例构造器`<init>()`方法）不同，它不需要显形调用父类的构造器，虚拟机会保证子类的`<clinit>()`方法执行前，父类的`<clinit>()`方法已经执行完毕。所以父类中定义的静态语句块要优先于子类的变量赋值操作。

接口中不能使用静态语句块，但仍然有变量初始化赋值操作，因此接口与类一样都会生成`<clinit>()`方法，但与类不同，执行接口的`<clinit>()`方法不需要执行父接口的`<clinit>()`方法，只有当父接口中定义的变量使用时，父接口才初始化，另外，接口的实现类在初始化时也不会执行接口的`<clinit>()`方法。

虚拟机会保证一个类的`<clinit>()`方法在多线程环境中被正确地加锁，同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他线程需要阻塞等待，直到活动线程执行`<clinit>()`方法完毕。如果在一个类的`<clinit>()`方法中耗时很长，可能造成多个线程阻塞。  


虚拟机规范严格规定 **有且只有** 5种情况必须立即对类进行初始化  
1. 遇到new，getstatic，putstatic或invokestatic这4条字节码指令时，如果类没有进行初始化，需要先触发其初始化。生成这4条指令最常见的Java代码场景：使用new实例化对象，读取或设置一个类的静态字段（被final修饰，已在编译期把结果放到常量池的静态字段除外），调用一个类的静态方法。   
2. 使用java.lang.refrect包的方法对类进行反射调用的时候，如果类没有进行初始化，则先触发其初始化。  
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。  
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。  
5. 当使用JDK 1.7 的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果是REF_getStitic，REF_putStatic，REF_invokeStatic的方法句柄，并且方法句柄对应的类没有进行初始化，则先触发其初始化。