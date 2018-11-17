---
title: C++学习5 IO输入输出 
tags: 梁子20163933
grammar_cjkRuby: true
---

# 简单IO流拾遗
首先，有关IO流的输入输出主要包括三个方面：
1. iostream，也就是读写流的基本类型，一般用于再控制台的输出输入。
2. fstream，基于对文件的输入输出，用于读取写入文件
3. sstream，读写内存string对象的类型

此外，为了支持宽字符的显示和读取，C++支持一种wchar_t类型，即wcin对应cin，wcout对应cout。还比如fstream下包括ifstream和wifstream两种类型。
当然，由于继承和多态的特性，istringstream与wistream都是继承自istream，因而都可以实现基类的操作。因而本笔记下面不做区别。

下面是对IO对象进行处理的时候的几点注意：
1. IO对象不能被拷贝
2. IO对象不能作为形参被传递，也不能作为函数的返回值。但是我们可以使用引用方式传递和返回流。
3. IO传递和返回的引用不能是const的，因为其再读写的过程中会发生改变。

注意，一个IO输入输出很有可能发生错误，再第280页给出了一个流对象的一些方法，用来实现对该流对象是否发生错误的访问。当一个流对象产生错误时，该对象后续的输入输出即会停止而无效，因为一般常常将其作为条件判断：
```c++
while(cin>>words)
{
//......
}
```
下面给出一个例子，即一个接受输入流作为参数、返回一个输入流的函数：
```c++
istream& ReadInput(istream& s)
{
	string A;
	while (s >> A, !s.eof())
	{
		if (s.bad())
			throw runtime_error("IO流错误");
		if (s.fail())
		{
			cerr << "数据错误,请重试" << endl;
			s.clear();
			s.ignore(100, '\n');
			continue;
		}
		cout << A << endl;
	}
	s.clear();
	return s;
```
## 查询流的状态：
可以使用rdstate来返回一个iostate类型的值从而实现查询流的状态，其中每一位的布尔值都表示一种特殊的效果，这里就不一一列举。

关于缓存器的输出和刷新常常使用endl，ends，flush等放在输出流的末尾，也使用unitbuf和nounitbuf来控制是否每一次都进行flush操作，不详细展开。
# fstream
首先给出一些基本的操作：
```c++
fstream f; //创建一个未绑定的文件流对象
fstream f(filename);//创建一个fstream对象并打开文件filename，其中filename可以是string类型，也可以是char数组。
fstream f(filename, mode);// 以一定的方式打开文件

f.open(filename);
f.close();
f.is_open();    //这三个较为常用，就不解释
```
对于一个已经打开的文件，我们同样可以判断其是否打开成功。常用的方法是：
```c++
ofstream out;
out.open(filename);
if(out)
{
......
}
```
## 自动构造和析构
即这些构造和析构的过程都是隐式的。
例如：
```c++
for (auto p=argv+1;p!=argv+argc;++p)
{
	ifstream f(*p);
	if(f)
		process(f);
	else
		cerr<<""<<endl;
}
```
以上代码中没有用到open，也没有使用close（），但是每次都会自动的调用这两个函数。
## mode of filestream
例如：
1. in 以只读模式打开
2. out 以写模式打开
3. app 每次写操作前均定位到文件末尾
4. ate 打开文件后立即定位到文件末尾
5. trunc截断文件                          ？？？？？？
6. binary 以二进制方式进行IO

这几种模式之间的制衡关系见第286页。

注意：
除非指定其为app模式，否则ofstream流的文件被打开后文件里之前的内容将会消失（很好理解）。
# string流
这一部分emmmmmmm还是略掉吧。感觉用处一般般。 = =！