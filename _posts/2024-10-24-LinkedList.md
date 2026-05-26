---
layout:     post
title:      LinkedList
subtitle:   Java学习笔记
date:       2024-10-24
author:     DM
header-img: img/post-bg-debug.png
catalog:    true
tags:       Java Mac Spring
---

LinkedList 的底层是双向链表结构，由于链表没有将元素存储在连续的空间中，而是存储在单独的节点中，然后通过引用将节点连接起来了，因此在任意位置插入或者删除元素时，不需要搬移元素，效率比较高。

***

注意：

*   LinkedList实现了list 接口
*   LinkedList的底层是用了双向链表
*   LinkedList没有实现RandomAcess接口，因此LinkedList不支持随机访问
*   LinkedList的任意位置插入和删除元素时效率比较高，时间复杂度为 0（1）
*   LinkedList比较适合任意位置插入的元素

### &#x20;LinkedList 和 ArrayList 的区别

*   在存储空间上，ArrayList在物理上一定连续， LinkedList在逻辑上连续，但物理上不一定连续
*   ArrayList支持随机访问，LinkedList 不支持随机访问
*   ArrayList 在头插入时需要搬移元素， LinkedList只需要修改引用的指向
*   ArrayList 在插入时空间不足 需要扩容， LinkedList 没有容量的概念
*   ArrayList 应用于“元素高效存储和频繁访问”， LinkedList 应用于“任意位置插入和删除频繁”

## 泛型// 泛型类

```java
GenericTest gtest = new GenericTest();
        gtest.list.add("zhangsan");
        System.out.println(gtest.list);

        GenericTest111<String, Integer> gtest111 = new GenericTest111<>();
        gtest111.list.add("zhangsan");
        gtest111.list111.add(29);
        System.out.println(gtest111.list);
        System.out.println(gtest111.list111);

class GenericTest {
    public static List<String> list = new ArrayList<>();

}
// 泛型类
class GenericTest111<K, E> {
    public List<K> list = new ArrayList<>();
    public List<E> list111 = new ArrayList<>();
}

```

## &#x20;set 去重        &#x20;

```java
		int[] numArr = { 1, 2, 3, 4, 5, 6, 2, 5 };
        HashSet<Integer> set = new HashSet<>();
        for (int i = 0; i < numArr.length; i++) {
            set.add(numArr[i]);
        }
        System.out.println(set);
        System.out.println(set.size());//统计处不重复的数字的个数
```

## &#x20;TreeSet 排序

```java
  // TreeSet的自动排序：原理是二叉树平衡法
        TreeSet<Integer> treeNum = new TreeSet<>();
        treeNum.add(33);
        treeNum.add(77);
        treeNum.add(44);
        treeNum.add(12);
        System.out.println(treeNum);
        //[12, 33, 44, 77]
```

## TreeSet自定义排序   &#x20;

```java
//TreeSet自定义排序
        TreeSet<Student> stuSet = new TreeSet<>(new Student.StudentCompForAge());
        stuSet.add(new Student("陆小凤", 20));
        stuSet.add(new Student("西门", 22));
        stuSet.add(new Student("花满楼", 21));
        stuSet.add(new Student("雪西门吹", 19));
        System.out.println(stuSet);
        //[姓名：雪西门吹, 年龄：19, 姓名：陆小凤, 年龄：20, 姓名：花满楼, 年龄：21, 姓名：西门, 年龄：22]
```

```java
Student类内容如下：
public class Student {
    int age;
    String name;

    Student() {
    }

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String toString() {
        return "姓名：" + name + ", 年龄：" + age;
    }

    // 根据age来升序排序的比较器
    static class StudentCompForAge implements Comparator {
        @Override
        public int compare(Object o1, Object o2) {
            Student u1 = (Student) o1;
            Student u2 = (Student) o2;
            // 注意：此处用自定义类的age来实施比较，
            // 所以age值相同的元素不能被加入成功
            return u1.age - u2.age;
        }
    }
}
```
