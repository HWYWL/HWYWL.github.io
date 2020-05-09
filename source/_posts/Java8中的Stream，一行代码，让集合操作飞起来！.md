---
title: Java8中的Stream，一行代码，让集合操作飞起来！
date: 2019-01-13 11:43:34
tags: 
 - Java
 - lambda
categories: 
 - Java
keywords: "Java,lambda"
description: Java8中的Stream，一行代码，让集合操作飞起来！。
---

## 简介
java8也出来好久了，接口默认方法，lambda表达式，函数式接口，Date API等特性还是有必要去了解一下。比如在项目中经常用到集合，遍历集合可以试下lambda表达式，经常还要对集合进行过滤和排序，Stream就派上用场了。用习惯了，不得不说真的很好用。

Stream作为java8的新特性，基于lambda表达式，是对集合对象功能的增强，它专注于对集合对象进行各种高效、便利的聚合操作或者大批量的数据操作，提高了编程效率和代码可读性。

Stream的原理：将要处理的元素看做一种流，流在管道中传输，并且可以在管道的节点上处理，包括过滤筛选、去重、排序、聚合等。元素流在管道中经过中间操作的处理，最后由最终操作得到前面处理的结果。

集合有两种方式生成流：
- stream() − 为集合创建串行流
- parallelStream() - 为集合创建并行流

![image](00A4764C88EB41819AAB9C696DCAF249)

上图中是Stream类的类结构图，里面包含了大部分的中间和终止操作。

中间操作主要有以下方法（此类型方法返回的都是Stream）：map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

终止操作主要有以下方法：forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

## 举例说明
首先为了说明Stream对对象集合的操作，新建一个Student类（学生类）,覆写了equals()和hashCode()方法


```java
@Data
public class Student {

    private Long id;

    private String name;

    private int age;

    private String address;

    public Student() {}

    public Student(Long id, String name, int age, String address) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.address = address;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", address='" + address + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age &&
                Objects.equals(id, student.id) &&
                Objects.equals(name, student.name) &&
                Objects.equals(address, student.address);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, age, address);
    }
}
```

### filter（筛选）

```java
public static void main(String [] args) {
    Student s1 = new Student(1L, "肖战", 15, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);

    List<Student> streamStudents = testFilter(students);
    streamStudents.forEach(System.out::println);
}

/**
 * 集合的筛选
 * @param students
 * @return
 */
private static List<Student> testFilter(List<Student> students) {
    //筛选年龄大于15岁的学生
//        return students.stream().filter(s -> s.getAge()>15).collect(Collectors.toList());
    //筛选住在浙江省的学生
    return students.stream().filter(s ->"浙江".equals(s.getAddress())).collect(Collectors.toList());
}
```
运行结果：
![image](064F89CEBBDC4437B44DD3125529A4DA)

这里我们创建了四个学生，经过filter的筛选，筛选出地址是浙江的学生集合。

### map(转换)

```java
public static void main(String [] args) {
    Student s1 = new Student(1L, "肖战", 15, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);

    testMap(students);
}

/**
 * 集合转换
 * @param students
 * @return
 */
private static void testMap(List<Student> students) {
    //在地址前面加上部分信息，只获取地址输出
    List<String> addresses = students.stream().map(s ->"住址:"+s.getAddress()).collect(Collectors.toList());
    addresses.forEach(a ->System.out.println(a));
}
```
运行结果

![image](6D2F8F77F11549D1967417EB622A3D8A)

map就是将对应的元素按照给定的方法进行转换。

### distinct(去重)

```
public static void main(String [] args) {
  testDistinct1();
}

/**
 * 集合去重（基本类型）
 */
private static void testDistinct1() {
    //简单字符串的去重
    List<String> list = Arrays.asList("111","222","333","111","222");
    list.stream().distinct().forEach(System.out::println);
}
```
运行结果：
![image](F06DD4734C914CD29A6E1A270DE26D6B)


```
public static void main(String [] args) {
  testDistinct2();
}

/**
 * 集合去重（引用对象）
 */
private static void testDistinct2() {
    //引用对象的去重，引用对象要实现hashCode和equal方法，否则去重无效
    Student s1 = new Student(1L, "肖战", 15, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    Student s5 = new Student(1L, "肖战", 15, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);
    students.add(s5);
    students.stream().distinct().forEach(System.out::println);
}
```
运行结果：

![image](C11539ADDC1E430F8ED438AE06020BE2)

可以看出，两个重复的“肖战”同学进行了去重，这不仅因为使用了distinct()方法，而且因为Student对象重写了equals和hashCode()方法，否则去重是无效的。

### sorted(排序)
```
public static void main(String [] args) {
    testSort1();
}

/**
 * 集合排序（默认排序）
 */
private static void testSort1() {
    List<String> list = Arrays.asList("333","222","111");
    list.stream().sorted().forEach(System.out::println);
}
```
运行结果：

![image](9A7919666C0F47CE9E5BD3EB8E9290C0)


```
public static void main(String [] args) {
    testSort2();
}

/**
 * 集合排序（指定排序规则）
 */
private static void testSort2() {
    Student s1 = new Student(1L, "肖战", 15, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);
    students.stream()
            .sorted((stu1,stu2) ->Long.compare(stu2.getId(), stu1.getId()))
            .sorted((stu1,stu2) -> Integer.compare(stu2.getAge(),stu1.getAge()))
            .forEach(System.out::println);
}
```
运行结果：

![image](81C4A1F144444D94A87D790C4D5AD75D)

上面指定排序规则，先按照学生的id进行降序排序，再按照年龄进行降序排序

### limit（限制返回个数）

```
public static void main(String [] args) {
    testLimit();
}

/**
 * 集合limit，返回前几个元素
 */
private static void testLimit() {
    List<String> list = Arrays.asList("333","222","111");
    list.stream().limit(2).forEach(System.out::println);
}
```
运行结果：

![image](19C0E0221FDE405D8E0ED0CCA3E6BF58)

### skip(删除元素)

```
public static void main(String [] args) {
    testSkip();
}

/**
 * 集合skip，删除前n个元素
 */
private static void testSkip() {
    List<String> list = Arrays.asList("333","222","111");
    list.stream().skip(2).forEach(System.out::println);
}
```
运行结果：

![image](D06818C8704B4C80AEA807013CCBBA9F)

### reduce(聚合)

```
public static void main(String [] args) {
    testReduce();
}
/**
 * 集合reduce,将集合中每个元素聚合成一条数据
 */
private static void testReduce() {
    List<String> list = Arrays.asList("欢","迎","你");
    String appendStr = list.stream().reduce("北京",(a,b) -> a+b);
    System.out.println(appendStr);
}
```
运行结果：

![image](A9E5293DA8754B658839A11CBB609BEF)

### min(求最小值)


```
public static void main(String [] args) {
    testMin();
}

/**
 * 求集合中元素的最小值
 */
private static void testMin() {
    Student s1 = new Student(1L, "肖战", 14, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);
    Student minS = students.stream().min((stu1,stu2) ->Integer.compare(stu1.getAge(),stu2.getAge())).get();
    System.out.println(minS.toString());
}
```
运行结果：

![image](D7E4271D589941F99438A5D5A63FF2D3)
上面是求所有学生中年龄最小的一个，max同理，求最大值。

### anyMatch/allMatch/noneMatch（匹配）

```
public static void main(String [] args) {
    testMatch();
}

private static void testMatch() {
    Student s1 = new Student(1L, "肖战", 15, "浙江");
    Student s2 = new Student(2L, "王一博", 15, "湖北");
    Student s3 = new Student(3L, "杨紫", 17, "北京");
    Student s4 = new Student(4L, "李现", 17, "浙江");
    List<Student> students = new ArrayList<>();
    students.add(s1);
    students.add(s2);
    students.add(s3);
    students.add(s4);
    Boolean anyMatch = students.stream().anyMatch(s ->"湖北".equals(s.getAddress()));
    if (anyMatch) {
        System.out.println("有湖北人");
    }
    Boolean allMatch = students.stream().allMatch(s -> s.getAge()>=15);
    if (allMatch) {
        System.out.println("所有学生都满15周岁");
    }
    Boolean noneMatch = students.stream().noneMatch(s -> "杨洋".equals(s.getName()));
    if (noneMatch) {
        System.out.println("没有叫杨洋的同学");
    }
}
```
运行结果：

![image](B0478494832B4B2B8A4DAB6D063E4448)

anyMatch：Stream 中任意一个元素符合传入的 predicate，返回 true

allMatch：Stream 中全部元素符合传入的 predicate，返回 true

noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

## 总结
上面介绍了Stream常用的一些方法，虽然对集合的遍历和操作可以用以前常规的方式，但是当业务逻辑复杂的时候，你会发现代码量很多，可读性很差，明明一行代码解决的事情，你却写了好几行。试试lambda表达式，试试Stream，你会有不一样的体验。