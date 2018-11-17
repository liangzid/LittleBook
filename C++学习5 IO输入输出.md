---
title: C++学习5 IO输入输出 
tags: 梁子20163933
grammar_cjkRuby: true
---

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