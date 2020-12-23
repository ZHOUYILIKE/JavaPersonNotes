## Java 浅拷贝和深拷贝

### 介绍

开发过程中，有时会遇到把现有的一个对象的所有成员属性拷贝给另一个对象的需求。
比如说对象 A 和对象 B，二者都是 ClassC 的对象，具有成员变量 a 和 b，现在对对象 A 进行拷贝赋值给 B，也就是 B.a = A.a; B.b = A.b;

这时再去改变 B 的属性 a 或者 b 时，可能会遇到问题：假设 a 是基础数据类型，b 是引用类型。
当改变 B.a 的值时，没有问题；
当改变 B.b 的值时，同时也会改变 A.b 的值，因为其实上面的例子中只是把 A.b 赋值给了 B.b，因为是 b 引用类型的，所以它们是指向同一个地址的。这可能就会给我们使用埋下隐患。



根据对对象属性的拷贝程度（基本数据类和引用类型），会分为两种：

- 浅拷贝 (`Shallow Copy`)
- 深拷贝 (`Deep Copy`)

### 浅拷贝

#### 1. 浅拷贝介绍

浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。即默认拷贝构造函数只是对对象进行浅拷贝复制(逐个成员依次拷贝)，即只复制对象空间而不复制资源。

#### 2. 浅拷贝特点

(1) 对于基本数据类型的成员对象，因为基础数据类型是值传递的，所以是直接将属性值赋值给新的对象。基础类型的拷贝，其中一个对象修改该值，不会影响另外一个。
 (2) 对于引用类型，比如数组或者类对象，因为引用类型是引用传递，所以浅拷贝只是把内存地址赋值给了成员变量，它们指向了同一内存空间。改变其中一个，会对另外一个也产生影响。

![img](https://upload-images.jianshu.io/upload_images/8878793-61eb6dbc8885a9bc.png?imageMogr2/auto-orient/strip|imageView2/2/w/670)

#### 3. 浅拷贝的实现

实现对象拷贝的类，需要实现 `Cloneable` 接口，并覆写 `clone()` 方法。

示例如下：

```java
public class Subject {

    private String name;

    public Subject(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "[Subject: " + this.hashCode() + ",name:" + name + "]";
    }
}
```

```java
public class Student implements Cloneable {

    //引用类型
    private Subject subject;
    //基础数据类型
    private String name;
    private int age;

    public Subject getSubject() {
        return subject;
    }

    public void setSubject(Subject subject) {
        this.subject = subject;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     *  重写clone()方法
     * @return
     */
    @Override
    public Object clone() {
        //浅拷贝
        try {
            // 直接调用父类的clone()方法
            return super.clone();
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }

    @Override
    public String toString() {
        return "[Student: " + this.hashCode() + ",subject:" + subject + ",name:" + name + ",age:" + age + "]";
    }
}
```

```csharp
public class ShallowCopy {
    public static void main(String[] args) {
        Subject subject = new Subject("yuwen");
        Student studentA = new Student();
        studentA.setSubject(subject);
        studentA.setName("Lynn");
        studentA.setAge(20);
        Student studentB = (Student) studentA.clone();
        studentB.setName("Lily");
        studentB.setAge(18);
        Subject subjectB = studentB.getSubject();
        subjectB.setName("lishi");
        System.out.println("studentA:" + studentA.toString());
        System.out.println("studentB:" + studentB.toString());
    }
}
```

```java
studentA:[Student: 460141958,subject:[Subject: 1163157884,name:lishi],name:Lynn,age:20]
studentB:[Student: 1956725890,subject:[Subject: 1163157884,name:lishi],name:Lily,age:18]
```

由输出的结果可见，通过 `studentA.clone()` 拷贝对象后得到的 `studentB`，和 `studentA`  是两个不同的对象。`studentA` 和 `studentB` 的基础数据类型的修改互不影响，而引用类型 `subject` 修改后是会有影响的。

**浅拷贝和对象拷贝的区别：**

```java
public static void main(String[] args) {
        Subject subject = new Subject("yuwen");
        Student studentA = new Student();
        studentA.setSubject(subject);
        studentA.setName("Lynn");
        studentA.setAge(20);
        Student studentB = studentA;
        studentB.setName("Lily");
        studentB.setAge(18);
        Subject subjectB = studentB.getSubject();
        subjectB.setName("lishi");
        System.out.println("studentA:" + studentA.toString());
        System.out.println("studentB:" + studentB.toString());
    }
```

这里把 `Student studentB = (Student) studentA.clone()` 换成了 `Student studentB = studentA`。
 输出的结果：

```java
studentA:[Student: 460141958,subject:[Subject: 1163157884,name:lishi],name:Lily,age:18]
studentB:[Student: 460141958,subject:[Subject: 1163157884,name:lishi],name:Lily,age:18]
```

可见，对象拷贝后没有生成新的对象，二者的对象地址是一样的；而浅拷贝的对象地址是不一样的。

### 深拷贝

#### 1. 深拷贝介绍

通过上面的例子可以看到，浅拷贝会带来数据安全方面的隐患，例如我们只是想修改了 `studentB` 的 `subject`，但是 `studentA` 的 `subject` 也被修改了，因为它们都是指向的同一个地址。所以，此种情况下，我们需要用到深拷贝。

> 深拷贝，在拷贝引用类型成员变量时，为引用类型的数据成员另辟了一个独立的内存空间，实现真正内容上的拷贝。

#### 2. 深拷贝特点

(1) 对于基本数据类型的成员对象，因为基础数据类型是值传递的，所以是直接将属性值赋值给新的对象。基础类型的拷贝，其中一个对象修改该值，不会影响另外一个（和浅拷贝一样）。
 (2) 对于引用类型，比如数组或者类对象，深拷贝会新建一个对象空间，然后拷贝里面的内容，所以它们指向了不同的内存空间。改变其中一个，不会对另外一个也产生影响。
 (3) 对于有多层对象的，每个对象都需要实现 `Cloneable` 并重写 `clone()` 方法，进而实现了对象的串行层层拷贝。
 (4) 深拷贝相比于浅拷贝速度较慢并且花销较大。

结构图如下：

![img](https:////upload-images.jianshu.io/upload_images/8878793-858c3f7c993e4348.png?imageMogr2/auto-orient/strip|imageView2/2/w/670)

深拷贝图



#### 3. 深拷贝的实现

对于 `Student` 的引用类型的成员变量 `Subject` ，需要实现 `Cloneable` 并重写 `clone()` 方法。

```java
public class Subject implements Cloneable {

    private String name;

    public Subject(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        //Subject 如果也有引用类型的成员属性，也应该和 Student 类一样实现
        return super.clone();
    }

    @Override
    public String toString() {
        return "[Subject: " + this.hashCode() + ",name:" + name + "]";
    }
}
```

在 `Student` 的 `clone()` 方法中，需要拿到拷贝自己后产生的新的对象，然后对新的对象的引用类型再调用拷贝操作，实现对引用类型成员变量的深拷贝。

```java
public class Student implements Cloneable {

    //引用类型
    private Subject subject;
    //基础数据类型
    private String name;
    private int age;

    public Subject getSubject() {
        return subject;
    }

    public void setSubject(Subject subject) {
        this.subject = subject;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     *  重写clone()方法
     * @return
     */
    @Override
    public Object clone() {
        //深拷贝
        try {
            // 直接调用父类的clone()方法
            Student student = (Student) super.clone();
            student.subject = (Subject) subject.clone();
            return student;
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }

    @Override
    public String toString() {
        return "[Student: " + this.hashCode() + ",subject:" + subject + ",name:" + name + ",age:" + age + "]";
    }
}
```

一样的使用方式

```java
public class ShallowCopy {
    public static void main(String[] args) {
        Subject subject = new Subject("yuwen");
        Student studentA = new Student();
        studentA.setSubject(subject);
        studentA.setName("Lynn");
        studentA.setAge(20);
        Student studentB = (Student) studentA.clone();
        studentB.setName("Lily");
        studentB.setAge(18);
        Subject subjectB = studentB.getSubject();
        subjectB.setName("lishi");
        System.out.println("studentA:" + studentA.toString());
        System.out.println("studentB:" + studentB.toString());
    }
}
```

输出结果是：

```java
studentA:[Student: 460141958,subject:[Subject: 1163157884,name:yuwen],name:Lynn,age:20]
studentB:[Student: 1956725890,subject:[Subject: 356573597,name:lishi],name:Lily,age:18]
```

由输出结果可见，深拷贝后，不管是基础数据类型还是引用类型的成员变量，修改其值都不会相互造成影响。