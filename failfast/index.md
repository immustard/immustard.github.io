# Fail-fast


<!--more-->



## 什么是**fail-fast**

[百度百科](https://baike.baidu.com/item/fail-fast/16329854)上是这么写的: `fail-fast`是Java集合(Collection)中的一种错误机制. 我觉得不太全面. 

下面来看一下[维基百科](https://en.wikipedia.org/wiki/Fail-fast)上这怎么写的: 

> In systems design, a **fail-fast** system is one which immediately reports at its interface any condition that is likely to indicate a failure. Fail-fast systems are usually designed to stop normal operation rather than attempt to continue a possibly flawed process. Such designs often check the system's state at several points in an operation, so any failures can be detected early. The responsibility of a fail-fast module is detecting errors, then letting the next-highest level of the system handle them.

从上面这段话就能看出来, `fail-fast`是在系统设计当中的一种**错误检测机制**, 一旦检测到**可能**发生错误, 就立即抛出异常, 程序不再继续运行. 



## 集合中的`fail-fast`

首先, 来复现这个错误: 

```java
List<String> nameList = new ArrayList<String>() {{
  add("张三");
  add("李四");
  add("小王");
  add("丑八怪");
}};

for (String name : nameList) {
  if (name.equals("丑八怪")) {
    nameList.remove(name);
  }
}
```

运行上面的代码就会抛出这样的异常: 

```log
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
```

上面的代码是在`for-each`中要删除集合中的元素而抛出的异常. 同样的, 增加(`add()`)元素也会抛出这个异常. 



## 异常产生的原因

来看一下源码: 

```java
public boolean add(E e) {
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  elementData[size++] = e;
  return true;
}

private void ensureCapacityInternal(int minCapacity) {
  ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
  modCount++;

	// ...
}

public boolean remove(Object o) {
  if (o == null) {
    for (int index = 0; index < size; index++)
      if (elementData[index] == null) {
        fastRemove(index);
        return true;
      }
  } else {
    for (int index = 0; index < size; index++)
      if (o.equals(elementData[index])) {
        fastRemove(index);
        return true;
      }
  }
  return false;
}

private void fastRemove(int index) {
  modCount++;
  // ...
}
```

从这段代码可以发现, 对`ArrayList`的`add()`, `remove()`, `clear()`方法, 只要涉及到改变集合中的元素的个数的方法都会导致`modCount`的改变, 但是没有对`expectedModCount`进行改变. 

```java
final void checkForComodification() {
  if (expectedModCount != modCount)
    throw new ConcurrentModificationException();
}
```

当`expectModCount`和`modCount`不同的时候, 就会抛出一开始出现的异常. 



## 避免产生异常

### 1. 使用普通`for`循环进行操作

```java
List<String> nameList = new ArrayList<String>() {{
  add("张三");
  add("李四");
  add("小王");
  add("丑八怪");
}};

for (int i = 0; i < nameList.size(); i++) {
  String name = nameList.get(i);

  if (name.equals("丑八怪")) {
    nameList.remove(name);
  }
}
```

这样虽然不会报错, 但是可能会产生漏删的情况出现. 



### 2. 使用`Iterator`进行操作

```java
List<String> nameList = new ArrayList<String>() {{
  add("张三");
  add("李四");
  add("小王");
  add("丑八怪");
}};

Iterator<String> iterator = nameList.iterator();

while (iterator.hasNext()) {
  if (iterator.next().equals("丑八怪")) {
    iterator.remove();
  }
}
```



### 3. 其实使用`for-each`也可以

```java
List<String> nameList = new ArrayList<String>() {{
  add("张三");
  add("李四");
  add("小王");
  add("丑八怪");
}};

for (String name : nameList) {
  if (name.equals("丑八怪")) {
    nameList.remove(name);
    break;
  }
}
```

当集合中只删除一个元素的时候, 在删除操作之后立即`break`, 使循环不进入下一次遍历就可以了. 



### 4. 使用`fail-safe`的集合类

### 5. 使用stream中的filter


---

> 作者:   
> URL: https://buli-home.cn/failfast/  

