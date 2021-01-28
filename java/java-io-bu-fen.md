# java IO 部分

### Java IO包采用设计模式

Java IO采用的是装饰器模式去设计的，使用过Java IO相关的功能可能都有疑问？为什么Java IO要用装饰器模式，比如下面这段代码：

```

InputStream in = new FileInputStream("test.txt");
InputStream bin = new BufferedInputStream(in);
byte[] data = new byte[1024];
while (bin.read(data) != -1) {
  //...
}

```
这段代码看着就是有点繁琐，为什么不能直接采用继承方式呢，比如这么写：
```
InputStream in = new BufferedInputStream("test.txt");
byte[] data = new byte[1024];
while (bin.read(data) != -1) {
  //...
}

```
继承比较单一，而且不易于扩展，如果以后再出一个支持读取基本数据类型的、支持字符串读写的等等，那么类继承会变得非常复杂，比如现在需要一个带有buffer效果且能读取基本数据类型的用装饰器就能很好实现，但是继承是无法做的。
```
InputStream in = new FileInputStream("test.txt");
InputStream bin = new BufferedInputStream(in);
DataInputStream din = new DataInputStream(bin);
//......

```

**总结**

装饰器模式主要解决继承关系过于复杂的问题，通过组合来替代继承。它主要的作用是给原始类添加增强功能。这也是判断是否该用装饰器模式的一个重要的依据。除此之外，装饰器模式还有一个特点，那就是可以对原始类嵌套使用多个装饰器。为了满足这个应用场景，在设计的时候，装饰器类需要跟原始类继承相同的抽象类或者接口。








