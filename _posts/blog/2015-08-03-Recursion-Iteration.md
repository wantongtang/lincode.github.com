---
layout: post
title: 递归和迭代 
description: 这一篇讨论递归函数，以及递归和迭代在计算过程上会呈现出不同的形态。
category: blog
---

## 递归函数

递归可以描述不同的概念，如果说一个函数是递归的，那么就是说函数的定义中（直接或者间接地）引用了该函数本身。

比如求斐波那契数列，使用Swift实现：

	func fib(n: Int) -> Int {
  		if n <= 1 {
    		return n
  		} 
    	return fib(n - 1) + fib(n - 2)
	}

从函数定义看，fib(n)的计算调用了fib(n-1)和f(n-2)。所以，fib是一个递归函数。

### 尾递归

递归函数中有一类叫`尾递归`，比较特殊，可以单独说一下。在说尾递归之前，可以先了解一下`尾调用`。尾调用是指一个函数的最后一个动作是一个函数调用的情形：即这个调用的返回值直接被当前函数返回。这种情形下称该调用位置为尾位置。若这个函数在尾位置调用本身（或是一个尾调用本函数的其他函数等等），则称这种情况为尾递归。尾递归是递归的一种特殊情形。尾调用不一定是递归调用。

	func f(n: Int) -> Int {
		return g(n: n)
	}

形如以上代码的函数就是尾调用。当调用变为调用自身时，尾调用就变成了尾递归。举一个求阶乘的尾递归的例子：

	func factorial(n: Int, total: Int) -> Int {
  		if n == 1 {
    		return total
  		}
  		return factorial(n - 1, total: n * total)
	}

### 尾递归优化

函数调用是有开销的。当一个函数调用发生时，计算机必须“记住”调用函数的位置 — 返回位置，才可以在调用结束时带着返回值回到原位置，以继续调用前的计算。返回位置一般会存放在调用栈上。

但在尾调用的情况中，计算机不需要记住尾调用的位置，因为该位置已经是函数的尾部，就算是保存了递归调用的现场，当从递归调用返回时，除了再次返回，也已无其他事可做了。所以，保存方法之前积累的各种状态对于递归调用结果已经没有任何意义了，因此完全可以把本次方法中留在堆栈中的数据清除，把空间让给最后的递归调用。这样的优化使得递归不会在调用堆栈上产生堆积，意味着即使是“无限”递归也不会让堆栈溢出。这就是尾递归优化的作用。

对函数调用在尾位置的递归的函数，由于函数自身调用次数很多，递归层级很深，尾递归优化则使原本 O(n) 的调用栈空间，变为只需要 O(1) 的空间。因此一些编程语言的标准要求语言实现进行尾调用消除，这样可以大大提升尾递归的效率。

## 计算过程

### 递归计算过程

我们以阶乘的计算为例说明计算过程所呈现的不同“形状”。

有一种计算阶乘的方式，这里使用递归函数定义了计算阶乘的函数：

	func factorial(n: Int) -> Int {
  		if n == 0 {
    		return 1
  		}
  		return n * factorial(n - 1)
	}

现在我们试着描述这个函数的计算过程，以factorial(5)为例，一步步代换其计算过程。我们可以看到一个先逐步展开而后收缩的形状。在展开阶段里，这一计算过程构造起一个推迟进行的操作所形成的链条（在这里是一个乘法的链条），收缩过程表现为这些运算的实际执行。其形状可以描绘为如下的图例：

	(factorial 5)
	(5 * (factorial 4))
	(5 * (4 * (factorial 3))
	(5 * (4 * (3 * (factorial 2))
	(5 * (4 * (3 * (2 * (factorial 1)))
	(5 * (4 * (3 * (2 * 1))))
	(5 * (4 * (3 * 2)))
	(5 * (4 * 6))
	(5 * 24)
	120

这样的计算过程是一个递归计算过程。递归计算过程由一个推迟执行的运算链条刻画，要执行递归计算过程，解释器就需要维护好那些以后要执行的操作的轨迹。

### 迭代计算过程

然后我们再使用另外一种观点来计算阶乘。我们可以得到如下函数：

	func fact-iter(n: Int, total: Int) -> Int {
  		if n == 1 {
    		return total
  		}
  		return fact-iter(n - 1, total: n * total)
	}

当我们也以factorial(5)为例，一步步代换其计算过程时。我们能发现其呈现如下形式：

	(factorial 5)
	(fact-iter 1 5)
	(fact-iter 5 4)
	(fact-iter 20 3)
	(fact-iter 60 2)
	(fact-iter 120 1)
	120

这个过程的形态并没有任何增长或者收缩的势态。计算过程的轨迹都保存在几个固定数目的状态变量中。一般来说，迭代计算过程是那种状态可以用固定数目的变量描述的计算过程；而与此同时，又有一套固定的规则，描述计算过程在才一个状态到下一个状态的转换规则；还需要一个结束检查，描述计算过程终止的条件。
	
从函数角度来看，`factorial`和`fact-iter`都函数是递归函数，进一步说，`fact-iter`是一个尾递归函数。我们可以发现尾递归函数都是一种迭代计算过程。

### 对比

上面描述的两种计算过程从结果上来看并没有太多差异：两者计算的都是同一个定义域里的同一个数学函数，都需要使用与n成正比的步骤去计算出结果。确实，这两个计算过程甚至采用了同样的乘运算序列，得到了相同的部分乘积序列。但在另一方面，如果我们考虑这两个计算的“形状”，就会发现他们之间的不同。

这种不同对于计算机而言却是重要的。在迭代的情况里，计算过程的任何一点，固定数目的状态变量都提供了有关计算状态的一个完整描述。而描述一个递归计算过程，需要一些“隐含”信息，它们并未保存在程序变量里，而是由解释器维持着，指明了在所推迟的运算所形成的链条里，计算过程正处于何处（这种解释器维持运算链条，需要使用一种称为栈的数据结构）。这个链条越长，需要保存的信息也就越多。

这也是尾递归优化存在的意义。通常递归计算过程可能递归很多步，这就意味着这上文所描述的运算链条会非常长。通常同一时刻计算机的资源是有限的，那么去除这种大规模资源的消耗将会有很多帮助。

从理论上说，所有的递归计算过程都可以转换为迭代计算过程。反之亦然，然而代价通常都是比较高的。但从算法结构来说，递归声明的结构并不总能够转换为迭代结构，原因在于结构的引申本身属于递归的概念，用迭代的方法在设计初期根本无法实现。

递归计算过程，通常容易理解，符合人类的思维习惯。但由于需要使用栈机制实现，其空间复杂度通常很高。对于一些递归层数深的计算，计算机会力不从心，空间上会以内存崩溃而告终。而且递归也带来了大量的函数调用，这也有许多额外的时间开销。所以在深度大时，它的时间复杂度和空间复杂度就都不好了。
