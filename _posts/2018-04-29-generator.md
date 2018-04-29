---
layout: post
title: "面试：你知道为什么会有 Generator 吗"
date: 2018-04-29
tags: [javascript]
categories: programe
---

之前面试时有被问到为什么会有 Generator， 还好没懵逼。想知道的吗，往下翻。

循环的问题
-----

`ES5` 里遍历一个数组需要这样:

    const colors = ["red", "green", "blue"];
    for (var i = 0, len = colors.length; i < len; i++) {
      console.log(colors[i]);
    }


可以看出:

1.  它既要追踪下标位置，
2.  还要判断循环何时停止。

这段代码逻辑简单, 但写法复杂而枯燥。而且很常用，所以很容易因手抖而出bug。为了简化写法，降低出错机率，`ES6` 引入了一些新的语法，其中一个是 `iterator`。

什么是 iterator
------------

`iterator` 也是一种对象，不过它有着专为迭代而设计的接口。它有`next` 方法，该方法返回一个包含 `value` 和 `done` 两个属性的对象 (下称 `result` )。前者是迭代的值，后者是表明迭代是否完成的标志 \-\- 布尔值: `true` 表示迭代完成，`false` 表示没有。`iterator` 内部有指向迭代位置的指针，每次调用`next`, 自动移动指针并返回相应的 `result`。

下面是一个自定义的 `iterator`:

    function createIterator(items) {
      var i = 0;
      return {
        next: function () {
          var done = (i >= items.length);
          var value = !done ? items[i++] : undefined; return {
            done: done,
            value: value
          };
        }
      };
    }



    var iterator = createIterator([1, 2, 3]);
    console.log(iterator.next()); // "{ value: 1, done: false }"
    console.log(iterator.next());  // "{ value: 2, done: false }"
    console.log(iterator.next());  // "{ value: 3, done: false }"
    console.log(iterator.next());  // "{ value: undefined, done: true }"
    // 后续的所有调用返回的结果都一样
    console.log(iterator.next());  // "{ value: undefined, done: true }"


可以看出，只需要我们不断执行`next`就可以，不需要手动追踪迭代的位置。比开篇的循环简单了些。

> **注意:** 最后一次迭代返回的`value`, 并不是集合(如上例子中的 `items` )里的值，而是 `undedined` 或者函数返回值。 [查看这里，可以了解返回值与result的关系](https://github.com/jeyvie/understanding-ES6/blob/master/docs/8.2.generator_advanced.md)

generator 是什么
-------------

上面虽然是比 `for` 循环简单了些，但手动写个 `iterator` 太麻烦了，所以`ES6` 推出 `generator` ，方便创建 `iterator`。也就是说，`generator` 就是一个返回值为 `iterator` 的函数。

其语法如下:

    function* createIterator() {
      yield 1;
      yield 2;
      yield 3;
    }
    // generators可以像正常函数一样被调用，不同的是会返回一个 iterator
    let iterator = createIterator();
    console.log(iterator.next().value); // 1
    console.log(iterator.next().value); // 2
    console.log(iterator.next().value); // 3


例子很明了，`*` 标明这是个 `generators`， `yield` 用来在调用 `next`时返回 `value`。

方法里的 `generator`, 可以简写:

    let o = {
      createIterator: function* (items) {
        for (let i = 0; i < items.length; i++) {
          yield items[i];
        }
      }
    };


    // 等同于
    let o = {
      *createIterator(items) {
        for (let i = 0; i < items.length; i++) {
          yield items[i];
        }
      }
    };



可以看到，创建 `generator` 函数有三种方法

1.  函数声明

        function* createIterator() {...}


2.  函数表达式

        const createIterator = function* () {...}


3.  对象里的简写方式

        let o = {
          *createIterator(items) {
             ...
           }
        };



可以认为，就是在函数关键字 `function` 和函数名之间加 `*`, 只不过，不同场景下关键和函数名可以省略罢了

    [function] * [name]() {}

    // * 可以靠近关键字也可以靠近函数名，或两不靠近，都可以


**注意点:**

1.  需要注意的是，`yield` 不能跨函数：

        function* createIterator(items) {
          items.forEach(function (item) {
            // 语法错误
            yield item + 1;
          });
        }


2.  箭头函数不能用做 `generator`


iterable 和 for-of 循环
--------------------

### iterable

和 `iterator` 紧密是相关的，有个叫 `iterable` 的对象。它有个 `Symbol.iterator` 属性，其值是个 `generator` 函数。

用代码描述的话，`iterable` 长这样：

    let collection = {
      items: [],
      *[Symbol.iterator]() {
        for (let item of this.items) {
          yield item;
        }
      }
    };


`ES6` 里的集合如数组、set、map甚至字符串，都是 `iterable`。`generator` 创建的 `iterator` 都被默认添加了 `Symbol.iterator` ，所以，它们也是 `iterable`。

所有的 `iterable`， 也都可以使用 `for-of` 循环。

### for-of 循环

虽然有了 `iterator` ，只要调用它的`next`方法，就可以迭代了。但是，每次手动调用也太麻烦了，所以 `ES6` 推出了 `for-of` 循环:

    const colors = ["red", "green", "blue"];
    for (let color of colors) {
      console.log(color);
    }


对比开篇的 `for` 循环:

    const colors = ["red", "green", "blue"];
    for (var i = 0, len = colors.length; i < len; i++) {
      console.log(colors[i]);
    }


可以看出，`for-of` 循环

1.  不用追踪迭代位置;
2.  不用判断循环终止条件;

只需要声明变量活动每次迭代的值即可，简洁明了。

**注意：**`for-of` 只能用在 `iterable` 上，用其他对象上会报错。

内置的 iterator
------------

`iterator` 是 `ES6` 里一个重要的部分，一些内置的数据类型都内置了 `iterator`，方便开发。

这里简要介绍下有 `iterator` 的数据类型。虽然简单，但不是不重要。

### 集合的 iterator

`ES6` 里的集合有三类:

1.  数组
2.  set
3.  map

他们下面的 `iterator` 有:

`entries():` 返回迭代结果为键值对的 `iterator`; `values():` 返回迭代结果为集合里的值的 `iterator`; `keys():` 返回迭代结果为集合里的 `key` 的 `iterator`;

下面以 `map` 为例：

    let tracking = new Map([
      ['name', 'jeyvie'],
      ['pro', 'fd'],
      ['hobby', 'programming']
    ]);

    for (let entry of tracking.entries()) {
      console.log(entry);
    }
    // ["name", "jeyvie"]
    // ["pro", "fd"]
    // ["hobby", "programming"]

    for (let key of tracking.keys()) {
      console.log(key);
    }
    // name
    // pro
    // hobby

    for (let value of tracking.values()) {
      console.log(value);
    }
    // jeyvie
    // fd
    // programming


其中 `set` 里的 `key` 和 `value` 相等，都是 `set` 里的值。数组的 `key` 是其下标 `index`, `value` 是里面的值。

另外，各集合都有默认的 `iterator` 供 **for-of** 调用。 其执行结果，反应的是集合如何被初始化的。

如上例子 `tracking` ：

    for (let value of tracking) {
      console.log(value);
    }
    // ["name", "jeyvie"]
    // ["pro", "fd"]
    // ["hobby", "programming"]


对应这 `Map` 实例化是出入的子项 \-\- 有两个元素的数组。

其他的`set`、 数组与之类似，不赘述了。

> 需要留意，经验证 `chrome v65` 和 `node v8.0` 里数组都没有 `values()`。也许是因为对数组而言，它比较冗余吧。

### 字符串的 iterator

字符串里 `iterator`，理解一句话就可以: **是基于 `code point` 而不是`code unit` 迭代的**。比较一下两个例子:

`例子1：` 基于 `code unit`

    var message = "A 𠮷 B";
    for (let i=0; i < message.length; i++) {
        console.log(message[i]);
    }

    // A
    // (空)
    // �
    // �
    // (空)
    // B


`例子2：` 基于 `code point`

    var message = "A 𠮷 B";
    for (let c of message) {
      console.log(c);
    }

    // A
    // (空)
    // 𠮷
    // (空)
    // B


基于`code point`可以理解为基于字符, 关于它是什么，可以查看字符串那章。

### NodeList 的 iterator

以前要迭代 `NodeList` (元素集合)，需要用 `for` 循环:

    // 可以这样
    for (var i=0, len=NodeList; i<len; i++) {
    	var el = NodeList[i]
    	// ....
    }



`ES6` 可以直接这样:

    for (let el of NodeList) {
    	// el 就是集合里的每个元素
    }


> 另外，现在 `NodeList` 也有`forEach`方法了

spread操作符

展开操作符 `...` 和 iterable
----------------------

展开操作符可以用在所有的 `iterable` 上，并根据该 `iterable` 的默认 `iterator` 决定取哪个值。

比如我们可以将字符串转为数组:

    [...'A𠮷B']
    // ['A', '𠮷', 'B']


结语
--

这里以 `for` 循环的问题开始， 讲述 `ES6` 里怎么解决这个问题，由此引出 `generator`, 然后带出了 `iterator` 和 `iterable`, 以及操作 `iterable` 的 `for-of`。 最后讲了 `ES6` 里结合 `for-of`, 能使`内置集合`的操作起来更加方便。

当然了，`for` 循环问题也许并不是最初 `generator` 产生的原因，`generator` 也不只是解决了 `for` 循环这一个问题，它还有更多的[高级功能](https://github.com/jeyvie/understanding-ES6/blob/master/docs/8.2.generator_advanced.md)，在实际中的作用更大，后面我会继续发表出来。

那我为什么要写这篇文章，还成了个“标题党”呢？因为我好奇，我不只是想知道一项技术当前到底怎么用，还是知道它为何出现，想知道其来龙去脉。这样，一方面能更了解了技术，另一反面，也更能了解技术发展的趋势，这样，才能可能看得清前方啊，不至于东奔西撞，走那么多弯路。

最后，一切都是抛砖引玉，欢迎大家批评指正！