---
layout:     post
title:      Swift函数式编程－函数
category: blog
description: Swift 支持函数式编程，这一篇介绍Swift中的函数
tags: blog
---

Swift支持函数式编程，这一篇介绍Swift中的函数。

### 高阶函数（Higher-order function）

高阶函数，指可以将其他函数作为参数或者返回结果的函数。

Swift中的函数都是高阶函数，这和Scala，Hashkell一致。与此对照的是，Java中没有高阶函数（Java 7支持闭包之前）。Java中方法没法单独存在，方法总是需要和类捆绑在一起。当你需要将一个函数传递作为参数给另外一个函数时，需要一个类作为载体来携带函数。这也是Java中监听器（Listener）的做法。

高阶函数对于函数式语言很重要，原因至少有两个:

- 首先，高阶函数意味着您可以更使用更高的抽象，因为它允许我们引入计算的通用方法。例如，可通过抽象出一个通用机制，遍历数组并向其中的每个元素应用一个（或多个）高阶函数。高阶函数可以被组合成为更多更复杂的高阶函数，来创造更深层的抽象。

- 其次，通过支持函数作为返回值，就可支持构建动态性与适应性更高的系统。

### 一级函数（First-class function）

一级函数，进一步扩展了函数的使用范围，似得函数成为语言中的“头等公民”。这意味函数可在任何其他语言结构（比如变量）出现的地方出现。一级函数是更严格的高阶函数。Swift中的函数都是一级函数。

### 闭包

闭包是一个会对它内部引用的所有变量进行隐式绑定的函数。也可以说，闭包是由函数和与其相关的引用环境组合而成的实体。

￼函数实际上是一种特殊的闭包,你可以使用{}来创建一个匿名闭包。使用 in 来分割参数和返回类型。

	let r = 1...3
	let t = r.map { (i: Int) -> Int in
  		return i * 2
	}

map函数遍历了数组，用闭包处理了所有元素。并返回了一个处理过的新数组。

Objective-C在后期加入了对闭包支持。闭包是一种一级函数。通过支持闭包，objective-C拓展其语言表达能力。但是如果将Swift的闭包语法与Objective-C的闭包相比，Swift的闭包显得相当简洁和优雅，Objective-C的闭包则显得有些繁重复杂。

### 函数柯里化（Function Curring）

函数柯里化（Function Curring），是指接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数。这个名词来源于逻辑学家 Hashkell Curring。编程语言Hashkell也取自这位逻辑学家的名字。

Hashkell中函数都可以柯里化。在Hashkell里的函数参数的型别声明也暗示了函数是柯里化的。Hashkell中，返回值和参数之间，各个参数之间都是以`－>`分隔。这是因为，如果你向可以接受多个参数的函数传入一个参数，函数仍然有返回值。它的返回值是另外一个函数。这个函数可以接受剩余的参数，我们称这个返回的函数为`不全呼叫函数`。本质上讲，Haskell的所有函数都只有一个参数。

下面语句在命令行中展示了Hashkell里max的型别：

	Prelude> :type max
	max :: Ord a => a -> a -> a

其实也可以写作：

	max :: (Ord a) => a -> (a -> a)

这意味着，如果向max传入一个参数a，将返回一个型别为`(a -> a)`的函数。

柯里化为构造新函数带来了方便。也免除了一些一次性的中间函数的编写工作。

Swift可以写出柯里化函数，虽然它还是保留了和Java类似的非柯里化函数的写法。以max函数为例，Swift中柯里化函数如下：

	func max(a: Int)(b: Int) -> Int {
 		return a > b ? a : b;
	}

	let max3 = max(3)
	max3(b: 5)

### 函数式编程

我们使用一个确定质数的简单例子来表现函数式编程的特点。在这个例子中，我们会用面向对象和函数式编程实现确定质数的算法。以对比两者的不同。

这个例子是确定某个整数是否是质数。质数的因数只能是及其本身。我们使用这种算法：首先找出数字的因数，然后求所有因数的和，如果所有因数和为该数字加一，就可以确定该数字是质数。

为了先用面向对象的通常写法来实现该算法：

	class PrimeNumberClassifier {

	   let number: Int

  	   init(number: Int){
    		self.number = number
  	   }

  		func isFactor(potential: Int) -> Bool {
    		return number % potential == 0
	  	}

  		func getFactors() -> [Int] {
    		var factors : [Int] = Array<Int>()
    		for it in 1...number {
      			if isFactor(it) {
        			factors.append(it)
      			}
    		}
    		return factors
  		}

  		func sumFactors() -> Int {
    		let factors = getFactors()
    		var sum = 0
    		for factor in factors {
      			sum += factor
    		}
    		return sum
  		}

  		func isPrime() -> Bool {
    		return self.sumFactors() == number + 1
  		}
	}

接着我们使用函数式写法：

	func isFactor(number: Int)(potential: Int) -> Bool {
	  return (number % potential) == 0
	}

	func factors(number: Int) -> [Int] {
  		let isFactorForNumber = isFactor(number)
  		return Array(1...number).filter {
  			isFactorForNumber(potential: $0)}
	}

	func sumFactors(number: Int) -> Int {
  		return factors(number).reduce(0){ (total, num) in 
  			total + num }
	}

	func isPrime(number: Int) -> Bool {
  		return sumFactors(number) == number + 1
	}

由于Swift的函数都是一级函数。所以，我们可以使用filter和reduce这样接受闭包的函数提供对筛选和求和更简洁的表达方式。函数式写法中，所有的函数都是无状态的，无副作用的。也就是说无论你调用几次，只要函数的输入参数确定了，函数的输出就确定了。由于无状态，这里的每个函数都是易于复用的。但你把而算法是通过组合各个函数而实现的。


### 总结

函数式编程的核心是函数，函数是“头等公民”。这就像面向对象语言的主要抽象方法是类。Swift中的函数具有函数式语言中的函数的所有特点。这种支持使得你可以很容易地使用Swift写出函数式风格的代码。