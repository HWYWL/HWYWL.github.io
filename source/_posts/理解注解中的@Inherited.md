---
title: 理解注解中的@Inherited
date: 2019-07-03 9:25:34
tags: 
 - Java
 - 注解
categories: 
 - Java
keywords: "Java,注解"
description: 理解注解中的@Inherited。
---
## @Inherited： 
   @Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。 
如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。 

**注意**：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation， 方法并不从它所重载的方法继承annotation。 

　　当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。 如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作： 检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。 

看下面的例子： 
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface ATable {

	public String name() default "";
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface BTable {
	public String name() default "";
}


@ATable
public class Super {
	private int superx;
	public int supery;
	public Super() {
	}
    private int superX(){  
        return 0;  
    }  
    public int superY(){  
        return 0;  
    }  
    
}


@BTable
public class Sub extends Super{
	private int subx;
	public int suby;
	private Sub()
	{  
	}  
    public Sub(int i){  
    }  
    private int subX(){  
        return 0;  
    }  
    public int subY(){  
        return 0;  
    }  
}



public class TestMain {
	public static void main(String[] args) {
		
		Class<Sub> clazz = Sub.class;
	    
		System.out.println("============================Field===========================");  
		System.out.println(Arrays.toString(clazz.getFields()));
        System.out.println(Arrays.toString(clazz.getDeclaredFields()));  //all + 自身  
        System.out.println("============================Method===========================");
        System.out.println(Arrays.toString(clazz.getMethods()));   //public + 继承  
        //all + 自身  
        System.out.println(Arrays.toString(clazz.getDeclaredMethods()));
        System.out.println("============================Constructor===========================");  
        System.out.println(Arrays.toString(clazz.getConstructors()));  
                System.out.println(Arrays.toString(clazz.getDeclaredConstructors()));  
        System.out.println("============================AnnotatedElement===========================");  
        //注解DBTable2是否存在于元素上  
        System.out.println(clazz.isAnnotationPresent(BTable.class));  
        //如果存在该元素的指定类型的注释DBTable2，则返回这些注释，否则返回 null。  
        System.out.println(clazz.getAnnotation(BTable.class));  
        //继承  
        System.out.println(Arrays.toString(clazz.getAnnotations()));  
        System.out.println(Arrays.toString(clazz.getDeclaredAnnotations()));  ////自身  
	}
}
```

分析下这段代码，这里定义了两个annotion，其中ATable使用了@Inherited, BTable没有使用  @Inherited,类Super和类Sub分别使用了ATable和BTable这两个注解，并且Sub类 继承Super类。 

这段程序的运行结果如下： 
```java
============================Field===========================
[public int annotion.inherit.Sub.suby, public int annotion.inherit.Super.supery]
[private int annotion.inherit.Sub.subx, public int annotion.inherit.Sub.suby]
============================Method===========================
[public int annotion.inherit.Sub.subY(), public int annotion.inherit.Super.superY(), public final void java.lang.Object.wait() throws java.lang.InterruptedException, public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]
[private int annotion.inherit.Sub.subX(), public int annotion.inherit.Sub.subY()]
============================Constructor===========================
[public annotion.inherit.Sub(int)]
[private annotion.inherit.Sub(), public annotion.inherit.Sub(int)]
============================AnnotatedElement===========================
true
@annotion.inherit.BTable(name=)
[@annotion.inherit.ATable(name=), @annotion.inherit.BTable(name=)]
[@annotion.inherit.BTable(name=)]
```

getFields()获得某个类的所有的公共（public）的字段，包括父类。 

getDeclaredFields()获得某个类的所有申明的字段，即包括public、private和proteced， 
但是不包括父类的申明字段。 同样类似的还有getConstructors()和getDeclaredConstructors()， getMethods()和getDeclaredMethods()。 

因此：Field的打印好理解，因为sub是super类的子类，会继承super的类 
同样method和constructor的打印也是如此。 

clazz.getAnnotations()可以打印出当前类的注解和父类的注解 
clazz.getDeclaredAnnotations()只会打印出当前类的注解 

如果注解ATable把@Inherit去掉。那么后面四行的输出结果为： 
```java
true
@annotion.inherit.BTable(name=)
[@annotion.inherit.BTable(name=)]
[@annotion.inherit.BTable(name=)]
```

**无法获取到@ATable的注解， 也就是说注解和普通类的区别是如果一个子类想获取到父类上的注解信息， 那么必须在父类上使用的注解上面 加上@Inherit关键字 **