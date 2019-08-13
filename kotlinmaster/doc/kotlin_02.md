## Kotlin 里那些「不是那么写的」

上期我们讲了 Kotlin 上手最基础的三个点：变量、函数和类型。大家都听说过，Kotlin 完全兼容 Java，这个意思是用 Java 写出来的 class 和 Kotlin 可以完美交互，而不是说你用 Java 的写法去写 Kotlin 也完全没问题，这个是不行的。这期内容我们就讲一下，Kotlin 里那些「不 Java」的写法。

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
     //👆 没有 public
            this.id = id
            this.name = name
        }
    }
    ```

可以发现有两点不同：

- Java 中构造器和类同名，Kotlin 中使用 `constructor` 表示。
- Kotlin 构造器没有 public 修饰，因为默认可见性就是公开的，关于可见性修饰符这里先不展开，后面会讲到。

#### init

还有，Java 里常常配合构造方法一起使用的 init 代码块，在 Kotlin 里的写法也有了一点点改变：你需要给它加一个 init 前缀。

- Java

    ``` java
    ☕️
    public class User {
       👇
        {
            // Java init block，第一步
        }
        public User() {
            // 构造器，第二步
        }
    }
    ```

- Kotlin

    ``` kotlin
    🏝️
    class User {
        👇
        init {
            // Kotlin init block，第一步
        }
        constructor() {
            // 构造器，第二步
        }
    }
    ```

Kotlin 的 init 代码块和 Java 一样，都在实例化时执行，并且执行顺序都在构造器之前。

### `final`

上一篇在「 `var` 和 `val`」 这一节中讲到 Kotlin 中的 `val` 和 Java 中的 `final` 类似，表示只读变量，不能修改。这里具体对比下 Java 和 Kotlin 中的只读变量：

- Java

  ``` java
  ☕️
   👇
  final int final1 = 1;
               👇  
  void method(final String final2) {
       👇
      final String final3 = "The parameter is " + final2;
  }
  ```
  
- Kotlin

  ``` kotlin
  🏝️
  👇
  val fina1 = 1
         // 👇 没有 val
  fun method(final2: String) {
      👇
      val final3 = "The parameter is " + final2
  }
  ```

可以看到不同点主要有：

- final 变成 val。
- 由于类型推断类型可以省略不写，写法上简短了一些。
- Kotlin 方法参数默认是 val 类型，所以参数前不需要写 val 关键字。

上一期说过，`var` 是 variable 的缩写， `val` 是 value 的缩写。

其实我们写 Java 代码的时候，很少会有人用 `final` 的对吧？但是我从 `final` 开始说起，是因为它是一个很好的切入点，来让我们去看一看 Kotlin 里那些「不 Java」的写法。所以稍后几分钟的内容，也都是从 `final` 来展开的。

我们继续说。`final` 用来修饰变量其实是很有用的，但大家都不用；可你如果去看看国内国外的人写的 Kotlin 代码，你会发现很多人的代码里都会有一堆的 `val`。而且这些写 `val` 的人，就是当初那些不写 `final` 的人。为什么？因为 `final`写起来比 `val` 麻烦一点：我需要多写一个单词。虽然只麻烦这一点点，但就导致很多人不写。

这就是一件很有意思的事：从 `final` 到 `val`，只是方便了一点点，但却让它们的使用频率有了巨大的改变。这种改变是会影响到代码质量的：在该加限制的地方加上限制，就可以减少代码出错的概率。

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

不过这个属于特殊用法，一般情况下 `val` 还是对应于 Java 中的 `final` 使用的。

### `static` property / function

刚才说到大家都不喜欢写 `final` 对吧？但有一种场景，大家是最喜欢用 `final` 的：常量。

``` java
☕️
public static final String CONST_STRING = "A String"
```

在 Java 里面写常量，我们用的是 `static` + `final`。而在 Kotlin 里面，除了 `final` 的写法不一样，`static` 的写法些也不一样，而且是更不一样。确切地说：在 `Kotlin` 里，静态变量和静态方法这两个概念被去除了。

那如果想在 Kotlin 中像 Java 一样通过类直接引用该怎么办呢？Kotlin 的答案是 `companion object`：

``` kotlin
🏝️
class Sample {
    ...
       👇
    companion object {
        val anotherString = "Another String"
    }
}
```

哈？为啥 Kotlin 越改越复杂了？不着急，我们先看看 `object` 是个什么东西。

#### `object`

Kotlin 里的 `object` ——首字母小写的，不是大写，Java 里的 `Object` 在 Kotlin 里不用了。

> Java 中的 `Object`  在 Kotlin 中换成了 `Any`，和 `Object` 作用一样：作为所有类的基类。

而 `object` 不是类，像 `class` 一样在 Kotlin 中属于关键字：

``` kotlin
🏝️
object Sample {
    ...
}
```

它的意思很直接：创建一个类，并且创建一个这个类的对象。这个就是 `object` 的意思：对象。

在代码中如果要使用这个对象，直接通过它的类名就可以访问：

``` kotlin
🏝️
Sample.name
```

这不就是单例么，所以在 Kotlin 中创建单例不用像 Java 中那么复杂，只需要把 `class` 换成 `object` 就可以了。

##### 单例类

我们看一个单例的例子，分别用 Java 和 Kotlin 实现的代码：

- Java 中的单例类：

    ``` java
    ☕️
    public class A {
        private static A sInstance;
        
        public static A getInstance() {
            if (sInstance == null) {
                sInstance = new A();
            }
            return sInstance;
        }
    
        private A() {
        }
    
        public int number = 1;
    
        public void method() {
            System.out.println("A.method()");
        }
    }
    ```
    
可以看到 Java 中为了实现单例类写了大量的模版代码，稍显繁琐。 
    
- Kotlin 中实现单例类：

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

    和类的定义类似，不过 `class` 关键字替换成 `object`，相比 Java 的实现简单多了。


##### 匿名类

除了单例类，Kotlin 还可以创建 Java 中的匿名类，只是写法上有点不同：

- Java：

    ``` java
    ☕️                                              👇 
    ViewPager.SimpleOnPageChangeListener listener = new ViewPager.SimpleOnPageChangeListener() {
    	@Override // 👈
    	public void onPageSelected(int position) {
    		// override
    	}
    };
    ```

- Kotlin：

    ``` kotlin
    🏝️              // 👇 没有 object 名字
    val listener = object: ViewPager.SimpleOnPageChangeListener() {
    		   // 👆 「= 和 object: 」共同组成对象表达式
        override fun onPageSelected(position: Int) {
            // override
        }
    }
    ```

    和 Java 创建匿名类的方式很相似，Kotlin 中这种写法称之为「对象表达式」，对象就是指 `object` 及后面修饰的部分，表达式就指的是 `=`。所以 `object` 及后面的部分不能单独存在，如果没有 `=` 以及前面的变量，这段代码就不能被认为是对象表达式，就会报错：

    ``` kotlin
    🏝️
      // 👇 compile error: Name expected
    object: ViewPager.SimpleOnPageChangeListener() {
        override fun onPageSelected(position: Int) {
            // override
        }
    }
    ```

    编译器提示 `object` 后需要一个对象的名字，因为编译器以为你想创建一个继承 `ViewPager.SimpleOnPageChangeListener` 的对象，而不是一个对象表达式。

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

前面讲到 Kotlin 有类的初始化代码，那有没有 Java 中的静态初始化代码呢？答案是有的，只是不能像 Java 那样放在类中，而是要像静态属性和方法一样放在 `companion object` 中：

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

##### 继承类和实现接口

类可以继承别的类，可以实现接口，那 `object` 可以吗？答案是可以的：

``` kotlin
🏝️
open class A {
    open fun method() {
        ...
    }
}

interface B {
    fun interfaceMethod()
}
  👇      👇   👇
object C : A(), B {

    override fun method() {
        ...
    }

    override fun interfaceMethod() {
        ...
    }
}

class D {          // 👇
    companion object : B {
        override fun interfaceMethod() {
            ...
        }
    }
}
```

#### top-level property / function 声明

除了静态方法这种简便的调用方式，Kotlin 还有更方便的东西：「`top-level declaration` 顶层声明」。其实就是把属性和函数的声明不写在 `class` 里面，这个在 Kotlin 里是允许的：

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
    String name; // 👈 可修改
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

Kotlin 和 Java 一样有三种集合类型：List、Set 和 Map，它们的含义分别如下：

- `List` 以固定顺序存储一组元素，元素可以重复。
- `Set` 存储一组互不相等的元素，通常没有固定顺序。
- `Map` 存储 键-值 对的数据集合，键互不相等，但不同的键可以对应相同的值。

从 Java 到 Kotlin，这三种集合类型的使用有哪些变化呢？我们依次看看。

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

2. `phoneCount` 和 `phoneCount1` 有什么区别？

   ``` kotlin
   🏝️
   class User {
       val phones = mutableListOf<String>()
       val phoneCount = phones.size
       val phoneCount1 get() = phones.size
   }
   ```

5. 同一个文件中，一个类的 `private` 属性可以被另一个类访问吗？
