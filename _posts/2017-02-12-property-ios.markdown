---
layout:     post
title:      "@property 修饰符"
subtitle:   ""
date:       2017-02-12 16:45:00
author:     "Kila2"
tags:
- weak strong retain copy assign
---
# @property 修饰符
修饰符的作用都是针对系统生成的getter、setter方法，对于自定义getter、setter的对象只能起到参考的作用，直接使用（_变量名）同样不受关键字的影响。

|关键字          | 基本类型        | Objective-C对象  | Block          |Core Foundation对象 |
|:-------------:|:--------------:|:---------------:|:--------------:|:-----------------:|
| assign        | √              |  √              |  √             | √                 |
| weak          |                |  √              |  √             |                   |         
| strong        |                |  √              |  √(=copy)      |                   |               
| retain        |                |  √              |  √(warning)    |                   |               
| copy          |                |  √              |  √             |                   |               
| atomic        | √              |  √              |  √             | √                 |               
| noatomic      | √              |  √              |  √             | √                 |   

注：对于Objective-C对象，Block，strong为默认修饰符,其他类型默认用assign修饰，assgin,strong不可共存。

## 基本类型int，float，double等

1. 基本类型不受ARC管理，由系统自动回收栈内存。
2. 用assign作为属性默认值，即getter、setter不做任何多余处理。
3. 默认支持线程保护，保护getter、setter的完整性。

## Objective-C对象

1. Objective-C对象受到ARC管理，根据引用计数自动回收内存。
2. assign与weak
 * 相同点：不增加引用计数
 * 不同点：weak引用计数为0时会把指针指向nil，assgin会产生野指针。
3. strong与retain
 * 特点：引用计数都增加+1,对于Objective-C对象strong=retain,但是对于Block不同，下面再说。
4. copy
 * copy比较复杂,先说一下我理解copy的语意，对于用copy修饰的对象。在setter中赋值a = b时候是有替换为a = [b copy]，即调用copy方法。如果该对象没有copy方法或者没有实现NSCopying协议，使用该修饰符可以通过编译，但在运行时会报错。而对于某些NSMutable对象，例如：
 
	```
	NSMutableArray *a = [[NSMutableArray alloc]init];
	
	NSMutableArray *b = [a copy];
	```

 * 由于copy方法的特性返回的对象实际上是不可修改的，实际上就是NSArray对象。
NSArray对象没有addObject，removeAll等方法，如果调用到必然出现运行时错误。
对于NSString的copy方法有些特殊，例如:

	```
	@property (copy) NSString *copyStr;
	
	NSMutableString *mStr = [NSMutableString stringWithString:@"string"];
	
	self.copyStr = self.mStr;
	
	[self.name2 appendString:@"11122"];
	```
 * 这时copyStr是“string11122”还是“string”？？？

 * 为了处理这个特殊性，NString copy会检查一下参数类型是不是可变的。
 * 如果是可变的，进行深拷贝。
 * 如果不是可变的，进行浅拷贝效果与用strong修饰相同。

5. 默认支持线程保护，保护getter、setter的完整性。

## Block

1. Block受到ARC管理，但于Objective-C对象有所不同，由系统自动回收内存。

2. 虽然上面的修饰符都可以用并且通过编译，但retain会出现警告，提示用copy替换。
 * 以下是个人理解可能有误，欢迎指正。
 * block比较复杂创建的时候是在栈中，运行的时候如果没有捕获对象就在栈中运行，如果有捕获对象就先copy到堆中运行。
3. assign，weak，retain,strong，copy
 * 都不会增加引用计数，block的引用计数始终为1。
 * 不管用那个关键字修饰可能内部实现都是一样的。
 * 但是为了更加符合语义推荐使用copy关键字。
 * 4. 默认支持线程保护，保护getter、setter的完整性。
* 关于block更详细的介绍可以参考[点击这里](http://tanqisen.github.io/blog/2013/04/19/gcd-block-cycle-retain/)

## Core Foundation对象

1. Core Foundation对象不受ARC管理，需要使用CFRetain CFRelease手动管理内存。
2. 用assign作为属性默认值
3. 默认支持线程保护，保护getter、setter的完整性。

