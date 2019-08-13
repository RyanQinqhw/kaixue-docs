## Kotlin 里那些「不是那么写的」

Kotlin 作为一门年轻的高级开发语言，相比上个世纪的老大哥 Java，在代码的简洁性和可维护性上都有不少的提高。Java 中四五行的代码逻辑，Kotlin 可能一行就搞定了，还有一些容易混淆出错的逻辑，Kotlin 给出了更加清晰合理的控制。这篇文章就让我们对比 Java 来看看 Kotlin 中「不是那么写的」地方。

### Constructor

上一篇中简单介绍了 Kotlin 的构造器，这一节具体看看 Kotlin 的构造器和 Java 有什么不一样。首先看两段分别用 Java 和 Kotlin 写的 `User` 类：

- Java

    ``` java
    ☕️
    public class User {
        int id;
        String name;
          👇    👇
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }
    }
    ```

- Kotlin

    ``` kotlin
    🏝️
    class User {
    // 👆 没有 public
        val id: Int
        val name: String
             👇
        constructor(id: Int, name: String) {
     // 👆 没有 public
            this.id = id
            this.name = name
        }
    }
    ```

可以发现构造器的写法主要有两点不同：

- Java 中构造器和类同名，Kotlin 中使用 `constructor` 表示。
- Kotlin 构造器没有 public 修饰，因为默认可见性就是公开的，关于可见性修饰符这里先不展开，后面会讲到。

Kotlin 除了和 Java 类似的构造器之外还引入了 「主构造器 primary constructor」，可以让你的代码更加直观和简洁：

``` kotlin
🏝️    // 👇 没有 constructor
class User(val id: Int, val name: String) {}
         // 👆 构造器参数直接作为属性声明
```

而和 Java 比较类似的构造器在 Kotlin 中称之为 「次构造器 secondary constructor」。一个区分主构造器和次构造器的简单办法是，声明在类的大括号里面的是次构造器，声明在外面的是主构造器。接下来分别看看这两种构造器。

#### 主构造器

Kotlin 中每个类最多只能有一个主构造器（可以没有），像下面这样写在类名后面：

``` kotlin
🏝️            👇
class User constructor(name: String) {}
```

如果需要限制构造器的可见性，直接写在 `constructor` 前面，关于可见性本文后面会讲到：

``` kotlin
🏝️           👇
class User private constructor(name: String) {}
```

当需要给构造器添加注解时，也是放在 `constructor` 前面的：

``` kotlin
🏝️              👇                                            👇
class User @JvmOverloads constructor(name: String, sex: String = "male")
```

这里 `@JvmOverloads` 表示在 Jvm 中生成重载的两个构造方法，相当于在 `User` 声明了两个构造器：

``` kotlin
🏝️
class User {
    constructor(name: String, sex: String) {
    	...   
    }
    constructor(name: String) : this(name, "male")
}
```

当不需要修饰构造器时可以省略掉关键字 `constructor`：

``` kotlin
🏝️       👇
class User(name: String) {}
```

##### `init`

如果我想在构造器中执行初始化操作该怎么做呢？为此 Kotlin 提供了初始化代码块来负责这部分任务：

``` kotlin
🏝️
init {
    ...
}
```

初始化代码块由 init 关键字和一对大括号构成，这其实不是 Kotlin 创造的新概念，对应的 Java 在类中也有类似的功能，用来完成初始化操作：

``` java
☕️
{
    ...
}
```

Java 中除了这种初始化代码快，还有静态初始化代码块：

``` java
☕️
class Sample {
     👇
    static {
        ...
    }
}
```

在 Kotlin 中类的静态初始化代码块移到了 `companion object` 「伴生对象」中，至于 `companion object` 是什么，这里先不展开，后文会讲到：

``` kotlin
🏝️
class Sample {
       👇
    companion object {
         👇
        init {
            ...
        }
    }
}
```

- 一个 Kotlin 类中可以有多个初始化代码块，它们的执行顺序和创建的顺序是一致的：

    ``` kotlin
    🏝️
    class User {
        init {
            println("First init block.")
        }
        init {
            println("Second init block.")
        }
    }
    ```

    实例化时输出：

    ``` bash
    First init block.
    Second init block.
    ```

- 当类中存在属性初始化代码时，执行的优先级和初始化代码块是同级的：

    ``` kotlin
    🏝️
    class User {
        val firstProperty = "First property.".also { println(it) }
        init {
            println("First init block.")
        }
        val secondProperty = "Second property.".also { println(it) }
        init {
            println("Second init block.")
        }
    }
    ```

    实例化时输出：

    ``` bash
    First property.
    First init block.
    Second property.
    Second init block.
    ```

    由此可见属性初始化代码和初始化代码块处于同一个执行上下文，另一个证据是这两块代码都可以访问主构造器的参数：

    ``` kotlin
    🏝️
                👇
    class User(name: String) {
                           👇
        val length: Int = name.length
        init {
                                  👇
            println("My name is $name")
        }
    }
    ```

##### 主构造器属性声明

Kotlin 还有一种简洁的写法用于将主构造器中的参数声明为属性，并用参数值初始化：

``` kotlin
🏝️             👇                👇
class User(val name: String, val age: Int) {}
        // 👆 val/var 表明声明为属性
```

这种写法等价于：

``` kotlin
🏝️    // 👇 没有 val/var
class User(name: String, age: Int) {
         👇
    val name: String = name
        👇
    val age: Int = age
}
```

相比之下确实简化了很多。

#### 次构造器

Kotlin 中的次构造器和 Java 类似，写在类中，通过 `constructor` 表示并且不可以省略：

``` kotlin
🏝️
class User {
    private val name: String
        👇
    constructor(name: String) {
        this.name = name
        println("My name is $name.")
    }
}
```

##### 次构造器需要调用主构造器

写次构造器时，如果存在主构造器，需要调用到主构造器，在括号后面加上 `: this()`：

``` kotlin
🏝️
class User(val name: String) {
    private var age: Int = 0
                                        // 👇 这里的 this(name) 调用的是主构造器
    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }
}
```

Java 中也有类似的概念：

``` java
☕️
class Sample {
    Sample() {
        System.out.println("first constructor");
    }

    Sample(String input) {
        this(); // 👈 和 Kotlin 中次构造器括号右边的 :this() 作用相同，表示调用另外一个构造器
        System.out.println("second constructor ");
    }
}
```

由于 Java 中没有主次构造器的概念，所以调用别的构造器不是强制的。

前面讲的属于直接调用主构造器，也可以通过间接的方式调用主构造器：

``` kotlin
🏝️
class User(val name: String) {
	  var age: Int = 0
    var gender: String = "male"
                                           👇
    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }
                                                           👇
    constructor(name: String, age: Int, gender: String) : this(name, age) {
        this.gender = gender
    }
}
```

这段代码里，第二个次构造器就是通过调用第一个次构造器间接调用了主构造器。通过 `this` 关键字引用别的构造器，通过参数定位具体的构造器。

##### 调用顺序

初始化代码块永远在次构造器之前执行：

- 被调用构造器的初始化代码总是先执行，而初始化代码块属于主构造器的一部分，所以初始化代码块在所有的次构造器之前执行：

    ``` kotlin
    🏝️
    class User constructor(name: String) {
      // 👇 属于主构造器一部分
        init {
            println("Init block.")
        }
                                            // 👇 次构造器执行前先调用主构造器
        constructor(name: String, age: Int) : this(name) {
            println("Secondary constructor.")
        }
    }
    ```

    创建上面这个类的实例时会输出：

    ``` bash
    Init block.
    Secondary constructor.
    ```

- 即使没有声明主构造器，初始化代码块也先于次构造器执行，这时次构造器相当于调用了一个空的主构造器：

    ``` kotlin
    🏝️
    class User {
        init {
            println("Init block.")
        }
        constructor(name: String) {
            println("Secondary Constructor.")
        }
    }
    ```

    创建这个类的实例时输出：

    ``` bash
    Init block.
    Secondary Constructor.
    ```

    上面这段代码等价于：

    ``` kotlin
    🏝️             👇
    class User constructor() {
        init {
            println("Init block.")
        }     
                                     👇
        constructor(name: String) : this() {
            println("Secondary Constructor.")
        }
    }
    ```

### `final`

上一篇在「 `var` 和 `val`」 这一节中讲到 Kotlin 中的 `val` 和 Java 中的 `final` 类似，表示只读变量，不能修改。这里具体对比下 Java 和 Kotlin 中的只读变量：

- Java

  ``` java
  ☕️
   👇
  final int final1 = 1;
               👇  
  void method(final String final2) {
      System.out.println(final2);
       👇
      final Date final3 = new Date();
      System.out.println(final3);
  }
  ```

- Kotlin

  ``` kotlin
  🏝️
  
  👇
  val fina1 = 1
         // 👇 没有 val
  fun method(final2: String) {
      println(final2)
      👇
      val final3 = Date()
      println(final3)
  }
  ```

可以看到不同点主要有：

- final 变成 val，由于类型推断，类型可以省略不写，写法上简短了一些。
- Kotlin 方法参数默认是 val 类型，所以参数前不需要写 val 关键字。

相比 Java 中通过额外添加 final 前缀来表示只读，Kotlin 的只读声明简化了很多，只是把 var 最后一个字母改成 l，大大增加了开发者使用只读类型的频率。虽然只是减少了一个单词，但考虑到变量声明的频率，总体效果还是很可观的。这种优化使得在该加只读限制的地方加上限制，减少了出现错误的概率，从而提高代码质量。

#### `val`自定义 getter

不过 `val` 和 `final` 还是有一点区别的，虽然 `val` 修饰的变量不能二次赋值，但可以通过自定义变量的 getter 方法来让变量每次被访问时返回动态计算的值：

``` kotlin
🏝️

👇
val size: Int
	get() { // 👈
        return items.size
    }
```

前面说到类的属性类型可以通过初始化代码进行类型推断，除此之外也可以通过 getter 方法的返回值推断，而且 Kotlin 中可以通过 `=` 直接连接函数表达式，所以上面这段代码可以简化为：

``` kotlin
🏝️  // 👇 没有类型声明，类型根据 items.size 推断    
val size get() = items.size
```

不过这个属于特殊用法，一般情况下 `val` 还是对应于 Java 中的 `final` 使用的。

重写 `val` 变量 `getter()` 方法一个可能的应用是用于简化一些没有参数的属性类方法调用：

- 方法形式：

    ``` kotlin
    🏝️
    
    👇
    fun isEmpty(): Boolean {
        return items.size == 0
    }
    ```

- `val` 形式：

    ``` kotlin
    🏝️
    
    👇     // 👇 没有括号
    val isEmpty: Boolean
    		get() {
            return items.size == 0
        }
    ```

上面的方法和属性的调用方式分别如下：

``` kotlin
any.isEmpty()
any.isEmpty // 👈 没有括号
```

少了一对括号，可以让代码简洁一些。

### `static` property / function

前面讲的 `var` / `val` 变量都是非静态变量，必须先创建一个实例对象才能使用，如果想通过类直接引用，就需要用到静态变量和方法。接下来看看 Kotlin 中的静态变量和方法：

#### Kotlin 中的静态变量和方法

Java 和 Kotlin 中声明静态变量和方法：

- 在 Java 中：

    ``` java
    ☕️
    class A {
                👇
        public static int property = 1;
                👇
        public static void method() {
            println("A.method()")
        }
    }
    ```

    这样就可以不需要创建该类的实例，直接通过类名调用：

    ``` java
    ☕️
                  👇
    int variable = A.property;
    👇
    A.method();
    ```

- 在 Kotlin 中如果想实现类似的功能该怎么做呢，在类中可以创建一个叫作 `companion object` 的东西，将想定义的静态属性和方法放在其中：

    ``` kotlin
    🏝️
    class A {
        // 👇 新东西
        companion object {
         // 👇 和类中声明相似
            val property: Int = 1
            fun method() {
                println("A.method()")
            }
        }
    }
    ```

    这样在调用的时候可以像 Java 一样去调用：

    ``` kotlin
    🏝️
                  👇
    val variable = A.property
    👇
    A.method()
    ```

    这个 `companion object`  是什么意思呢，为什么要使用这种方式定义类级别的变量或者方法，先来看看什么是 object。

#### `object`

##### `object` 创建单例类

`object` 字面意思是对象，与 Java 中需要通过 `new` 创建一个对象不同，Kotlin 中可以通过 `object` 直接创建一个对象实例：

``` kotlin
🏝️

// 👇 class 替换成了 object
object A {
 // 👇 和普通类中声明类似
    val number: Int = 1
    fun method() {
        println("A.method()")
    }
}
```

和类的定义类似，不过 `class` 关键字替换成 `object` ，调用方式如下：

``` kotlin
🏝️
         // 👇 和调用静态变量类似
val result = A.number + 1
👇 
A.method()
```

看着是不是很像 Java 中的单例，`object` 一个很重要的用途就是声明一个单例类。

对比看看 Java 中要实现上面这个单例类需要怎么做：

``` java
☕️
public class A {
              👇
    private static A sInstance;
            👇
    public static A getInstance() {
        if (sInstance == null) {
            sInstance = new A();
        }
        return sInstance;
    }
      👇
    private A() {
    }

    public int number = 1;

    public void method() {
        System.out.println("A.method()");
    }
}
```

调用的时候：

``` java
☕️             // 👇 多个方法
int result = A.getInstance().number + 1;
       👇
A.getInstance().method()
```

可以看到 Java 中为了实现单例类写了大量的模版代码，在没有 Kotlin 的时候大家写多了也就习惯了，直到遇见了 Kotlin 才发现单例类原来可以这么简单，而且调用也比较简洁，不需要 getInstance() 方法。

通过 `object` 创建的对象也可以继承别的类或者接口，和创建类是一样的：

``` kotlin
🏝️
open class A {
    open fun method() {
        println("A.method()")
    }
}

interface B {
    fun interfaceMethod()
}
  👇      👇   👇
object C : A(), B {

    override fun method() {
        println("C.method()")
    }

    override fun interfaceMethod() {
        println("C.interfaceMethod()")
    }
}
```

##### `object` 创建匿名类

有时需要改变类方法的实现，但因为改动很少不想创建该类的子类，Java 中是通过创建匿名内部类来实现的：

``` java
☕️                                              👇 
ViewPager.SimpleOnPageChangeListener listener = new ViewPager.SimpleOnPageChangeListener() {
	@Override // 👈
	public void onPageSelected(int position) {
		// override
	}
};
```

Kotlin 中通过对象表达式来表示，对象表达式简单说就是 `=` 加上 `object` 声明的对象：

``` kotlin
🏝️              // 👇 没有 object 名字
val listener = object: ViewPager.SimpleOnPageChangeListener() {
		   // 👆 「= 和 object: 」共同组成对象表达式
    override fun onPageSelected(position: Int) {
        // override
    }
}
```

和 Java 创建匿名类的方式很相似，你可以简单理解为通过 object 创建一个匿名内部类。这里需要注意的是，`=` 后的语句不能单独存在，因为对象表达式是指将对象赋值给一个变量或者作为参数传递给方法，如果没有 `=` 以及前面的变量，这段代码就不能称为对象表达式，就会报错：

``` kotlin
🏝️
  // 👇 compile error: Name expected
object: ViewPager.SimpleOnPageChangeListener() {
    override fun onPageSelected(position: Int) {
        // override
    }
}
```

`object` 后编译器提示这里需要一个对象的名字，因为编译器以为你想创建一个继承 `ViewPager.SimpleOnPageChangeListener` 的对象，而不是一个对象表达式。

##### `companion object`

前面说到访问 `object` 中的变量或者方法时直接通过类名引用，就像 Java 的静态变量和方法一样，不同的是不需要在每个变量和方法前面用 `static` 修饰，因为 `object` 创建的对象内所有变量和方法默认都是静态的，没得选。如果只想让类中的一部分方法和变量是静态的该怎么做呢：

``` kotlin
🏝️
class A {
          👇
    object B {
        var c: Int = 0
    }
}
```

如上，可以在类中创建一个对象，把需要静态的变量或方法放在内部对象 B 中，外部可以通过如下的方式调用该静态变量：

``` kotlin
🏝️
A.B.c
 👆
```

类中嵌套的对象可以用 `companion` 修饰：

``` kotlin
🏝️
class A {
       👇
    companion object B {
        var c: Int = 0
    }
}
```

`companion` 可以理解为伴随、伴生，表示修饰的对象和外部类绑定，这样的好处是调用的时候可以省掉对象名：

``` kotlin
🏝️
A.c // 👈 B 没了
```

当有 `companion` 修饰时，对象的名字也可以省略掉：

``` kotlin
🏝️
class A {
                // 👇 B 没了
    companion object {
        var c: Int = 0
    }
}
```

这就是这节最开始讲到的，和 Java 静态变量或方法的等价写法：`companion object`。

#### top-level property / function 声明

不同于 Java 所有的方法和变量都必须写在类中，Kotlin 可以在类外写变量和方法，这个称之为「top-level property/function」即「顶层声明」。

``` kotlin
🏝️
package com.hencoder.plus // 👈 属于 package

// 不在 class/object 内
fun topLevelFuncion() {
}
```

这样写的属性和函数，不属于任何 `class`，而是直接属于 `package`，它和静态变量、静态方法一样是全局的，但用起来更方便：你在其它地方用的时候，就连类名都不用写：

``` kotlin
🏝️
import com.hencoder.plus.topLevelFunction // 👈 直接 import 方法

topLevelFunction()
```

写在顶级的方法或者变量有个好处，在 Android Studio 中写代码时，IDE 很容易根据你写的方法前几个字母自动联想出相应的方法，提高了写代码的效率，而且可以减少项目中的重复代码。

##### 命名相同的顶级函数

顶级函数不写在类中可能有一个问题：如果在不同文件中声明命名相同的函数，使用的时候会不会混淆？来看一个例子：

* 在 `org.kotlinmaster.library` 包下有一个方法 method：

  ``` kotlin
  🏝️
  package org.kotlinmaster.library1
                             👆
  fun method() {
      println("library1 method()")
  }
  ```

* 在 `org.kotlinmaster.library2` 包下也有一个同名方法：

  ``` kotlin
  🏝️
  package org.kotlinmaster.library2
                             👆
  fun method() {
      println("library2 method()")
  }
  ```

在使用的时候如果同时调用这两个同名方法会怎么样：

```kotlin
🏝️
import org.kotlinmaster.library1.method
                           👆
fun test() {
    method()
                       👇
    org.kotlinmaster.library2.method()
}
```

可以看到当出现两个同名顶级函数时，IDE 会自动加上包前缀来区分，这也印证了顶级函数是属于包的特性。

#### 对比

那在实际使用中，在 `object`、`companion object` 和 top-level 中该选择哪一个呢？简单来说按照下面这两个条件判断：

- 如果想写工具类的功能，直接创建 top-level 的函数，放在文件中。
- 如果需要继承别的类或者实现接口，就用 `object` 和 `companion object`。

### 常量

除了上面讲到的静态变量和方法会用到 `static`，Java 中声明常量时也会用到：

- Java 中声明常量：

    ``` java
    ☕️
    public class Sample {
                👇     👇
        public static final int CONST_NUMBER = 1;
    }
    ```

- Kotlin 中声明常量：

    ``` kotlin
    🏝️
    class Sample {
        // 👇 在 companion object 内
        companion object {
             👇                  // 👇 基础类型
            const val CONST_NUMBER = 1
        }
    }
    ```

发现不同点有：

- Kotlin 的常量必须声明在类的伴随对象内，因为常量是静态的。
- Kotlin 新增了修饰常量的 `const` 关键字。

除此之外还有一个区别：

- Kotlin 中只有基本类型和 String 类型可以声明成常量。

原因是 Kotlin 中的常量指的是 「compile-time constant 编译时常量」，它的意思是「编译器在编译的时候就知道这个东西在每个调用处的实际值」，因此可以在编译时直接把这个值硬编码到代码里使用的地方。

而非基础和 String 类型的变量，可以通过调用对象的方法改变对象内部的值，这样这个变量就不是常量了，来看一个 Java 的例子，比如一个 User 类：

``` java
☕️
public class User {
    int id; // 👈 可修改
    String name; // 👈
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

在使用的地方声明一个 `static final` 的 User，它是不能二次赋值的：

``` java
☕️
static final User user = new User(123, "Zhangsan");
  👆    👆
```

但是可以通过访问这个类的成员变量改变它的值：

``` java
☕️
user.name = "Lisi";
      👆
```

所以相比 Java 里声明的常量，Kotlin 的常量限制更严格，更加符合常量的定义。

### 数组和集合

前面讲了 Kotlin 中的对象和常量，接下来看看另一个编程中常见的主题：数组和集合。

#### 数组

先声明一个 String 数组：

- Java 中的写法：

    ``` java
    ☕️
    String[] strs = {"a", "b", "c"};
          👆        👆
    ```

- Kotlin 中的写法：

    ``` kotlin
    🏝️
    val strs: Array<String> = arrayOf("a", "b", "c")
                👆              👆
    ```

可以看到 Kotlin 中的数组是一个拥有泛型的类，创建方法也是泛型方法，和集合数据类型一样。

##### 取值和修改

- Kotlin 中获取或者设置数组元素和 Java 一样可以使用方括号加下标的方式索引：

    ``` kotlin
    🏝️
    println(strs[0])
       👇      👆
    strs[1] = "B"
    ```

- 除此之外，Kotlin 的 Array 类型还可以使用 `get()` 和 `set()` 方法取值和赋值：

    ``` kotlin
    🏝️
    println(strs.get(0))
         👇      👆
    strs.set(1, "B")
    ```

第二种方式有点像集合类型，这正是 Kotlin 中将数组泛型化的原因：对数组的操作像集合一样功能更强大，由于泛型化 Kotlin 可以通过扩展方法的方式给数组增加很多有用的工具方法：

- `contains()`
- `first()`
- `find()`

这样数组的实用性就大大增加了。

##### 不支持协变

Kotlin 的数组编译成字节码使用的仍然是 Java 的数组，但在语言层面是泛型实现，这样会失去协变 (covariance) 特性，就是子类数组对象不能赋值给父类的数组变量：

 - Kotlin

    ``` kotlin
    🏝️
    val strs: Array<String> = arrayOf("a", "b", "c")
                      👆
    val anys: Array<Any> = strs // compile-error: Type mismatch
                    👆
    ```

- 而这在 Java 中是可以的：

    ``` java
    ☕️
    String[] strs = {"a", "b", "c"};
      👆
    Object[] objs = strs; // success
      👆
    ```

关于协变的问题，这里就先不展开，后面讲泛型的时候会提到。

#### 集合

大多数编程语言都有集合这个数据结构，用来表示一组数目可变、类型相同的数据。常见的集合类型有三种：`List`、`Set` 和 `Map`，它们的含义分别如下：

- `List` 以固定顺序存储一组元素，元素可以重复。
- `Set` 存储一组互不相等的元素，通常没有固定顺序。
- `Map` 存储 键-值 对的数据集合，键互不相等，但不同的键可以对应相同的值。

##### List

- Java 中创建一个列表：

  ``` java
  ☕️
  List<String> strList = new ArrayList<>();
                          👆
  strList.add("a");
  strList.add("b");
  strList.add("c"); // 👈 添加元素繁琐
  ```

- Kotlin 中创建一个列表：

  ``` kotlin
  🏝️             // 👇 因为类型推断，省略类型参数 <String>
  val strList = listOf("a", "b", "c") // 👈 一句代码创建、添加元素
         // 👆 类型推断，省略类型声明 :List<String>
  ```

首先能看到的是 Kotlin 中创建一个 `List` 特别的简单，一句代码搞定，有点像创建数组的代码。而且 Kotlin 中的 `List` 多了一个特性：支持 covariant (协变)。也就是说，可以把子类的 `List` 赋值给父类的 `List`：

- Kotlin：

  ``` kotlin
  🏝️
  val strs: List<String> = listOf("a", "b", "c")
                  👆
  val anys: List<Any> = strs // success
                 👆
  ```

- 而这在 Java 中是会报错的：

  ``` java
  ☕️
  List<String> strList = new ArrayList<>();
         👆
  List<Object> objList = strList; // 👈 compile error: incompatible types
        👆
  ```

对于协变的支持与否，`List` 和数组刚好反过来了。

###### 和数组的区别

Kotlin 中数组和 MutableList 的 API 是非常像的，主要的区别是数组的元素个数不能变。那在什么时候用数组呢？

这个问题在 Java 中就存在了？数组和 `List` 的功能类似，`List` 的功能更多一些，直觉应该用 `List` 。但数组也不是没有优势，基础类型 (`int[]`、`float[]`) 的数组不用自动装箱，性能好一点。

在 Kotlin 中也是同样的道理，在一些性能需求比较苛刻的场景，并且元素类型是基础类型时，用数组好一点。不过这里要注意一点，Kotlin 中要用专门的基础类型数组类 (`IntArray` `FloatArray` `LongArray`) 才可以免于装箱。

##### Set

- Java 中创建一个 `Set`：

  ``` java
  ☕️
  Set<String> strSet = new HashSet<>();
                        👆
  strSet.add("a");
  strSet.add("b");
  strSet.add("c"); // 👈 添加元素繁琐
  ```

- Kotlin 中创建相同的 `Set`：

  ``` kotlin
  🏝️            // 👇 省略类型参数
  val strSet = setOf("a", "b", "c") // 👈 一句代码
        // 👆 省略类型声明
  ```

和 `List` 类似，一句代码创建一个 `Set`，同样具有 covariant (协变) 特性。

##### Map

- Java 中创建一个 `Map`：

  ``` java
  ☕️
  Map<String, Integer> map = new HashMap<>();
                             👆
  map.put("key1", 1);
  map.put("key2", 2);
  map.put("key3", 3);
  map.put("key4", 3); // 👈 添加元素繁琐
  ```

- Kotlin 中创建一个 `Map`：

  ``` kotlin
  🏝️         // 👇 省略类型参数
  val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 3) // 👈 一句代码
     // 👆 省略类型声明
  ```

和上面两种集合类型相似创建代码很简单，一行搞定。

###### 取值和修改

- Kotlin 中的 Map 除了和 Java 中的一样可以使用 `get()` 根据键获取对应的值，还可以使用方括号的方式获取：

    ``` kotlin
    🏝️               👇
    val value1 = map.get("key1")
                   👇
    val value2 = map["key2"]
    ```

- 类似的，Kotlin 中也可以用方括号的方式改变 `Map` 中键对应的值：

    ``` kotlin
    🏝️         // 👇 和 mapOf() 有什么区别？下文会讲到
    val map = mutableMapOf("key1" to 1, "key2" to 2)
        👇
    map.put("key1", 2)
       👇
    map["key1"] = 2
    ```

##### 可变集合/不可变集合

上面修改 `Map` 值的例子中，创建函数用的是 `mutableMapOf()` 方法而不是 `mapOf()`，难道只有 `mutableMapOf()` 创建的才可以修改吗？是的，Kotlin 中集合分为两种类型：只读的和可变的。只读的集合在创建的时候就要确定好值，创建好后集合的 size 和元素值都不能改变。

- `listOf()` 创建不可变的 `List`，`mutableListOf()` 创建可变的 `List`。
- `setOf()` 创建不可变的 `Set`，`mutableSetOf()` 创建可变的 `Set`。
- `mapOf()` 创建不可变的 `Map`，`mutableMapOf()` 创建可变的 `Map`。

可以看到，有 mutable 前缀的方法创建的可变的集合。不可变的集合可以通过 `toMutable*()` 系方法转换成可变的集合：

``` kotlin
🏝️
val strList = listOf("a", "b", "c")
            👇
strList.toMutableList()
val strSet = setOf("a", "b", "c")
            👇
strSet.toMutableSet()
val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 3)
         👇
map.toMutableMap()
```

然后就可以对集合进行修改了，这里有一点需要注意下：`toMutable*()` 返回的是一个新建的集合，原有的集合还是不可变的。

#### `Sequence`

除了集合 Kotlin 还引入了一个新的容器类型 `Sequence`，它和 `Iterable` 一样用来遍历一组数据并可以对每个元素进行特定的处理，先来看看如何创建一个 `Sequence`。

##### 创建

- 类似 `listOf()` ，使用一组元素创建：

    ``` kotlin
    🏝️
    sequenceOf("a", "b", "c")
    ```

- 使用 `Iterable` 创建：

    ``` kotlin
    🏝️
    val list = listOf("a", "b", "c")
    list.asSequence()
    // 👆 List 实现了 Iterable 接口
    ```

- 使用 lamda 表达式创建：

    ``` kotlin
    🏝️                          // 👇 第一个元素
    val sequence = generateSequence(0) { it + 1 }
                                    // 👆 lamda 表达式，负责生成第二个及以后的元素，it 表示前一个元素
    println(sequence.take(3).toList())
                       // 👆 只取 sequence 中前三个元素
    ```

    上面这段代码输出：

    ``` bash
    [0, 1, 2]
    ```

这看起来和 `Iterable` 一样呀，为啥要多此一举使用 `Sequence` 呢？因为 `Sequence` 在两点上实现和 `Iterable` 不一样：

- 调用处理函数处理元素时，`Iterable` 是立即执行， `Sequence` 是懒加载。
- 调用多个处理函数时，`Iterable` 是一个函数遍历完所有元素后再执行下一个函数，`Sequence` 是一个元素执行完所有函数后再遍历下一个元素。

emmm...好像并不知道是什么意思。没关系，我们结合例子看看这两点不同。

##### 懒加载

来看一个例子：将一组字符串转大写输出。

-  `Iterable` ：

    ``` kotlin
    🏝️
    val letterList = listOf("a", "d")
        .map {
            println("map: $it")
            it.toUpperCase()
        }
    println("Upper case letter:")
    println(letterList) // 👈 letterList 使用的地方
    ```

    输出：

    ``` bash
    map: a
    map: d // 👈 map 操作先执行
    Upper case letter:
    [A, D]
    ```

-  `Sequence` ：

    ``` kotlin
    🏝️
    val letterSequence = sequenceOf("a", "d")
        .map {
            println("map: $it")
            it.toUpperCase()
        }
    println("Upper case letter:")
    println(letterSequence.toList()) // 👈 letterSequence 使用的地方
    ```

    输出：

    ``` bash
    Upper case letter:
    map: a // 👈 map 操作后执行
    map: d
    [A, D]
    ```

通过输出结果可以看到 `Sequence` 只有在使用时才执行 `map` 函数，而 `Iterable` 在使用前也就是调用 `map` 函数时就立即执行了。这样实现有什么好处呢？ `Iterable` 每调用一次函数就会生成一个新的 `Iterable`，下一个函数再基于新的 `Iterable` 执行，每次函数调用产生的临时 `Iterable` 会导致额外的内存消耗，而 `Sequence` 避免了这个问题。

##### 遍历函数执行顺序

再看一个例子：将一组字符串先转大写，然后判断其中有没有字母 B 开头的。

- `Iterable`：

    ``` kotlin
    🏝️
    val result = listOf("a", "bc", "d", "e", "f")
        .map {
            println("map: $it")
            it.toUpperCase()
        }
        .any { it.startsWith("B") }
    println(result)
    ```

    输出：

    ``` bash
    map: a
    map: bc
    map: d
    map: e
    map: f // 👈 5 次 map
    true
    ```

- `Sequence`：

    ``` kotlin
    🏝️
    val result = sequenceOf("a", "bc", "d", "e", "f")
        .map {
            println("map: $it")
            it.toUpperCase()
        }
        .any { it.startsWith("B") }
    println(result)
    ```

    输出：

    ``` bash
    map: a
    map: bc // 👈 2 次 map
    true
    ```

可以看到，`Sequence` 执行的 `map` 操作相比 `Iterable` 少了三次，避免了不必要的转换操作。

##### 对比

在数据比较少时，`Sequence` 相比 `Iterable` 的性能提升不怎么明显，而且为了实现懒加载反而有一些额外性能消耗，带来的收益不一定比成本大。使用的时候该怎么选，简单来说就是：

- 数据量大、遍历函数多，使用 `Sequence`。
- 数据量小、遍历函数少，使用 `Iterable`。

### 可见性修饰符

讲完了数据集合，再看看 Kotlin 中的可见性修饰符，Kotlin 中有四种可见性修饰符：`public` `private ` `protected` `internal`：

- `public `：公开，可见性最大，哪里都可以引用。
- `private`：私有，可见性最小，仅对所在类和所在文件可见。
- `protected`：保护，相当于 `private` + 子类可见。
- `internal`：内部，仅对 module 内可见。

相比 Java 少了一个包内可见修饰符，多了一个 `internal`「module 内可见」。这一节结合例子讲讲 Kotlin 这四种可见性修饰符，以及在 Kotlin 和 Java 中的不同。先来看看 `public`：

#### `public`

Java 中没写可见性修饰符时，表示包内可见，只有在同一个 `package` 内可以引用：

``` java
☕️                         👇
package org.kotlinmaster.library; 
// 没有可见性修饰符
class User {
}

```

``` java
☕️                         👇
package org.kotlinmaster.library;

public class Example {
    void method() {
        new User(); // success
    }
}
```

``` java
☕️
package org.kotlinmaster;
                       👆
import org.kotlinmaster.library.User; // compile-error
                          👆
public class OtherPackageExample {
    void method() {
        new User(); // compile-error: 'org.kotlinmaster.library.User' is not public in 'org.kotlinmaster.library'. Cannot be accessed from outside package
    }
}
```

`package` 外如果要引用，需要在 `class` 前加上可见性修饰符 `public` 表示公开。

Kotlin 中如果不写可见性修饰符，就表示公开，和 Java 中 `public` 修饰符具有相同效果。在 Kotlin 中也可以加上 `public` 修饰符，不过 IDE 会提示你删掉，因为默认就是 `public`。

#### `@hide`

在 Android 的官方 sdk 中，有一些方法只想对 sdk 内可见，不想开放给用户使用，因为这些方法不太稳定，在后续版本中很有可能会修改或删掉。为了实现这个特性，会在方法的注释中添加一个 Javadoc 方法 `@hide`，用来限制客户端访问：

``` java
☕️
/**
* @hide 👈
*/
public void hideMethod() {
    ...
}
```

但这种限制不太严格，可以通过反射访问到限制的方法。针对这个需求，Kotlin 引进了一个更为严格的可见性修饰符：`internal`。

#### `internal`

`internal` 表示修饰的类、方法仅对 module 内可见，这里的 module 具体指的是什么呢？它表示一组共同编译的 kotlin 文件，常见的形式有：

- Android Studio 里的 module
- Maven project

`internal` 在写一个 library module 时非常有用，当需要创建一个方法仅开放给 module 内部使用，但不想开放给使用者，因为后面可能会修改，这时就应该用  `internal` 可见性修饰符。

#### Java 的包内可见为什么没了？

Java 的包内可见在 Kotlin 中被弃用掉了，Kotlin 中与它最接近的可见性修饰符是 `internal`「module  内可见」。为什么会弃用掉包内可见？我觉得有这几个原因：

- Kotlin 鼓励创建 top-level 方法和属性，一个源码文件可以包含多个类，使得 Kotlin 的源码结构更加扁平化，包结构不再像 Java 中那么重要。
- 为了代码的解耦和可维护性，module 越来越多、越来越小，使得 `internal` 「module 内可见」已经可以满足对于代码封装的需求。

#### `protected`

- Java 中 `protected` 表示包内可见 + 子类可见。
- Kotlin 中 `protected` 表示 `private` + 子类可见。

可见 Kotlin 相比 Java `protected` 的可见范围收窄了，原因是 Kotlin 中不再有包内可见的概念了，相比 Java 的可见性着眼于 `package`，Kotlin 更关心的是 module。

#### `private`

- Java 中的 `private` 表示类中可见，外部包含类可见。
- Kotlin 中的 `private` 表示类中或所在文件内可见，外部包含类不可见。

`private` 在 Java 和 Kotlin 中的区别：

- 在 Java 中可以访问内部类的 `private` 变量：

  ``` java
  ☕️
  public class Outter {
  
      public static void method() {
          Inner inner = new Inner();
                              👇
          int result = inner.number * 2; // success
      }
                      👇
      private static class Inner {
            👇
          private int number = 0;
      }
  }
  ```

- 在 Kotlin 中是不允许的：

  ``` kotlin
  🏝️
  class Outter {
      
      fun method() {
          val inner = Inner()
                              👇
          val result = inner.number * 2 // compile-error: Cannot access 'number': it is private in 'Inner'
      }
       👇
      class Inner {
            👇
          private val number = 1
      }
  }
  ```

##### 可以修饰类和接口

- Java 中一个文件只允许一个外部类，所以 `class`  和 `interface` 不允许设置为 `private`，因为声明 `private`  后无法被使用，这样就没有意义了。

- Kotlin 允许同一个文件声明多个 `class` 和 top-level 的方法和属性，所以 Kotlin 中允许类和接口声明为 `private`，因为同个文件中的别的成员可以访问：

  ``` kotlin
  🏝️                   👇
  private interface Interface {
    👆
      fun method()
  }
  
    👇          👇       👇
  private class Impl : Interface {
      val number = 1
      override fun method() {
          println("Impl method()")
      }
  }
                   // 👇 在同一个文件中，所以可以访问
  private val impl = Impl()
  
  private val result = impl.number * 2
  ```

---

### 思考题

1. 次构造器写在初始化代码块之前，谁先执行？

2. `phoneCount` 和 `phoneCount1` 有什么区别？

   ``` kotlin
   🏝️
   class User {
       val phones = mutableListOf<String>()
       val phoneCount = phones.size
       val phoneCount1 get() = phones.size
   }
   ```

3. 下面这段代码有没有问题？为什么？

   ``` kotlin
   🏝️
   val list = listOf("a", "b", "c")
   list.toMutableList()
   list.add("d")
   ```

5. 同一个文件中，一个类的 `private` 属性可以被另一个类访问吗？
