## JAVA注解

### Java 自带注解（系统注解）


- @Override

表示重写注解

- @Deprecated

表示过时的方法

- @Suppvisewarnings

忽略某些警告

### 注解的分类

#### 按照运行机制分类：

- 源码注解

  注解只是在源码时存在，编译成class时就不存在了。

- 运行时注解

  在运行时注解仍然存在并且可能会影响程序逻辑。

- 编译时注解

  在源码和class文件中都存在。 


#### **元注解：**

　　元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：
　　	1.@Target
　　	2.@Retention
　　	3.@Documented
　　	4.@Inherited
　　这些类型和它们所支持的类在java.lang.annotation包中可以找到。

- **@Target：**

  ​**@Target**说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

  ​**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

  　**取值(ElementType)有：**

　　　　1.CONSTRUCTOR:用于描述构造器
　　　　2.FIELD:用于描述域
　　　　3.LOCAL_VARIABLE:用于描述局部变量
　　　　4.METHOD:用于描述方法
　　　　5.PACKAGE:用于描述包
　　　　6.PARAMETER:用于描述参数
　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

- **@Retention：**

  　**@Retention**定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

  　**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

  　**取值（RetentionPoicy）有：**

　　　　1.SOURCE:在源文件中有效（即源文件保留）
　　　　2.CLASS:在class文件中有效（即class保留）
　　　　3.RUNTIME:在运行时有效（即运行时保留）

- **@Documented:**

  ​**@Documented**用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

- **@Inherited：**

  ​**@Inherited** 元注解是一个标记注解，`@Inherited`阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

　　注意：`@Inherited annotation`类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

　　当`@Inherited annotation`类型标注的annotation的Retention是`RetentionPolicy.RUNTIME`，则反射API增强了这种继承性。如果我们使用`java.lang.reflect`去查询一个`@Inherited annotation`类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

### 顺便举个栗子

如何从任意一个对象中获取想要的值？

有这样一个注解声明

``` java
package com.company;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Created by zh on 16/6/5.
 */

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)

public @interface Banner {
    String value();
}
```

有这样一个业务类

``` java
package com.company;

/**
 * Created by zh on 16/6/5.
 */
public class Item {
    @Banner("url")
    public String url;
    @Banner("title")
    public String title;

    public String other;
}
```

获取Item对象指定值

``` java
package com.company;

import java.lang.reflect.Field;

public class Main {

    public static void main(String[] args) {
        // write your code here
        Item item = new Item();
        item.url = "http://www.mostring.com";
        item.title = "Hallo";
        printBanner(item);
        Bn b = new Bn("test","http://ola.so");
        printBanner(b);

    }

    public static void printBanner(Object object) {
        Class cls = object.getClass();
        Field[] fields = cls.getDeclaredFields();
        for (Field f : fields) {
            boolean isBannerAt = f.isAnnotationPresent(Banner.class);
            if (!isBannerAt) {
                continue;
            }
            Banner b = f.getAnnotation(Banner.class);
            f.setAccessible(true);
            try {
                Object vla = f.get(object);
                System.out.println(b.value() + "---" + vla);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }
}
```

[Java Annotation认知(包括框架图、详细介绍、示例说明)](http://www.cnblogs.com/skywang12345/p/3344137.html)