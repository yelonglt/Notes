#Java创建对象

###Java创建对象的五种方式
1. 使用new关键字，调用构造函数
2. 使用Class类的newInstance方法，调用构造函数
3. 使用Constructor类的newInstance方法，调用构造函数
4. 使用clone方法，没有调用构造函数
5. 使用反序列化，没有调用构造函数

### 示例对象类Student
```
package com.dmall.business.dto

public class Student {

    public String name;
    public int age;

    public Student() {
        
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Hello name:" + name + "-age:" + age;
    }
}
```

###使用new关键字
使用new关键字，我们可以调用任意无参和有参构造函数。

```
Student student = new Student();
Student student = new Student("YeLong",27);
```
###使用Class类的newInstance方法
首先获取Class对象的三种方法

```
Class<?> clazz = Student.class;
//类名的路径必须是全路径
Class<?> clazz = Class.forName("com.dmall.business.dto");
Class<?> clazz = ClassLoader.getSystemClassLoader().loadClass("com.dmall.business.dto");

```
使用newInstance方法只能调用无参构造函数

```
Class<Student> clazz = Student.class;
Student student = clazz.newInstance();
```

###使用Constructor类的newInstance方法
1. 先获取Class对象
2. 获取Constructor对象，调用newInstance方法。
3. 这个可以调用无参构造函数和有参构造函数

```
//调用无参构造函数
Constructor<?> constructor = clazz.getConstructor();
Student student = (Student) constructor.newInstance();

//调用有参构造函数
Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
Student student = (Student) constructor.newInstance("yelong", 12);

```

###使用clone方法
使用clone方法是进行深拷贝，没有调用构造方法。这里区分浅拷贝和深拷贝

1. 浅拷贝，就是两个对象指向同一引用（同一块内存地址）
2. 深拷贝，想要深拷贝一个对象，这个对象必须实现Cloneable接口实现clone方法，并且在clone方法内部将这个对象引用的其他对象也要clone一份。

```
Student student = new Student("YeLong", 27);
Student cStudent = (Student) student.clone();
```
###使用反序列化
当我们序列或者反序列化一个对象时，JVM会给我们创建一个单独的对象。在反序列化的时候，创建对象不会调用任何构造函数。

```
ObjectInputStream in = new ObjectInputStream(new FileInputStream("data.obj"));
Student student = (Student) in.readObject();
```
