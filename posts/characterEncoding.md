#字符编码

##Unicode

计算机内部，所有信息最终表示为二进制的字符串，也就是二进制数据。

一个字节8个二进制位(bit)可以组合成256种不同的状态。美国将其中128种状态与英语字符做了对应，形成了一套字符编码ASCII。

其他国家的如果也在这256种状态里做文章，那么一种状态在不同地方含义就不同；另外中文这种不够用的只能另起炉灶，采用多个字节表示一个状态，如GB2312中文编码。

为了统一，出现了unicode字符编码，一个大集合，将世界上所有符号都纳入，每个符号对应一个独一无二的编码。

但unicode只规定了二进制码与符号之间的对应关系，没规定该如何存储。如果采用大字节一个存储位，对英语来说浪费；另外如何区分unicode与ASCII。出现了不同的unicode实现方式。

	- utf-32, 4字节表示一个字符，完全对应unicode编码，浪费空间

	- unicode码最常见的实现方式utf-8是一种变长的编码方式，如果一个码是单字节的话其表示和ASCII一样。

	- usc-2, 两个字节存一个unicode码，随着unicode的扩大，显然不够用了

	- utf-16，ucs-2的超集，2个或者4个字节存一个unicode码

##js编码方式

js使用的不是以上的编码方式，而采用了ucs-2编码，两个字节存入字符的unicode码。所以当处理字符编码为4字节的字符就尴尬了，需要手动判断一下字符的码点范围，判断方法见参考2。

es6解决了这些历史遗留问题

```js
for (let s of string ) {
  // ...
}
Array.from(string).length
String.fromCodePoint()：从Unicode码点返回对应字符
String.prototype.codePointAt()：从字符返回对应的码点
String.prototype.at()：返回字符串给定位置的字符
//u，正则提供了u修饰符提供了对4字节码点的支持
unicode正规化，'\u01D1'.normalize() === '\u004F\u030C'.normalize()，unicode对合成字符提供了不同的表示方法，用normalize方法使其正规化
```

##base64

由上面utf-8等的介绍可以发现，**utf-8等解决是如何将一个符号（或者说unicode码）编码为二进制字节数据**，而base64解决的是**如何将一段二进制数据表示为字符串**，比如某些不支持直接读取二进制数据的场合，可以将二进制数据转为base64码然后传进来。

选出64个字符，a-z, A-Z, 0-9, +/=, 作为基本字符集，将其他符号转为这个字符集种的字符。

转换步骤：

	1. 每三个字节为一组
	2. 将一组分为4部分
	3. 每部分前面加个00，就是4个字节
	4. 将每个字节与符号集对应，就是base64的编码值

可以看到比原文大1/3左右

问：utf-8转换为base64？
答：utf-8本来就是符号的二进制码的保存形式，所以说直接转就行

问：保证js与php将一段字符串转出同样的base64码？
答：首先确定base64码的二进制数据来源，php的字符串utf-8，js为utf-16，所以要先将utf-16转为utf-8，然后base64转之，解码base64解，然后utf-8转为utf-16

##参考

- [字符编码笔记](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

- [Unicode与JavaScript详解](http://www.ruanyifeng.com/blog/2014/12/unicode.html)