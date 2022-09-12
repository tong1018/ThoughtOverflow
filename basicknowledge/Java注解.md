## Java注解

### 1、注解的定义和作用

#### 1.1 定义

> 注解是那些插入到源代码种使用其他工具可以对其进行处理的标签。这些工具可以在源码层次上进行操作，或者可以处理编译器在其中放置注解的类文件。
>
> 注解不会改变程序的编译方式。

Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种注释机制。Java 语言中的<font color='red'>类、方法、变量、参数和包等</font>都可以被标注。和 Javadoc 不同，Java 标注可以通过反射获取标注内容。在编译器生成类文件时，标注可以被嵌入到字节码中。Java 虚拟机可以保留标注内容，在运行时可以获取到标注内容 。 当然它也支持自定义 Java 标注。

#### 1.2 作用

>  注解是一个辅助工具，在JUnit、Struts、Spring灯工具框架中被广泛使用
>
>  - 附属文件的自动生成，例如部署描述符或者bean信息类
>  - 测试、日志、事物语义等代码的自动生成
>
>  注解可以在运行时处理，也可以在字节码级别上进行处理



### 2、注解的使用

#### 2.1 内置注解

##### 2.1.1 作用到代码的注解

| 注解名称          | 主要作用                                                     |
| ----------------- | ------------------------------------------------------------ |
| @Override         | 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误 |
| @Deprecated       | 标记过时方法。如果使用该方法，会报编译警告                   |
| @SuppressWarnings | 指示编译器去忽略注解中声明的警告                             |

> @SuppressWarnings源码如下：
>
> ```java
> @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
> @Retention(RetentionPolicy.SOURCE)
> public @interface SuppressWarnings {
>  String[] value();
> }
> ```
>
> 其中内部String[]主要接受值为：
>
> - deprecation：使用了不赞成使用的类或方法时的警告；
> - unchecked：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型;
> - fallthrough：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;
> - path：在类路径、源文件路径等中有不存在的路径时的警告;
> - serial：当在可序列化的类上缺少 serialVersionUID 定义时的警告;
> - finally：任何 finally 子句不能正常完成时的警告;
> - all：关于以上所有情况的警告。



##### 2.1.2 元注解

**@Target**

@Target 元注解可以应用于一个注解，以限制该注解可以应用到哪些项上。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface BugReport{ ... }
```

| 元素类型        | 注解使用场合                     | 元素类型       | 注解使用场合         |
| --------------- | -------------------------------- | -------------- | -------------------- |
| ANNOTATION_TYPE | 注解类型声明                     | FIELD          | 成员域(包括enum常量) |
| PACKAGE         | 包                               | PARAMETER      | 方法或者构造器参数   |
| TYPE            | 类(包括enum)及接口(包括注解类型) | LOCAL_VARIABLE | 局部变量             |
| METHOD          | 方法                             | TYPE_PARAMETER | 类型参数             |
| CONSTRUCTOR     | 构造器                           | TYPE_USE       | 类型用法             |

> 一条没有@Target限制的注解可以应用再任何项上。编译器将会检查你是否将一条注解只应用到了某个允许项上。

**@Retention**

@Retention 元注解用于指定一条注解应该保留多长时间

保留策略：

| 保留规则        | 技术       |                             描述                             |
| --------------- | ---------- | :----------------------------------------------------------: |
| SOURCE 源码级别 | APT        | 在编译期能够获取注解与注解声明的类，包括类中的成员信息，一般用于生成额外的辅助类。不包括在类文件中的注解。 |
| CLASS字节码     | 字节码插桩 | 在编译生成class之后，根据注解作为判断，通过修改class的数据来实现字节码级别的插桩操作。包括在类文件中的注解，但是虚拟机不需要将他们载入（默认策略） |
| RUNTIME 运行时  | 反射       | 在程序运行期间，通过反射技术动态获取注解与属性元素值，来完成逻辑操作。包括类文件中的注解，并有虚拟机载入。通过反射API可获得他们 |



**@Documented**

@Documented 元注解为像javadoc这样的归档工具提供了一些提示。

**@Inherited**

@Inherited 元注解只能应用于对类的注解。如果一个类具有继承注解，那么他的所有子类都会自动具有同样的注解。

> **Demo**
>
> ```java
> @Inherited
> @Documented
> @Target(ElementType.TYPE)
> @Retention(RetentionPolicy.RUNTIME)
> public @interface DocumentA {
> }
> 
> @Target(ElementType.TYPE)
> @Retention(RetentionPolicy.RUNTIME)
> public @interface DocumentB {
> }
> 
> @DocumentA
> class A{ }
> 
> class B extends A{ }
> 
> @DocumentB
> class C{ }
> 
> class D extends C{ }
> 
> //测试
> public class DocumentDemo {
> 
>  public static void main(String[] args) {
>      A instanceA = new B();
>      System.out.printf("used annotations of @Inherited: {%s}\n", Arrays.toString(instanceA.getClass().getAnnotations()));
> 
>      C instanceC = new D();
>      System.out.printf("unused annotations of @Inherited: {%s}", Arrays.toString(instanceC.getClass().getAnnotations()));
> 
>  }
> 
> /**
>   * 运行结果:
>     used annotations of @Inherited:[@com.thoughtoverflow.annotation.inherited.DocumentA()]
>     unused annotations of @Inherited: []
>   */
> }
> ```
>
> 由运行结果可以得知B继承了A中的注解

##### 2.1.3 额外注解

| 注解名称             | 主要作用                                                     |
| -------------------- | ------------------------------------------------------------ |
| @SafeVarargs         | Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告 |
| @FunctionalInterface | Java 8 开始支持，标识一个匿名函数或函数式接口                |
| @Repeatable          | Java 8 开始支持，标识某注解可以在同一个声明上使用多次        |

#### 2.2 注解语法

##### 2.2.1 Annotation通用定义

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
}

/**
1、使用@interface意味它这实现了java.lang.annotation.Annotation接口，@interface是必须的；(Annotaion接口的实现细节都由编译器完成。通过@interface定义注解后，该注解不能继承其他的注解或接口)
2、使用@Documented修饰该Annotation，则表示该注解将会出现在javadoc中；
3、使用@Target(ElementType.TYPE)，则表示该类是来修饰“类、接口(包括注释类型)或枚举声明”
4、使用@Retention(RetentionPolicy.RUNTIME)，意味着编译器会将该Annotation信息保留在.class文件中，并且能被虚拟机读取
**/
```

所有注解接口都隐式地拓展`java.lang.annotaion.Annotation`接口

##### 2.2.2 注解属性

注解的属性和类中定义的变量类似。注释注解中的变量都是成员变量，注解中是没有方法的，只有成员变量，变量名就是使用注解括号中对应的参数名，变量返回值注解括号中对应参数类型。

注解本质本身就是一个Annotation接口，注解中其实可以有属性和方法，但是接口中的属性都是static final的，对于注解没有什么意义，而我们定义接口的方法就相当于注解的属性。

**注解属性类型：**

- 基本类型：int、short、long、byte、char、double、float或者boolean
- String
- Class：具有一个可选的类型参数，例如Class<? extend MyClass>
- enum类型
- 注解类型
- 有前面所属类型组成的数组(有数组组成的数组不是合法的元素类型)

```java
// eg.注解元素类型
public @interface BugReport {
    enum Status { UNCONFRMED, CONFIRMED, FIXED, NOTABUG };
    // 基本数据类型
    boolean showStopper() default false;
    // String
    String assignedTo() default "[none]";
    // 具有可选类型的参数的类
    Class<?> testCase() default Void.class;
    // enum类型
    Status status() default Status.UNCONFIRMED;
    // 注解类型
    Reference ref() default @Reference(); // an annotation type
    // String类型数组
    String[] reportedBy();
}

```

##### 2.2.3 注解格式

每个注解格式：`@AnnotationName(elementName1 = value1, elementName2 = value2)`

注解中元素的顺序无关紧要，如果某个元素的值并未指定，那么久使用声明的默认值

> 默认值并不是和注解存储在一起的；相反地，他们是动态计算而来的。也因为注解是由编译器计算而来的， 因此，所有的元素值必须是编译期常量
>
> 一个注解元素永远不能设置为null，甚至不允许其默认值为null。
>
> 注解中引入循环依赖是一种错误。

一个项可以有多个注解，如果注解的作者将其声明为可重复的，那么可以多次重复使用同一个注解

##### 2.2.4 注解各类声明

注解可以出现在许多地方，这些地方可以分为两类：<font color='red'>声明和类型用法声明注解</font>

注解可以声明的位置：`包、类(包括enum)、接口(包括注解接口)、方法、构造器、实例域(包含enum常量)、局部变量、参数变量、类型参数`

- 对于类和接口，需要将注解放置在`class` 和`interface`关键词的前面

  ```java
  @Entity public class User {...} // 类
  ```

- 对于变量，需要将他们放置在类型的前面

  ```java
  @SuppressWarnings("unchecked") List<User> users = ...;
  public User getUser(@Param("id") String userId)
  ```

- 泛化类或者方法中的类型参数

  ```java
  public class Cache<Immutable V> {...}
  ```

- 局部变量

  > 对局部变量的注解<font color='red'>只能在源码级别上进行处理</font>。类文件并不描述局部变量。因此，所有的局部变量注解在编译完一个类的时候就会遗弃掉。

- 包——包的注解不能再源码界别之外存在

  ```java
  /**
  	Package-level javadoc
   */
  @GPL(version="3")
  package com.horstmann.corejava;
  import org.gnu.GPL;
  ```

#### 2.3 Annotation相关

> 注解的本质其实就是一个Annotation接口。

##### 2.3.1 Annotation框架

![image-20210207112136755](https://markdown-img-ct.oss-cn-beijing.aliyuncs.com/img/image-20210207112136755.png)

- 一个Annotation和一个RetentionPolicy关联——每个Annotation对象，都会有唯一的RetentionPolicy属性
- 一个Annotation和1~n个ElementType关联——对于每个Annotation对象，可以有若干个ElementType属性
- Annotation有许多实现类，包括：Deprecated, Documented, Inherited, Override 等等

##### 2.3.2 Annotation组成部分

Java中Annotation有三个非常重要的主干类组成：`Annotation`、`ElementType`、`RetentionPolicy`

- **Annotation.java**

  ```java
  package java.lang.annotation;
  
  public interface Annotation {
  
      boolean equals(Object obj);
  
      int hashCode();
  
      String toString();
  
      Class<? extends Annotation> annotationType();
  }
  ```

  > **Annotation就是一个接口**
  >
  > 每 1 个 Annotation 对象，都会有唯一的 RetentionPolicy 属性；至于 ElementType 属性，则有 1~n 个。

- **ElementType.java**

  ```java
  package java.lang.annotation;
  
  public enum ElementType {
      TYPE,               /* 类、接口（包括注释类型）或枚举声明  */
  
      FIELD,              /* 字段声明（包括枚举常量）  */
  
      METHOD,             /* 方法声明  */
  
      PARAMETER,          /* 参数声明  */
  
      CONSTRUCTOR,        /* 构造方法声明  */
  
      LOCAL_VARIABLE,     /* 局部变量声明  */
  
      ANNOTATION_TYPE,    /* 注释类型声明  */
  
      PACKAGE             /* 包声明  */
  }
  ```

  > **ElementType 是 Enum 枚举类型，它用来指定 Annotation 的类型**
  >
  > 当 Annotation 与某个 ElementType 关联时，就意味着：Annotation有了某种用途。例如，若一个 Annotation 对象是 METHOD 类型，则该 Annotation 只能用来修饰方法。

- **RetentionPolicy.java**

  ```java
  package java.lang.annotation;
  
  public enum RetentionPolicy {
      SOURCE,            /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */
  
      CLASS,             /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */
  
      RUNTIME            /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
  }
  ```

  > **RetentionPolicy 是 Enum 枚举类型，它用来指定 Annotation 的策略**
  >
  > 通俗点说，就是不同 RetentionPolicy 类型的 Annotation 的作用域不同，主要有SOURSE、CLASS、RUNTIME这三种策略。

##### 2.3.3 Annotation 作用

**1) 编译检查**

Annotation具有“让编译器进行编译检查的作用”。例如@SuppressWarnings, @Deprecated 和 @Override 都具有编译检查作用。

**2) 在反射中使用Annotation**

在反射的 Class, Method, Field 等函数中，有许多于 Annotation 相关的接口

**3) 根据Annotation生成帮助文档**

通过给 Annotation 注解加上 @Documented 标签，能使该 Annotation 标签出现在 javadoc 中。

**4) 帮助代码查看**

通过 @Override, @Deprecated 等，我们能很方便的了解程序的大致结构。

另外，我们也可以通过自定义 Annotation 来实现一些功能。

#### 2.4 注解与反射机制

##### 2.4.1 AnnotatedElement 接口

由前面章节可以得知Java中所有的注解都继承了Annotation接口，该接口是所有注解的父接口。为了运行时能够很好地准确的获取到注解的相关信息，Java在 `java.lang.reflect` 中添加了AnnotatedElement接口，通过该接口提供的方法可以利用反射技术地读取注解的信息，如反射包的Constructor类、Field类、Method类、Package类和Class类都实现了。

**AnnotatedElement API**

| 返回值                    | 方法名称                                                     | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| \<T extends Annotation> T | getAnnotation(Class\<T> annotationClass)                     | 该元素如果存在指定类型的注解，则返回这些注解，否则返回 null  |
| Annotation[]              | getAnnotations()                                             | 返回此元素上存在的所有注解，包括从父类继承的                 |
| boolean                   | isAnnotationPresent(Class<? extends Annotation> annotationClass) | 如果指定类型的注解存在于此元素上，则返回 true，否则返回 false |
| \<T extends Annotation> T | getDeclaredAnnotation(Class\<T> annotationClass)             | 如果直接存在此注释，则返回指定类型的该元素的注注解，否则为null。 此方法将忽略继承的注释。 （如果此元素上没有直接存在任何注释，则返回null。） |
| Annotation[]              | getDeclaredAnnotations()                                     | 返回直接存在于此元素上的所有注解，注意，不包括父类的注解，调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响，没有则返回长度为0的数组 |

**Demo**

```java
package com.thoughtoverflow.annotation;

import java.lang.annotation.Annotation;
import java.util.Arrays;

/**
 * 测试AnnotatedElement接口
 * @author Tommy Chen
 */
@DocumentB
public class DocumentReflectDemo extends A{

    public static void main(String[] args) {
        Class<?> clazz = DocumentReflectDemo.class;

        // 获取DocumentReflectDemo的
        DocumentA documentA = clazz.getAnnotation(DocumentA.class);
        System.out.printf("A:%s\n", documentA.toString());

        // 获取该元素上的所有注解，包含从父类继承
        Annotation[] annotations1 = clazz.getAnnotations();
        System.out.printf("annotations1:%s\n", Arrays.toString(annotations1));

        // 获取该元素上的所有注解，不包含继承
        Annotation[] annotations2 = clazz.getDeclaredAnnotations();
        System.out.printf("annotations2:%s\n", Arrays.toString(annotations2));

        // 判断注解DocumentA是否在该元素上
        boolean hasAnnotationA = clazz.isAnnotationPresent(DocumentA.class);
        System.out.printf("hasAnnotationA:%s\n", hasAnnotationA);
        
        /**
        运行结果:
        A:@com.thoughtoverflow.annotation.DocumentA()
		annotations1:[@com.thoughtoverflow.annotation.DocumentA(), 		@com.thoughtoverflow.annotation.DocumentB()]
		annotations2:[@com.thoughtoverflow.annotation.DocumentB()]
hasAnnotationA:true
        **/

    }
}
```

#### 2.5 运行时处理器

##### 2.5.1 Demo 利用运行时注解组装数据库SQL的构建语句过程

参考《Thinking in Java》

**自定义注解**

- @DBTable：将数据库表对应的实例类标识出来
- @Constraints：对数据库表进行属性的约束，主要约束项有是否为主键约束，是否为null，是否唯一
- @SQLString：将数据库表中对应的属性标识出来，该注解针对varchar类型

```java
/**
 * 约束注解，对字段进行约束
 * created on 2021/2/8.
 * @author Tommy Chen
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {

    // 判断是否作为主键约束
    boolean primaryKey() default false;
    // 判断是否允许为null
    boolean allowNull() default false;
    // 判断是否唯一
    boolean unique() default false;
}

/**
 * 数据库表注解
 * created on 2021/1/8.
 * @author Tommy Chen
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DBTable {
    String name() default "";
}

/**
 * 注解Integer类型的字段
 * Created on 2021/1/8.
 * @author Tommy Chen
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLInteger {
    // 该字段对应数据库表列名
    String name() default "";

    // 嵌套注解
    Constraints constraints() default @Constraints;
}

/**
 * 注解String类型的字段
 * Created on 2021/1/8.
 * @author Tommy Chen
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    // 该字段对应数据库表列名
    String name() default "";

    // 列类型分配的长度，如varchar(30)的30
    int value() default 0;

    // 嵌套注解
    Constraints constraints() default @Constraints;
}
```

**实例类Bean**

对应数据库中表的实例类，通过注解进行属性标识以及约束

```java
package com.thoughtoverflow.annotation.sqldemo;

/**
 * 数据库表Member对应的实例类bean
 * created on 2021/2/8
 * @author Tommy Chen
 */
@DBTable(name = "MEMBER")
public class Member {
    // 主键ID
    @SQLString(name = "ID", value = 50, constraints = @Constraints(primaryKey = true))
    private String id;

    // 姓名
    @SQLString(name = "NAME", value = 30)
    private String name;

    // 年龄
    @SQLInteger(name = "AGE")
    private Integer age;

    // 个人描述
    @SQLString(name = "DESCRIPTION", value = 150, constraints = @Constraints(allowNull = true))
    private String description;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}

```

**注解处理器**

通过反射机制获取被注解标识的数据库Bean实例类，同时将改类中被注解标识且约束的属性进行读取，最后生成构造数据库表的SQL语句

```java
package com.thoughtoverflow.annotation.sqldemo;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

/**
 * 运行时注解处理器，构造表创建语句
 * created on 2021/2/8
 * @author Tommy Chen
 */
public class TableCreator {

    /**
     *
     * @param className
     * @return
     * @throws ClassNotFoundException
     */
    public static String createTableSQL(String className) throws ClassNotFoundException {
        Class<?> clazz = Class.forName(className);
        DBTable dbTable = clazz.getAnnotation(DBTable.class);
        if(dbTable == null) {
            System.out.printf("No DBTable annotations in class %s", className);
            return null;
        }

        // 获取注解中指定的数据库表名
        String tableName = dbTable.name();
        if(tableName.length() < 1)
            tableName = clazz.getName().toUpperCase();

        // 获取表属性
        List<String> columnDefs = new ArrayList<>();
        for (Field field : clazz.getDeclaredFields()) {
            String columnName = null;
            Annotation[] annotations = field.getDeclaredAnnotations();
            if(annotations.length < 1)
                continue; // Not a db table column
            if(annotations[0] instanceof SQLInteger) {
                SQLInteger sqlInteger = (SQLInteger) annotations[0];

                if(sqlInteger.name().length() < 1)
                    columnName = field.getName().toUpperCase();
                else
                    columnName = sqlInteger.name();
                // 确定该属性以及属性约束
                columnDefs.add(columnName + " INT" +
                        getConstraints(sqlInteger.constraints()));

            }

            //判断String类型
            if(annotations[0] instanceof SQLString) {
                SQLString sString = (SQLString) annotations[0];
                // Use field name if name not specified.
                if(sString.name().length() < 1)
                    columnName = field.getName().toUpperCase();
                else
                    columnName = sString.name();
                columnDefs.add(columnName + " VARCHAR(" +
                        sString.value() + ")" +
                        getConstraints(sString.constraints()));
            }

        }
        // 数据库表构建语句
        StringBuilder createCommand = new StringBuilder(
                "CREATE TABLE " + tableName + "(");
        for(String columnDef : columnDefs)
            createCommand.append("\n    ").append(columnDef).append(",");

        // Remove trailing comma
        return createCommand.substring(
                0, createCommand.length() - 1) + ");";
    }

    /**
     * 判断该字段是否有其他约束
     * @param con
     * @return String
     */
    private static String getConstraints(Constraints con) {
        String constraints = "";
        if(!con.allowNull())
            constraints += " NOT NULL";
        if(con.primaryKey())
            constraints += " PRIMARY KEY";
        if(con.unique())
            constraints += " UNIQUE";
        return constraints;
    }

    public static void main(String[] args) throws Exception {
        String[] arg={"com.thoughtoverflow.annotation.sqldemo.Member"};
        for(String className : arg) {
            System.out.println("Table Creation SQL for " +
                    className + " is :\n" + createTableSQL(className));
        }
    }
}
```

**测试结果**

```java
Table Creation SQL for com.thoughtoverflow.annotation.sqldemo.Member is :
CREATE TABLE MEMBER(
    ID VARCHAR(50) NOT NULL PRIMARY KEY,
    NAME VARCHAR(30) NOT NULL,
    AGE INT NOT NULL,
    DESCRIPTION VARCHAR(150));
```



### 3、注解处理器APT(Annotation Processor Tool)

#### 3.1 概念

注解处理器是(Annotation Processor)是`javac`的一个工具，用来在编译时扫描和编译和处理注解(Annotation)。一个注解处理器它以Java代码或者(编译过的字节码)作为输入，生成文件(通常是java文件)。这些生成的java文件不能修改，并且会同其手动编写的java代码一样会被`javac`编译。过程总的来说为：==标记了注解的类，变量等作为输入内容，经过注解处理器处理，生成想要生成的java代码。==

#### 3.2 运行方式以及运行阶段

`.java -> javac -> .class`

- `.java`文件需要通过`javac`编译成`.class`文件
- 采集所有注解信息
- 将采集的信息包装成元素节点Element
- <font color='red'>由`javac`去调起注解处理器</font>，执行自定义的注解处理程序
- 注解处理程序实在`.class`文件被`javac`编译成字节码文件之前执行的



### 4、相关面试题



### Reference

[《Java核心技术 卷Ⅱ：高级特性》第八章 脚本、编译和注解处理](https://item.jd.com/12791368.html)

[深入理解Java注解类型(@Annotation)](https://blog.csdn.net/javazejian/article/details/71860633)

[菜鸟教程：Java 注解（Annotation）](