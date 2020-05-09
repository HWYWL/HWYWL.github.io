---
title: 在Java中如何高效判断数组中是否包含某个元素
date: 2018-08-28 15:27:34
tags: 
 - Java
categories: 
 - Java
keywords: "Java"
description: 在Java中如何高效判断数组中是否包含某个元素。
---

如何检查一个数组(无序)是否包含一个特定的值？这是一个在Java中经常用到的并且非常有用的操作。

同时，这个问题在Stack Overflow中也是一个非常热门的问题。在投票比较高的几个答案中给出了几种不同的方法，但是他们的时间复杂度也是各不相同的。本文将分析几种常见用法及其时间成本。

### 使用List

```java
public static boolean useList(String[] arr, String targetValue) {
    return Arrays.asList(arr).contains(targetValue);
}
```

### 使用Set

```java
public static boolean useSet(String[] arr, String targetValue) {
    Set<String> set = new HashSet<String>(Arrays.asList(arr));
    return set.contains(targetValue);
}
```

### 使用循环判断

```java
public static boolean useLoop(String[] arr, String targetValue) {
    for(String s: arr){
        if(s.equals(targetValue))
            return true;
    }
    return false;
}
```

使用Arrays.binarySearch()

```
Arrays.binarySearch()方法只能用于有序数组！！！如果数组无序的话得到的结果就会很奇怪。
```
查找有序数组中是否包含某个值的用法如下：

```
public static boolean useArraysBinarySearch(String[] arr, String targetValue) { 
    int a =  Arrays.binarySearch(arr, targetValue);
    if(a > 0)
        return true;
    else
        return false;
}
```

# 时间复杂度
下面的代码可以大概的得出各种方法的时间成本。基本思想就是从数组中查找某个值，数组的大小分别是5、1k、10k。这种方法得到的结果可能并不精确，但是是最简单清晰的方式。


```
public static void main(String[] args) {
    String[] arr = new String[] {  "CD",  "BC", "EF", "DE", "AB"};

    //use list
    long startTime = System.nanoTime();
    for (int i = 0; i < 100000; i++) {
        useList(arr, "A");
    }
    long endTime = System.nanoTime();
    long duration = endTime - startTime;
    System.out.println("useList:  " + duration / 1000000);

    //use set
    startTime = System.nanoTime();
    for (int i = 0; i < 100000; i++) {
        useSet(arr, "A");
    }
    endTime = System.nanoTime();
    duration = endTime - startTime;
    System.out.println("useSet:  " + duration / 1000000);

    //use loop
    startTime = System.nanoTime();
    for (int i = 0; i < 100000; i++) {
        useLoop(arr, "A");
    }
    endTime = System.nanoTime();
    duration = endTime - startTime;
    System.out.println("useLoop:  " + duration / 1000000);

    //use Arrays.binarySearch()
    startTime = System.nanoTime();
    for (int i = 0; i < 100000; i++) {
        useArraysBinarySearch(arr, "A");
    }
    endTime = System.nanoTime();
    duration = endTime - startTime;
    System.out.println("useArrayBinary:  " + duration / 1000000);
}
```

运行结果：

```
useList:  13useSet:  72useLoop:  5useArraysBinarySearch:  9
```

# 使用一个长度为1k的数组

```
String[] arr = new String[1000];Random s = new Random();for(int i=0; i< 1000; i++){
    arr[i] = String.valueOf(s.nextInt());
}
```

结果：

```
useList:  112useSet:  2055useLoop:  99useArrayBinary:  12
```

# 使用一个长度为10k的数组

```
String[] arr = new String[10000];Random s = new Random();for(int i=0; i< 10000; i++){
    arr[i] = String.valueOf(s.nextInt());
}
```
结果：

```
useList:  1590useSet:  23819useLoop:  1526useArrayBinary:  12
```

# 总结
显然，使用一个简单的循环方法比使用任何集合都更加高效。许多开发人员为了方便，都使用第一种方法，但是他的效率也相对较低。因为将数组压入Collection类型中，首先要将数组元素遍历一遍，然后再使用集合类做其他操作。

如果使用Arrays.binarySearch()方法，数组必须是已排序的。由于上面的数组并没有进行排序，所以该方法不可使用。

实际上，如果你需要借助数组或者集合类高效地检查数组中是否包含特定值，一个已排序的列表或树可以做到时间复杂度为O(log(n))，hashset可以达到O(1)。

**（英文原文结束，以下是译者注）**

### 使用ArrayUtils
除了以上几种以外，Apache Commons类库中还提供了一个ArrayUtils类，可以使用其contains方法判断数组和值的关系。


```
public static boolean useArrayUtils(String[] arr, String targetValue) {
    return ArrayUtils.contains(arr,targetValue);
}
```

同样使用以上几种长度的数组进行测试，得出的结果是该方法的效率介于使用集合和使用循环判断之间（有的时候结果甚至比使用循环要理想）。


```
useList:  323useSet:  3028useLoop:  141useArrayBinary:  12useArrayUtils:  181-------useList:  3703useSet:  35183useLoop:  3218useArrayBinary:  14useArrayUtils:  3125
```

其实，如果查看ArrayUtils.contains的源码可以发现，他判断一个元素是否包含在数组中其实也是使用循环判断的方式。

部分代码如下：


```
if(array == null) {
    return -1;
} else {
    if(startIndex < 0) {
        startIndex = 0;
    }

    int i;
    if(objectToFind == null) {
        for(i = startIndex; i < array.length; ++i) {
            if(array[i] == null) {
                return i;
            }
        }
    } else if(array.getClass().getComponentType().isInstance(objectToFind)) {
        for(i = startIndex; i < array.length; ++i) {
            if(objectToFind.equals(array[i])) {
                return i;
            }
        }
    }

    return -1;
}
```

所以，相比较之下，我更倾向于使用ArrayUtils工具类来进行一些合数祖相关的操作。毕竟他可以让我少写很多代码（因为自己写代码难免有Bug，毕竟apache提供的开源工具类库都是经过无数开发者考验过的），而且，效率上也并不低太多。
